---
title: "Terraform for Microsoft Azure"
created: 2024-10-02
modified: 2024-10-02
tags: ["AZURE", "TERRAFORM", "IAC"]
draft: false
---

## Introduction

I'm currently working on a personal project which requires me to use Terraform. I decided to create this document to potentially help other people out with their journey with learning Terraform since it can be a complex topic to learn.

## What is Terraform?

Terraform is widely used by organizations to create **Infrastructure as Code (IaC)** as it allows you to build, change, and managa your cloud infrastructure in safe, consistent, and repeatable way. Terraform also supports many different cloud providers such as Amazon Cloud, Google Cloud, DigitalOceans and much more.

## Microsoft Azure CLI

Terraform requires us to install Microsoft Azure CLI on our local machine before we can start working with it. We can install Microsoft Azure CLI using the following command on Windows:

```shell
# Install Microsoft Azure CLI 
winget install -e --id Microsoft.AzureCLI
```

We will need to login to our Microsoft Azure account and select the subscription which we want to work on: 

```shell
# Login to Microsoft Azure
az login

# Shows all available subscriptions in our Tenant 
az account list

# To choose a specific subscription
az account set --subscription="SUBSCRIPTION_ID"
```

When all these steps has been completed we can now start coding our cloud environment using terraform. It's important to note that without Microsoft Azure CLI we won't be able to deploy the resources for our cloud environment.

## Terraform Code

We will need to initalize our work folder with terraform before we can start coding with it. To initialize the work folder with terraform use the following command:

```shell
terraform init
```

We can now create a `provider.tf` file and include the following configuration: 

```terraform title="/terraform/provider.tf"
terraform {
    required_providers {
        azurerm = {
            source = "hashicorp/azurerm"
            version = "~> 4.0"
        }
    }
}

provider "azurerm" {
    features{}
    subscription_id = "000000000-0000-4abe-995f-73851a2822d6"
    resource_provider_registrations = "none"
}
```

Inside `subscription_id` field you will need to put your own subscription identification otherwise terraform will throw an message. Now we can create a `main.tf` file where all our cloud resources will be defined for our cloud environment:

```terraform title="/terraform/main.tf"
# Creates a resource group
resource "azurerm_resource_group" "rg-terraform" {
    name = "rg-terraform"
    location = "west europe"
}

# Creates a virtual network 
resource "azurerm_virtual_network" "weu-vnet" {
    name = "weu-vnet"
    resource_group_name = azurerm_resource_group.rg-terraform.name
    location = azurerm_resource_group.rg-terraform.location
    address_space = [ "10.0.0.0/16" ]
}

# Creates a subnet inside virtual network
resource "azurerm_subnet" "weu-vnet-external-subnet" {
    name = "weu-vnet-external-subnet"
    resource_group_name = azurerm_resource_group.rg-terraform.name
    virtual_network_name = azurerm_virtual_network.weu-vnet.name
    address_prefixes = ["10.0.1.0/24"]
}
```

All the terraform code does is creating a resource group, virtual network, and a subnet inside the vitual network. We can deploy these resources using the following commands:

```shell
# Shows us the changes that will be done to our environment 
terraform plan 

# Deploys the cloud resources to our cloud environment 
terraform apply

# Deletes all cloud resources that terraform deployed
terraform destroy 
```

Note that `terraform destroy` will completely delete all the resources which were created and any changes done to them with click ups will be deleted. What is fantastic about terraform is the ability to quickly deploy and destroy the cloud resources instead of potentially messing it up doing it manually.

## Terraform Advanced Code 

In this section of the document I'll go through the following concept:

- Creating resource group
- Creating virtual network
- Creating subnet mask
- Creating virtual machine
- Creating security group

It can be a bit difficult for beginners to understand the code below therefore I highly recommend googling the things which you're unfamiliar with.

```terraform title="/terraform/variables.tf"
variable "default-resource-group" {
  type        = string
  description = "Default resource group"
  default     = "rg-terraform"
}

variable "default-resource-location" {
  type        = string
  description = "Default location for resource group and resources"
  default     = "West Europe"
}
```

```terraform title="/terraform/main.tf"
resource "azurerm_resource_group" "rg-terraform" {
  name     = var.default-resource-group
  location = var.default-resource-location
}

resource "azurerm_virtual_network" "weu-vnet" {
  name                = "weu-vnet"
  resource_group_name = var.default-resource-group
  location            = var.default-resource-location
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_subnet" "weu-vnet-external-subnet" {
  name                 = "weu-vnet-external-subnet"
  resource_group_name  = var.default-resource-group
  virtual_network_name = azurerm_virtual_network.weu-vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_security_group" "weu-vnet-sgroup" {
  name                = "weu-vnet-sgroup"
  resource_group_name = var.default-resource-group
  location            = var.default-resource-location

  security_rule {
    name                       = "SSH"
    priority                   = 150
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_public_ip" "weu-vm1-ip" {
  name                = "weu-vm1-ip"
  location            = var.default-resource-location
  resource_group_name = var.default-resource-group
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "weu-vm1-ni1" {
  name                = "weu-vm1-ni1"
  resource_group_name = var.default-resource-group
  location            = var.default-resource-location

  ip_configuration {
    name                          = "weu-vm1-ip"
    subnet_id                     = azurerm_subnet.weu-vnet-external-subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.weu-vm1-ip.id
  }
}

resource "azurerm_network_interface_security_group_association" "weu-vm1-ni1-sgroup" {
  network_interface_id      = azurerm_network_interface.weu-vm1-ni1.id
  network_security_group_id = azurerm_network_security_group.weu-vnet-sgroup.id
}

resource "azurerm_linux_virtual_machine" "weu-vm1" {
  name                  = "weu-vm1"
  resource_group_name   = var.default-resource-group
  location              = var.default-resource-location
  network_interface_ids = [azurerm_network_interface.weu-vm1-ni1.id]
  size                  = "Standard_DS1_v2"

  os_disk {
    name                 = "weu-vm1-os-disk"
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }

  computer_name                   = "weu-vm1"
  admin_username                  = "secure-admin"
  admin_password                  = "password123!"
  disable_password_authentication = false
}
```

```terraform title="/terraform/output.tf"
output "azurerm_public_ip" {
    value = "${azurerm_public_ip.weu-vm1-ip.*.ip_address}"
}
```

All the terraform code does is creating a resource group, virtual network, network security group, an virtual machine which is connected to the virtual network with a public ip-address assigned to it. Once all the cloud resources are deployed the public ip-address of the virtual machine is printed to our terminal.

## Conclusion

Suprisingly, Terraform is a game-changer for engineers that works with deploying cloud resources as it allows us to quickly deploy and destroy our cloud resources and if something goes wrong with our environ-ment we can quickly roll back to the old version instead of having business down-time.