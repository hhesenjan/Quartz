---
title: "Setting up SentinelOne Data Connector for Microsoft Sentinel"
created: 2026-09-18
modified: 2026-09-18
tags: ["SIEM", "SOC", "SENTINELONE"]
draft: false
---

## Introduction

Microsoft Sentinel has a SentinelOne Data Connector which can be used for retrieving data from different REST endpoints in SentinelOne. This allows us to store logs inside of Microsoft Sentinel which the SOC vendor can use to respond to security incidents. In this article, I'll go through setting up the SentinelOne Data Connector.

## V1 .vs V2 Data Connector

The **SentinelOne V2 Data Connector** comes with support for **Unified Alert Management (UAM)** and **Wayfinder**. While **SentinelOne V2 Data Connector** doesn't come with support for these different capabilities. And migrating from V1 to V2 shouldn't break anything since `SentinelOneAlertsV2_CL` maps the same columns so all the analytics rules, hunting query, and workbooks works without needing any changes.

## Setting up Service User

1. Go to **SentinelOne -> Policies and Settings**.

   ![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 01.png]]
2. Click on "New Service User".

	![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 02.png]]
3. Enter the **Service Name, Description, and Expiration Date**.

	![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 03.png]]
4. Copy the **API Key** for the **Service User**.

	![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 04.png]]

## Setting Up Data Connector

1. Go to **Microsoft Sentinel -> Content Management -> Content Hub**.

	![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 05.png]]
2. Go to **Microosft Sentinel -> SentinelOne V2 (via Codeless Connector Framework) (Preview)**.

	![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 06.png]]
3. Click on **"Add new instance"**. 

   ![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 07.png]]
4. Enter the **Management URL, and API Key from Service User**.

	![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 08.png]]
5. The **SentinelOne Data Connector** should be active now.

	![[0056 Setting up SentinelOne Data Connector for Microsoft Sentinel 09.png]]

Once the SentinelOne Data Connector is setup it should take anywhere from 5 to 15 minutes before the logs arrives into the different tables.

## Conclusion

The SentinelOne V2 Data Connector requires us to create a Service User with viewer only permissions and from there the SentinelOne V2 Data Connector can be setup using the **Management URL** and **API Key for Service User**. You can read more about SentinelOne V2 Data Connector through [GitHub](https://github.com/Azure/Azure-Sentinel/blob/master/Solutions/SentinelOne/Data%20Connectors/README.md).
