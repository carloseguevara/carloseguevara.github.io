---
title: "Apply Domain Drive Development in ETL"
layout: post
author: "Carlos Guevara"
header-style: text
tags:
- Python
- DataEngineering
---

Domain:
All ours data models
Interfaces

Infrastructure:
Here we have the most confuse point. In general infrastructure represent the use of third-party framework, that could be
pandas or pyspark. But in this context, infrastructure will be our storages solutions and the data sources.
For example, in terms of storage solutions, if we have one ETL that will write data in some place, we could
have `S3Repository`, `DynamoRepository`, `RedshiftRepository`.
If we need client's data, we could have `SalesforceRepository`, `CRMRepository` and so on.

With this we could avoid to have coupling to the data storage solution and data source, with an easy way to switch between them.

But, what happen if I want to change the technology used to have in memory the information. For example, if we have an
important growing of data size, and we need to change pandas to pyspark.

Well, in general this represents more changes that only use one library instead of others. We maybe need to change the
cloud solution when our ETL are running, add initialize code for the new framework, change
