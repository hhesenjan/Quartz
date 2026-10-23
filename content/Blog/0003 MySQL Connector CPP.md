---
title: "MySQL Connector C++"
created: 2024-04-20
modified: 2024-04-20
tags: ["MySQL", "SQL"]
draft: false
---

## Introduction 

MySQL Connector C++ contains the source code, libraries (.lib), and dynamic link libraries (.dll) which are necessary to create a connection with MySQL server using C++. Once the connection is established you can execute commands to create, update, and delete databases, tables, and records within the tables.

## MySQL Connector for MacOS 

Installing MySQL Connector C++ on MacOS:

```bash
brew install mysql-connector-c++
```

Compiling with G++ using MysQL Connector C++:

```bash
g++ -std=c++17 -o main main.cpp -I "/opt/homebrew/Cellar/mysql-connector-c++/8.3.0/include" -L "/opt/homebrew/Cellar/mysql-connector-c++/8.3.0/lib -lmysqlcppconn8"
```

## MySQL Connector for Windows

Installing MySQL Connector C++ on Windows: [MySQL C++ Connector Download](https://dev.mysql.com/downloads/connector/cpp/)

Setting up MYSQL Connector C++ for Visual Studio 2022:

1. `Additional Include Directories` = `C:\MySQL\MySQL Connector C++ 8.4\include`
2. `Additional Library` = `C:\MySQL\MySQL Connector C++ 8.4\lib64\vs14`
3. `Additional Dependencies` = `libssl.lib;libcrypto.lib;mysqlcppconn-static.lib;`

## Basics of MySQL Connector C++ 

You will need to include the `jdbc.hheader` header since it will import all the headers that are necessary to establish a connection with MySQL server and to create, update, remove, and execute on MySQL server.

```cpp
#include <mysql/jdbc.h>
```

To establish a connection with MySQL server, the objects `sql::Driverand` and `sql::Connection` are needed as these two objects works together to establish the connection to MySQL server.

```cpp
#include <mysql/jdbc.h>

int main() {
    // SQL Objects
    sql::Driver* driver;
    sql::Connection* con;

    // Preparing driver object
    driver = get_driver_instance();

    // Connecting to the database
    con = driver->connect("[IP-ADDR]", "[USERNAME]", "[PASSWORD]");

    // Exiting the application
    return 0;
}
```

Once the connection has been successfully established, the `sql::Statement` object can be used to execute updates to MySQL database.

```cpp
#include <mysql/jdbc.h>

int main() {
    // SQL Objects
    sql::Driver* driver;
    sql::Connection* con;
    sql::Statement* stmt;

    // Preparing driver object
    driver = get_driver_instance();

    // Connecting to the database
    con = driver->connect("[IP-ADDR]", "[USERNAME]", "[PASSWORD]");

    // Setting up statement
    stmt = con->createStatement();

    // Creating a database named `test_db`
    stmt->executeUpdate("CREATE DATABASE IF NOT EXISTS test_db;");

    // Selecting the database
    stmt->executeUpdate("USE test_db;");

    // Creating users table
    stmt->executeUpdate("CREATE TABLE IF NOT EXISTS users(FirstName varchar(255), LastName varchar(255));");

    // Exiting the application
    return 0;
}
```

MySQL Connector C++ also comes with prepared statements `sql::PreparedStatement` to avoid SQL injection vulnerabilities. It’s important to use prepared statements when user input is being injected onto queries since these can be modified to return values from other databases and tables.

**SQL Injection Vulnerable**

```cpp
#include <mysql/jdbc.h>

int main() {
    // SQL Objects
    sql::Driver* driver;
    sql::Connection* con;
    sql::Statement* stmt;

    // Preparing driver object
    driver = get_driver_instance();

    // Connecting to the database
    con = driver->connect("[IP-ADDR]", "[USERNAME]", "[PASSWORD]");

    // Setting up statement
    stmt = con->createStatement();

    // Selecting the database
    stmt->executeUpdate("USE test_db;");

    // Creating users table
    stmt->executeUpdate("CREATE TABLE IF NOT EXISTS users(FirstName varchar(255), LastName varchar(255));");

    // String variables
    std::string sql_query, fname, lname;

    // User input for firstname
    std::cout << "Firstname: ";
    std::cin >> fname;

    // User input for lastname
    std::cout << "Lastname: ";
    std::cin >> lname;

    // SQL Injection Vulnerable
    sql_query = "INSERT INTO users(FirstName, LastName) VALUES ('" + fname + "'," + lname + "');";

    // Executes the `sql_query`
    stmt->executeQuery(sql_query);

    // Exiting the application
    return 0;
}
```

**MySQL Prepared Statements:**

```cpp
#include <mysql/jdbc.h>

int main() {
    // SQL Objects
    sql::Driver* driver;
    sql::Connection* con;
    sql::Statement* stmt;
    sql::PreparedStatement* pre_stmt;

    // Preparing driver object
    driver = get_driver_instance();

    // Connecting to the database
    con = driver->connect("[IP-ADDR]", "[USERNAME]", "[PASSWORD]");

    // Setting up statement
    stmt = con->createStatement();

    // Selecting the database
    stmt->executeUpdate("USE test_db;");

    // Creating users table
    stmt->executeUpdate("CREATE TABLE IF NOT EXISTS users(FirstName varchar(255), LastName varchar(255));");

    // String variables
    std::string sql_query, fname, lname;

    // User input for firstname
    std::cout << "Firstname: ";
    std::cin >> fname;

    // User input for lastname
    std::cout << "Lastname: ";
    std::cin >> lname;

    // Preparing MySQL Prepared Statement to avoid SQL injection
    pre_stmt = con->prepareStatement("INSERT INTO users(FirstName, LastName) VALUES (?,?);");

    // FirstName is assigned fname
    pre_stmt->setString(1, fname);

    // LastName is assigned lname
    pre_stmt->setString(2, lname);

    // Executing prepared statement
    pre_stmt->execute();

    // Exiting the application
    return 0;
}
```

You can also use MySQL Connector C++ to return queries from the MySQL server using `sql::ResultSet`.

```cpp 
int main() {
    // SQL Objects
    sql::Driver* driver;
    sql::Connection* con;
    sql::ResultSet* res;

    // Preparing driver object
    driver = get_driver_instance();

    // Connecting to the database
    con = driver->connect("[IP-ADDR]", "[USERNAME]", "[PASSWORD]");

    // Setting up statement
    stmt = con->createStatement();

    // Selecting the database
    stmt->executeUpdate("USE test_db;");

    // Executing query
    res = stmt->executeQuery("SELECT FirstName, LastName FROM users;");

    // Printign FirstName and LastName from `users` table 
    while (res->next()) {
        std::cout << "Firstname: " << res->getString(1) << std::endl;
        std::cout << "Lastname: " << res->getString(2) << "\n" << std::endl;
    }

    // Exiting the application
    return 0;
}
```

## Conclusion

MySQL Connector C++ comes with all the necessary features that are needed to create, update, execute, and delete databases, tables, and records on MySQL server. It also comes with security features such as prepared statement to avoid SQL injection on our MySQL server. With the correct implementation and best practices the MySQL Connector C++ can be a fantastic tool to build applications.