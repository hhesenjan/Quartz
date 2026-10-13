---
title: "Building MCP Server with Rust"
created: 2026-07-26
modified: 2026-07-26
tags: ["AI", "MCP", "Rust"]
draft: false
---

## Introduction

I'm seeing that MCP is becoming more and more popular since it allows us to connect a tool into LLM where it can automatically perform the actions for us. I understand the usefulness of it but when it comes to giving that kind of access to a normal users it seems as a terrible ideas since these MCP servers usually uses API keys. And managing all the API keys which comes with MCP servers becomes extremely difficult. Instead of blindly ignoring the tool, I decided to dive into it since it will probably come in good use in the future.


## What is MCP servers?

Model Context Protocol (MCP) servers can be used for performing API calls, and communicating with the operating system and database. It's a way for us to increase the effectiveness of AI desktop applications such as Claude, and OpenAI since it allows us to send and retrieve data from a database as an example. The MCPs components can be separated in the following ways:

| Component | Description |
|---|---|
| **Tools** | A tool is an executable function that performs a specific operation. |
| **Resources** | A resource is meant for providing contextual information to the AI desktop application such as file content, database records, and API calls. |
| **Prompts** | A prompt is a reusable template that helps with structuring interactions with AI models. |
| **Notifications** | A notification is used for informing users about updates and changes that has happened to the MCP server. |

The MCP server can also be locally based where it uses STDIO transport layer or a remote server which uses the Streamable HTTP transport layer. I'll be focused on building a MCP server which is locally based. 

## Building MCP Server

I'll be building a MCP server which retrieves rss pages from ZerodayInitiative and HackerNews. Additionally, I'll be building functions to communicate with postgresql database to store tasks for the todo lists. Let's get started with building a MCP server!

```toml title="Cargo.toml"
[package]
name = "husenjan_mcp"
version = "0.1.0"
edition = "2024"

[dependencies]
rmcp = { version = "0.3", features = ["server", "macros", "transport-io"] }
tokio = { version = "1.46", features = ["full"] }
reqwest = { version = "0.12", features = ["json"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "std", "fmt"] }
rss = { version = "2.1", features = ["serde"] } 
serde_yml = "0.0.12"
sqlx = { version = "0.9", features = ["runtime-tokio", "postgres", "macros"] }
```

I added more libraries such as `rss`, `serde_yml`, `sqlx`  which will be used for parsing data and communicating with postgresql database. The `main()` function will be used for initializing logging, database connection, and starting the MCP server. 

```rust title="main.rs"
use std::time::Duration;
use anyhow::Context;
use rmcp::{ServiceExt};
use serde::Deserialize;
use sqlx::postgres::PgPoolOptions;
use tracing_subscriber::{EnvFilter, fmt};

//
// Application Configurations
//
#[derive(Debug, Deserialize)]
pub struct AppConfig {
    pub database_url : String,
    pub max_connection: u32,
    pub acquire_timeout_secs: u64,
    pub idle_timeout: u64
}

//
// Info - 
// Loads the configureation file.
//
impl AppConfig {
    pub fn load(path: &str) -> anyhow::Result<Self> {
        let content = std::fs::read_to_string(path)
            .with_context(|| format!("Failed to read from file: {path}"))?;

        let config: AppConfig = serde_yml::from_str(&content)
            .with_context(|| format!("Failed to parse config: {path}"))?;

        Ok(config)
    }
}

//
// Initializing Logging
//
pub fn init_logging() {
    fmt()
        .with_env_filter(EnvFilter::try_from_default_env().unwrap_or_else(|_| EnvFilter::new("info") ))
        .with_writer(std::io::stderr) 
        .init();

    tracing::info!("Tracing is initialized");
}

//
// Main -
// Responsible for establishing connection with postgresql and running the MCP server.
//
#[tokio::main]
async fn main() -> anyhow::Result<(), anyhow::Error> {
    // Initializes logging.
    init_logging();

    // Notifies about database connection.
    tracing::info!("Loading config.yaml file.");

    // Getting database info from config.yaml file.
    let config = AppConfig::load("C:\\McpServer\\config.yaml")?;

    // Notifies about database connection.
    tracing::info!("Connecting to postgresql database.");

    // Initalizing database connection.
    let db_pool = PgPoolOptions::new()
        .max_connections(config.max_connection)
        .acquire_timeout(Duration::from_secs(config.acquire_timeout_secs))
        .idle_timeout(Duration::from_secs(config.idle_timeout))
        .connect(config.database_url.as_str())
        .await?;

    // Notifies that server has been initialized.
    tracing::info!("Starting HusenjanMCP server.");

    // Running the MCP server
    let transport = (tokio::io::stdin(), tokio::io::stdout());
    let service = husenjan_mcp::HusenjanMCP::new(db_pool).serve(transport).await?;
    service.waiting().await?;

    Ok(())
}
```

