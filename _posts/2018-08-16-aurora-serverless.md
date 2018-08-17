---
layout: single
classes: wide
title: "Migrating from and RDS Postgres Instance to Aurora Serverless"
description: "Updating Soup Bot to use the now publicly available Aurora Serverless as its data store."
categories:
  - Projects
---

Amazon has now opened up [Serverless Aurora](https://aws.amazon.com/rds/aurora/serverless/) publicly. This means [Soup Bot](/projects/2018/05/07/Soup-Bot.html) can follow the trend of serverless culture and free itself from the shackles of its [AWS Lambda function to handle starting and stopping an RDS instance](https://github.com/ciferkey/soup-bot/blob/master/provision.py) and instead use the new on demand Aurora cluster. This is an especially timely change since Daves Fresh Pasta has stopped serving soup recently so now Soup Bot is not only serverless but also "soupless".

The big promise of Serverless Aurora is that it will only charge you for on-demand usage and handle spinning up and down the compute resources for you. There are some cost differences however which may not make this an ideal choice if you are on the super low usage side like Soup Bot (running for only roughly five hours each week). 

Some differences that stood out to me:
- RDS instances must provision a minimum of 20gb of storage even if you usage is much lower. This currently accounts for most of my current costs for running Soup Bot. Aurora Serverless does not have provisioned space, only used space so this is nice for a small databases. Thus the "$0.115 per GB-month of provisioned GP2 storage" for the RDS instance becomes "$0.10 per GB-month of consumed storage".
- RDS instances take a longer time to start up (on the order of a couple minutes). Thus means the provisioning script is set to run 15 minutes before Soup Bot starts up. Aurora Serverless obviates this.
- RDS instances can be setup with 1 vCPU/1 GiB memory ($0.018 per RDS db.t2.micro instance hour). Aurora Serverless allows a minimum capacity of 2 "capacity units" (sometimes they also seem to talk about "Aurora Compute Units" or "ACU") where each compute unit is 2gb of ram ($0.12 per compute hour, $0.10 per GB-month or storage).
- Aurora Serverless does not allow public access for the database so I had to [use cloud 9](https://aws.amazon.com/getting-started/tutorials/configure-connect-serverless-mysql-database-aurora/) if I wanted to query the database.
- Aurora Serverless does not allow creating a database during cluster setup like you would with a regular RDS instance. Instead it can be created by using the command line in cloud9 or programatically ([sqlalchemy-utils](https://sqlalchemy-utils.readthedocs.io/en/latest/) provides a create_database method)

I would also like to note the change is not a completely perfect comparison since Soup Bot was previously using a Postgres instance on RDS and Aurora Serverless currently only supports MySQL (Postgres support is coming later). This mean I had to [switch the engine](http://docs.sqlalchemy.org/en/latest/core/engines.html#mysql) to mysql-connector-python and handle some differences between Postgres and MySQL (mainly the fact that I could not use the URL as as primary key due to the URLs being too long).

The only difficulty I encountered was that I needed to make sure the Aurora instance was created in the same security group as the lambda function, and that I needed to updated the inbound rules on the security group to allow the correct ports for accessing MySQL (since it had previously been configured to only allow Postgres through). Once those changes were made the lambda was able to access the new database just the same as the old database.
