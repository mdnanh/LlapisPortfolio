---
title: "Streaming E-Commerce Data Engineering"
date: 2023-10-01
draft: false
author: "Llapis"
tags:
  - Apache Spark
  - Apache Druid
  - Apache Kafka
  - MiniO
description: "Pipeline processes real-time data for e-commerce platforms."
socialShare: False
---

[![Airflow](https://img.shields.io/badge/Airflow-2.9.3-lightgrey)](https://airflow.apache.org/docs/)
[![Kafka](https://img.shields.io/badge/Kafka-lastest-black)](https://kafka.apache.org/documentation/)
[![Druid](https://img.shields.io/badge/Druid-30.0.0-lightblue)](https://druid.apache.org/docs/latest/design/)
[![MiniO](https://img.shields.io/badge/MiniO-lastest-darkred)](https://min.io/docs/minio/kubernetes/upstream/)
[![Postgres](https://img.shields.io/badge/Postgres-14alpine-blue)](https://www.postgresql.org/)
[![Superset](https://img.shields.io/badge/Superset-lastest-darkblue)](https://superset.apache.org/docs/intro)


## Overview

The **Streaming Ecommerce** project is an innovative solution that integrates **streaming data processing** with **ecommerce platforms**. It aims to deliver real-time insights and actions for enhancing user experience and improving sales performance. By utilizing **Apache Kafka**, **Apache Druid**, and **MiniO** for storage, the project processes massive amounts of user activity and transaction data in real time.

![all_text](/images/projects/streaming/viz.jpg)

## Features

- **Real-Time Data Processing**: Process customer actions, product views, and transactions in real time.
- **Ecommerce Data Integration**: Streamline customer behavior analysis and product performance tracking.
- **Data Analytics**: Generate actionable insights for marketing and sales strategies.

## Architecture
![alt text](/images/projects/streaming/architecture.png)

## Technologies Used
- **Apache Kafka**  
  [Kafka](https://kafka.apache.org/documentation/) is a distributed messaging platform that allows you to sequentially log streaming data into topic-specific feeds, which other applications can consume. In this project, Kafka is used to collect and buffer events that are then ingested by Druid for real-time analytics and querying.
- **Apache Druid**  
  [Apache Druid](https://druid.apache.org/docs/latest/design/) provides low-latency real-time data ingestion from Kafka, flexible data exploration, and rapid data aggregation. Druid is designed to handle high throughput and allows for real-time querying of data, unlike traditional data lakes. It is referred to as a "data river," meaning that data is continually processed and made available in real-time. Druid provides a significant speed improvement over Apache Hive for real-time data processing, making it ideal for this project.
- **Apache Superset**   
  [Apache Superset](https://superset.apache.org/docs/intro/) is an open-source data visualization tool that enables graphical representation of data. It supports various databases through SQLAlchemy, and in this project, it is used to visualize real-time data and analytics, helping stakeholders make informed decisions.
- **Apache Airflow**  
  [Airflow](https://airflow.apache.org/docs/) is a task scheduling platform that allows you to create, orchestrate, and monitor data workflows. In this project, Airflow is used to schedule and manage tasks, such as sending data to Kafka topics for further processing.
- **MinIO**  
  [MinIO](https://min.io/docs/minio/kubernetes/upstream/index.html) is a high-performance, distributed object storage system that is compatible with Amazon S3. It provides scalable and durable storage for large datasets. In this project, MinIO is used to store the incoming stream data for later retrieval and processing. Its compatibility with S3 APIs makes it easy to integrate with existing cloud storage systems.
- **Docker**  
  [Docker](https://www.docker.com/) is used for containerizing the entire application, ensuring consistent environments across development, testing, and production. It allows the various components of the system (Kafka, Druid, Airflow, MinIO, etc.) to run in isolated containers, making the deployment process more streamlined and scalable.

## Init

All code has been uploaded repo [Github](http://localhost:1313/projects/streamingrealtime/).

### 1. Installation

This project is built with docker compose, so make sure you have docker and docker-compose installed. Follow the steps instructed in [Docker](https://docs.docker.com/get-started/get-docker/) to install it. 

Then, pull this repo and start the journey.

**NOTE: Data collected from [Tiki](https://tiki.vn/)**

### 2. Start services 🕸️

```sh
cd StreamingEcommerceDE
```

* For the first time, start the MiniO service first to initialize the deepstorage containing the streaming data. Change two param `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` by yourself in file `docker-compose.yml` at the block `minio` service. Then, run command:

```sh
docker-compose up -d minio
```
* Follow this [step-by-step to run Druid with deepstorage Minio](https://blog.min.io/how-to-druid-superset-minio/) to clearly. 

Now, Minio server is running on port `localhost:9001`, login to the server and create a `druidbucket` to store segments and indexing_logs, create a service account (under the Identity menu), edit the user policy to only allow access to `druidbucket` and in the Druid configuration (`./druid/environment`) below use the service account’s `access key` and `secret key`.

* Then, run all the services with command:
```sh
docker-compose up -d
```
* The `USER` and `PASSWORD` of some services are configured in `docker-compose.yml`, Apache Airflow's password is provided in `airflow/standalone_admin_password.txt`.
#### SERVER
Service | URL |
--- | --- |
MiniO | http://localhost:9001 |
Apache Druid | http://localhost:8888 |
Apache Superset | http://localhost:8088 |
Apache Airflow | http://localhost:8080 |

### 3. Streaming data to Druid ♾️
* The file `KafkaProducerEcomm.py` send a message demo data to Kafka `Ecommerce` topic every second with fake transaction data, the structure of data message as below:

```code
{
      'id': 274992707,
       'name': 'Hộp Cơm 3 Tầng Lunch Box Kèm Muỗng Đũa Và Túi Đựng Tiện Lợi',
       'brand_name': 'PEAFLO',
       'price': 192000,
       'Origin': 'Hàn Quốc / Trung Quốc',
       'category': 'Dụng cụ chứa đựng thực phẩm',
       'original_price': 235000,
       'discount': 43000,
       'discount_rate': 18,
       'purchase_id': 'P17281500820277577',
       'quantity': 5,
       'city': 'Quảng Ninh',
       'code_city': 'VN-13',
       'create_at': '2024-10-06 00:41:22'
}
```

* From Druid load data from Kafka `kafka:9092`, choice `Ecommrce` topic and config data result table.

![all_text](/images/projects/streaming/druid_connect.gif)

* For more infomation, reach [github](https://github.com/apache/druid?tab=readme-ov-file) and about configure ingest data process, reach [Ingestion overview](https://druid.apache.org/docs/latest/ingestion/index.html).

### 4. Visualization 💹
* From Superset server add Druid database with the sqlalchemy uri:

```code
druid://broker:8082/druid/v2/sql/
```

* More detail at [Connecting to Databases](https://superset.apache.org/docs/configuration/databases/)
* Create dashboard with amazing chart from `Ecommerce` table

![all_text](/images/projects/streaming/viz.jpg)

🔥🔥🔥                                                            🤝🤝🤝                                                                  🔥🔥🔥