The MCP server components are created inside of `router.rs` file. The tools are also declared there which allows the AI desktop application to call these different tools to perform actions. Additionally, there are data structures which are used to define the arguments that are available for the different functions (remember that tools are functions).

```rust title="router.rs"
use anyhow::Context;
use rmcp::{ServerHandler, handler::server::{router::tool::ToolRouter, tool::Parameters}, model::*, schemars::{self, JsonSchema}, tool, tool_handler, tool_router};
use serde::Deserialize;
use sqlx::PgPool;
use crate::{make_hackernews_request, make_zdi_published_request, make_zdi_upcoming_request};

//
// MCP Server Params
//
#[derive(Deserialize, JsonSchema)]
pub struct McpZDIParams {
    total: usize
}

#[derive(Deserialize, JsonSchema)]
pub struct McpHackernewsParams {
    total: usize
}

#[derive(Deserialize, JsonSchema)]
pub struct McpTodoListPatchParams {
    task_name: String,
    task_priority: String,
    task_process: String,
    new_task_name: String,
    new_task_process: String,
    new_task_priority: String,
    new_task_notes: String,
}

#[derive(Deserialize, JsonSchema)]
pub struct McpTodoListDeleteParams {
    task_name: String,
    task_priority: String,
    task_process: String,
    task_notes: String
}

#[derive(Deserialize, JsonSchema)]
pub struct McpTodoListPostParams {
    task_name: String,
    task_priority: String,
    task_process: String,
    task_notes: String
}

//
// Router
//
pub struct HusenjanMCP {
    tool_router: ToolRouter<HusenjanMCP>,
    db_pool: PgPool
}

// 
// Tool Router
//
#[tool_router]
impl HusenjanMCP {
    pub fn new(db_pool: PgPool) -> Self {
        Self {
            tool_router: Self::tool_router(),
            db_pool: db_pool
        }
    }

    //
    // Data retrieval functions - 
    // Retrieves RSS data from ZDI and HackerNews.
    //
    #[tool(description = "Get CVEs from ZerodayInitiative (ZDI). The parameter `total` allows us to specify the amount of CVEs to return.")]
    async fn get_zdi_published_cves(&self, Parameters(McpZDIParams { total }): Parameters<McpZDIParams>) -> Result<String, rmcp::ErrorData> {
        // Logging call.
        tracing::info!("The get_zdi_published_cves function has been called.");

        let data = make_zdi_published_request(total)
            .await
            .context("Failed to retrieve data from make_zdi_published_request function.")
            .map_err(|e| rmcp::ErrorData::internal_error(format!("{e:#}"), None))?; 


        Ok(data)
    }

    #[tool(description = "Get upcoming CVEs from ZerodayInitiative. The parameter `total` allows us to specify the amount of CVEs to return.")]
    async fn get_zdi_upcoming_cves(&self, Parameters(McpZDIParams { total }) : Parameters<McpZDIParams>) -> Result<String, rmcp::ErrorData> {
        // Logging call.
        tracing::info!("The get_zdi_upcoming_cves function has been called.");

        let data = make_zdi_upcoming_request(total)
            .await
            .context("Failed to retrieve data from make_zdi_upcomming_request function.")
            .map_err(|e| rmcp::ErrorData::internal_error(format!("{e:#}"), None))?; 

        Ok(data)
    }

    #[tool(description = "Get news articles from Hackernews.")]
    async fn get_hackernews_articles(&self, Parameters(McpHackernewsParams { total }): Parameters<McpHackernewsParams>) -> Result<String, rmcp::ErrorData> {
        // Logging call.
        tracing::info!("The get_hackernews_articles function has been called.");

        let data = make_hackernews_request(total)
            .await
            .context("Failed to retrieve data from make_hackernews_request function().")
            .map_err(|e| rmcp::ErrorData::internal_error(format!("{e:#}"), None))?;

        Ok(data)
    }

    //
    // Database functions -
    // Built for communicate with the postgresql database.
    //

    // Get Todo List
    #[tool(description = "Get Todo List.")]
    async fn get_todo_list(&self) -> Result<String, rmcp::ErrorData> {
        // Logging call.
        tracing::info!("The get_todo_list function has been called.");

        let rows = sqlx::query!("SELECT * from TodoList")
            .fetch_all(&self.db_pool)
            .await
            .map_err(|e| rmcp::ErrorData::internal_error(format!("{e:#}"), None))?;
        
        let data = rows
            .iter()
            .map(|e|{
                format!("Task Name: {}\n Task Process: {}\nTask Priority: {}\nTask Notes: {}\n",
                    e.task_name.as_deref().unwrap_or("Unknown"),
                    e.task_process.as_deref().unwrap_or("Unknown"),
                    e.task_priority.as_deref().unwrap_or("Unknown"),
                    e.task_notes.as_deref().unwrap_or("Unknown")
                )
            }).collect::<Vec<String>>().join("\n--\n");

        Ok(data)
    }

    // Add Task
    #[tool(description = "Add task to Todo List.")]
    async fn add_task(&self, Parameters(McpTodoListPostParams { task_name, task_process, task_priority, task_notes}) : Parameters<McpTodoListPostParams>) -> Result<String, rmcp::ErrorData> {
        // Logging call.
        tracing::info!("The add_task function has been called.");

        sqlx::query!("INSERT INTO TodoList (task_name, task_process, task_priority, task_notes) VALUES ($1, $2, $3, $4);",
            &task_name,
            &task_process,
            &task_priority,
            &task_notes
        )
        .execute(&self.db_pool)
        .await
        .map_err(|e| ErrorData::internal_error(format!("{e:#}"), None))?;

        Ok("Successfully added to Todo List.".to_string())
    }

    // Update Task
    #[tool(description = "Update a task from Todo List.")]
    async fn update_task(
        &self, 
        Parameters(McpTodoListPatchParams { task_name, task_priority, task_process, new_task_name, new_task_process, new_task_priority, new_task_notes }): Parameters<McpTodoListPatchParams>
    ) -> Result<String, rmcp::ErrorData> {
        // Logging call.
        tracing::info!("The update_task function has been called.");

        sqlx::query!(
            "UPDATE TodoList 
            SET task_name = $1, 
                task_process = $2, 
                task_priority = $3, 
                task_notes = $4 
            WHERE 
                task_name = $5 
            AND task_process = $6 
            AND task_priority = $7",
            new_task_name, new_task_process, new_task_priority, new_task_notes,
            task_name, task_process, task_priority
        )
        .execute(&self.db_pool)
        .await
        .map_err(|e| rmcp::ErrorData::internal_error(format!("{e:#}"), None))?;

        Ok("Updated row successfully.".to_string())
    } 

    // Delete Task
    #[tool(description = "Delete task from Todo List.")]
    async fn delete_task(&self, Parameters(McpTodoListDeleteParams { task_name, task_process, task_priority, task_notes}) : Parameters<McpTodoListDeleteParams>) -> Result<String, rmcp::ErrorData>{
        // Logging call.
        tracing::info!("The delete_task function has been called.");

        sqlx::query!("DELETE FROM TodoList WHERE task_name = $1 AND task_process = $2 AND task_priority = $3 AND task_notes = $4",
            task_name,
            task_process,
            task_priority,
            task_notes
        ).execute(&self.db_pool)
        .await
        .map_err(|e| rmcp::ErrorData::internal_error(format!("{e:#}"), None))?;

        Ok("Deleted the row successfully.".to_string())
    }
}

//
// Server Handler
//
#[tool_handler]
impl ServerHandler for HusenjanMCP {
    fn get_info(&self) -> ServerInfo {
        ServerInfo {
            capabilities: ServerCapabilities::builder().enable_tools().build(),
            ..Default::default()
        }
    }
}
```

