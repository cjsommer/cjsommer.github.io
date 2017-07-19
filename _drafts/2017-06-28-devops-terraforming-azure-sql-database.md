---
layout: post
title: DevOps - Terraforming Azure SQL Database
date: 2017-06-28 00:00
author: cjsommer@gmail.com
comments: true
categories: [SQL Server]
---

Configuring my system
* Register for an Azure account
* Install Terraform - https://www.terraform.io/downloads.html
* Install Azure CLI - https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
* Configure Azure credentials for Terraform (using Azure CLI)
```
az login
az account list
az ad sp create-for-rbac --role="Contributor" --scope="/subscriptions/<subscription-id>"
az login --service-principal -u <appId> -p <password> --tenant <tenantId>
```
* Create a resource group in Azure
* Create a SQL Server
* Create a SQL Database
* Create a firewall rule so you can access it

Terraform docs are great references for all of the providers