The `ZDI_RSS_PUBLISHED_URL` and `ZDI_RSS_UPCOMING` are constant variables which consists of URLs to rss data. The `make_zdi_published_request()` function retrieves published CVES in rss data while the `make_zdi_upcoming _request()` function retrieves the upcoming CVEs in rss data.

```rust title="zdi.rs"
use std::time::Duration;
use anyhow::Context;
use rss::Channel;

const ZDI_RSS_PUBLISHED_URL : &str = "https://www.zerodayinitiative.com/rss/published/";
const ZDI_RSS_UPCOMING_URL : &str = "https://www.zerodayinitiative.com/rss/upcoming/";

pub async fn make_zdi_published_request(total: usize) -> Result<String, anyhow::Error> {
    // Logging
    tracing::info!("Calling the make_zdi_published_request function.");

    // Building the reqwest client.
    let client = reqwest::Client::builder()
        .timeout(Duration::from_secs(3))
        .build()?;

    // Making an request to /published
    let rsp = client
        .get(ZDI_RSS_PUBLISHED_URL)
        .send()
        .await?;

    // Retrieving the body data
    let body = rsp
        .text()
        .await.unwrap_or("".into());

    // Extracting only needed information from RSS body.
    let data = Channel::read_from(body.as_bytes())
        .context("Failed to convert RSS on make_zdi_published_request().")?
        .items
        .iter()
        .take(total)
        .map(|e| {
            let cve = format!("Title: {}\nDescription: {}\nAuthor: {}\n",
                e.title.as_deref().unwrap_or("Unknown"),
                e.description.as_deref().unwrap_or("Unknown"),
                e.author.as_deref().unwrap_or("Unknown")
            );

            cve
        }).collect::<Vec<String>>().join("\n--\n");

    Ok(data)
}

pub async fn make_zdi_upcoming_request(total: usize) -> Result<String, anyhow::Error> {
    // Logging
    tracing::info!("Calling the make_zdi_upcoming_request function.");

    // Building the reqwest client.
    let client = reqwest::Client::builder()
        .timeout(Duration::from_secs(3))
        .build()?;

    // Making an request to /upcoming
    let rsp = client
        .get(ZDI_RSS_UPCOMING_URL)
        .send()
        .await?;

    // Retrieving the body data
    let body = rsp  
        .text()
        .await
        .unwrap_or("".into());

    // Extracting only needed information from RSS body.
    let data = Channel::read_from(body.as_bytes())
        .context("Failed to convert RSS on make_zdi_published_request().")?
        .items
        .iter()
        .take(total)
        .map(|e| {
            let cve = format!("Title: {}\nDescription: {}\nAuthor: {}\n",
                e.title.as_deref().unwrap_or("Unknown"),
                e.description.as_deref().unwrap_or("Unknown"),
                e.author.as_deref().unwrap_or("Unknown")
            );

            cve
        }).collect::<Vec<String>>().join("\n--\n");
    
    Ok(data)
}
```

The `HACKER_NEWS_RSS_URL` is constant variable for HackerNews RSS URL. The `make_hackernews_request()` function is responsible for retrieving the rss data and parsing it.

```rust title="hackernews.rs"
use std::time::Duration;
use anyhow::Context;
use rss::Channel;

//
// Constants
//
const HACKERNEWS_RSS_URL : &str = "https://news.ycombinator.com/rss";


//
// Info - 
// Makes a request to HackerNews and retrieves the RSS body.
//
pub async fn make_hackernews_request(total: usize) -> Result<String, anyhow::Error> {
    // Logging
    tracing::info!("Calling the make_hackernews_request function.");

    // Building the reqwest client.
    let client = reqwest::Client::builder()
        .timeout(Duration::from_secs(3))
        .build()?;

    // Making an request to /published
    let rsp = client
        .get(HACKERNEWS_RSS_URL)
        .send()
        .await?;

    // Retrieving the body data
    let body = rsp
        .text()
        .await.unwrap_or("".into());

    // Extracting only needed information from RSS body.
    let data = Channel::read_from(body.as_bytes())
        .context("Failed to convert RSS on make_hackernews_request().")?
        .items
        .iter()
        .take(total)
        .map(|e| {
            let article = format!("Title: {}\nDescription: {}\nLink: {}\n",
                e.title.as_deref().unwrap_or("Unknown"),
                e.description.as_deref().unwrap_or("Unknown"),
                e.link.as_deref().unwrap_or("Unknown")
            );

            article
        }).collect::<Vec<String>>().join("\n--\n");

    Ok(data)
}
```

Once the MCP server is compiled it's ready to be used for retrieving data from Zeroday Initiative and HackerNews. It can also communicate with postgresql database to add, update, and delete tasks which allows us to keep track of things we need to finish off.

## Running MCP Server

To run the MCP server the following configurations has to be added into Claude's `claude_desktop_config.json` file.

```powershell title="Command"
code $env:AppData\Claude\claude_desktop_config.json
```

```json title="claude_desktop_config.json"
"mcpServers": {
    "HusenjanMCP": {
        "command": "C:\\Users\\Husenjan\\Documents\\Learning\\HusenjanMCP\\target\\debug\\husenjan_mcp.exe"
    }
}
```

Also inside of the following folder `C:\McpServer`, we need to create a YAML file called for `config.yaml` with the following content.

```YAML title="config.yaml"
database_url : "postgres://username:password@localhost:5432/todolist"
max_connection: 5
acquire_timeout_secs: 5
idle_timeout: 5
```

The MCP server should now automatically start once the Claude desktop application is ran.

## Conclusion

Building a MCP server has taught me a-lot about the importance of MCP servers. Its a way for us to excelerate the AI desktop application to perform real actions on a system which the specific instructions that we provide it. This can help with increasing productivity since a user might never actually need to leave the AI desktop application to retrieve, change, and delete data when it can be done through AI desktop application.

I also understand that there are risks with MCP servers where the user can accidentally allow the LLM to perform terrible mistake such as deleting all the data inside of a SQL table as an example. To avoid these situations its important for us to implement strict instructions and access for the LLM where it can only delete one data at the time and not multiple ones within few miliseconds to seconds apart. 