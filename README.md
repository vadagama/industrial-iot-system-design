- [Purpose of the document](#purpose-of-the-document)
- [Context](#context)
- [Project objectives](#project-objectives)
- [Description of the existing solution](#description-of-the-existing-solution)
- [Functional requements](#functional-requements)
- [Non-functional requirements](#non-functional-requirements)
- [C4 Model context diagram To Be](#c4-model-context-diagram-to-be)
- [C4 Model container diagram To Be](#c4-model-container-diagram-to-be)
- [Backend architecture](#backend-architecture)
- [Data model](#data-model)
- [Frontend architecture](#frontend-architecture)
- [Solution Deployment](#solution-deployment)
- [Observability](#observability)
  - [Monitoring](#monitoring)
  - [Alerting](#alerting)
  - [Logging](#logging)
- [Network architecture](#network-architecture)
- [Architecture Decision Records](#architecture-decision-records)
  - [ADR-1 Microservices vs Monolith](#adr-1-microservices-vs-monolith)
  - [ADR-1 Selecting a DBMS for storing operational information](#adr-1-selecting-a-dbms-for-storing-operational-information)


# Purpose of the document
This document contains the system design of the IIoT (Industrial Internet of Things) system.
The Document contains descriptions of:
- The overall context of the solution
- Architecture of the existing solution
- Functional requirements
- Non-functional requirements
- Architecture of the solution being developed
- Description of integrations with related systems
- Logging and monitoring
- Software requirements
- Hardware requirements
- Network architecture
- Architecture decision records

# Context
At present, the process of monitoring the situation at the main production sites and information support for decision-making in the field of ensuring optimal equipment condition is carried out by the monitoring service on the basis of information from automated systems in the production facilities on the parameters of equipment and technological processes.

Parameter data from automated systems and other sources are transferred to a centralised database via software data collectors. The main method of equipment condition control is parametric monitoring by two-zone static setpoints based on passport characteristics and expert data. Operational personnel of the situation centre, when receiving alarm messages, act according to the situation, based on their individual experience. The experience of each employee is not digitised and is not transferred from one to another, which brings significant losses in the quality of work to prevent equipment failures, in particular, there is no stability in the quality of work with parameters - it is impossible to predict whether the personnel can cope with the prevention of downtime or not in a particular situation.

# Project objectives
- Monitoring and analysing equipment operation.
- Detection of deviations of operating modes from the set parameters at an early stage and timely taking measures to eliminate or reduce the risk of failure of subdivision equipment, which may lead to financial, production losses, reduction of overall equipment efficiency, environmental violations or creation of unsafe working conditions for equipment and personnel.
- Improving the quality and efficiency of decisions to eliminate and prevent failures and critical malfunctions through the use of tools for automated complex analysis of information from various sources, using methods of in-depth diagnostics of equipment condition, mathematical modelling and predictive analytics.


# Description of the existing solution
Currently the solution works as follows:
- Producers write the signals required for projects from the Kafka OPC servers to the storage of the consumer systems. The number of signals and consumers is constantly growing. The existing database is already heavily loaded, the accepted SLAs are not met, and there is no unified solution architecture.
- Only the information required for ML projects is transferred to DataLake.
- There is no single place to store the data archive.
- There is no common tag naming rule, tag descriptions are not maintained or are maintained locally at production sites. There is no single directory of all signals available for collection.
- When a new need arises, data collection requests are created. The collection process is not standardised.

![As Is Architecture](images/as_is.png)

# Functional requements

The system is structured on a modular basis to allow for modernisation, scaling in terms of data sources and expansion of functionality without redesigning the entire system.
List of the main functional modules with a description of their purpose:

| Module                                                 | Purpose                                                                                                                                           |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Monitoring of equipment condition parameters           | Identification of possible malfunctions of production equipment                                                                                   |
| Assessment of the technical condition of the equipment | Analysing data on the equipment                                                                                                                   | condition, determining the causes of malfunctions and failures |
| Alarm management                                       | Taking corrective actions to return the equipment to a serviceable state and reduce material costs.                                               |
| process efficiency assessment                          | Analysing data on the efficiency of personnel actions to prevent equipment failures and ASMD operation (reporting set)                            |
| Configurator                                           | Configuration of objects and modules                                                                                                              |
| Predictive analytics                                   | Early detection of hidden defects on equipment                                                                                                    |
| Power supply                                           | Identification of possible risks for the operation of production equipment related to the operation and condition of power supply system elements |

# Non-functional requirements

| Quality attribute                                                         | Value             |
| ------------------------------------------------------------------------- | ----------------- |
| Time to restore from backup                                               | 72h               |
| Service recovery time                                                     | 8h                |
| Backup retention time                                                     | 7d                |
| Allowable data loss time                                                  | 24h               |
| Availability                                                              | 99%               |
| Number of data sources (parameters) currently in the centralised database | 15000             |
| Number of acknowledgements per month                                      | 2100              |
| Number of actuations per month                                            | 5500              |
| Planned number of registered users until                                  | 120               |
| Planned number of users working simultaneously in the system              | 10                |
| Planned data capacity (GB)                                                | 2466              |
| Estimated data volume growth per month  (MB)                              | 300               |
| Projected number of data sources (parameters) until                       | 50000             |
| Performance (1st screen start)                                            | no more than 1sec |

# C4 Model context diagram To Be

The following diagram shows the C4 model container diagram for the IIoT system.

![Container Diagram](images\IIoT_Context_diagram.jpg)

The implemented solution consists of the following main elements:
1.	**Controllers** is data sources that generate events.
2.** OPC server** is a system that collects and aggregates events from controllers.
3. OPC Producer is a system directing events from OPC servers to DataLake.
5.	**DataLake** is a system that collects and stores all data that is collected by OPC producers. 
1.	**Kafka** is a distributed streaming data collection system.
2. **Hive** is a system for accessing historical Hadoop data.
6.	**Data Governance **is a system in which parameters are mapped to aggregates and tag enrichment with additional information is provided.
7.	**Microsoft Active Directory enterprise directory service** - a system that contains information about system users and their groups.
8.	**Production Management System** is designed to help employees to get answers to the questions: what we produce, where, how much and of what quality.
9. **Critical equipment identification system** is the system is designed to identify critical equipment, by assessing safety, production and environmental risks, and measuring equipment performance

A description of the interaction of the above elements:
- Data from sensors (controllers) is collected by the OPC Server. No business logic is applied at the controller and OPC Server level to ensure that only valid, complete and non-contradictory data is processed in the following steps.
- Data from the OPC Server is consumed by the OPC Producer, or by the Data Producer and written to a Kafka Topic (approximately 0.2 to 10 messages per second). If it is necessary to realise higher throughput, replication of these components is possible.
- The data from the Kafka topics as well as the Hive tables are transferred to the Monitoring System for further processing.
- The Monitoring System also interacts with Data Governance, Active Directory, Metal Tracking System, CCTV, etc.
- Data Governance - obtaining a valid data model of aggregates, their properties and associated tags
- Active Directory - corporate directory service contains up-to-date information about users of the corporate network (account, full name, email, groups, etc.)

# C4 Model container diagram To Be

The following diagram shows the C4 model container diagram for the IIoT system.

![Container Diagram](images\IIoT_Container_diagram.jpg)


At the level of the monitoring system itself, the following structure of containers is proposed:
1. **frontend** - user interface for data visualisation and execution of user queries (including the formation of directories).
2. **core** - API for asmd-frontend, as well as a layer of business logic.
3. **etl-kafka** - ETL layer for processing message threads from Kafka topics.
4. **etl-dg** - ETL layer for getting up-to-date directories from Data Governance.
5. **db** - a layer for storing data (database).
6.	**Data Governance** - a system providing up-to-date information about parameters and technical locations.
7.	**Kafka** - a streaming platform that collects events from sensors via OPC-Servers.

Description of the container-level solution:
1.	The users of the system through a web browser send a request to the frontend, which is an HTTP server and a reverse proxy server. 
2. frontend through REST API requests interacts with asmd-core to retrieve data from asmd-db and also execute various business logic.
3.	The etl-kafka service is used to process events from the Kafka streaming platform, which in turn stores events from OPC producers in tops.
4.	Events undergo various checks in etl-kafka according to specified business rules (setpoint checking, complex rules). 
5.	Also events processed in etl-kafka are stored in PostgreSQL database (db). 
6. core interacts with the db database both read and write: receiving data for displaying on the UI, receiving directories, changing business rules.
7.	The directories of technical places and parameters with a set of attributes come from the Data Governance system. Integration is realised through etl-dg microservice, which accesses tables bd (Data Governance system database) on a schedule, parses the data and saves them in our database db

# Backend architecture
The business logic layer is implemented using the following technologies:
- Spring - https://spring.io/ 
- Spring Framework Boot
- Spring for Apache Kafka
- Spring REST Docs
- Spring Security
- Spring Data
- Liquibase - Open Source Version Control for Database

# Data model
The business logic layer in terms of etl-kafka, etl-dg and core is linked to db in terms of data read/write.
The diagram shows the detailed data model of the system down to the table level with logical relationships between entities.

![Container Diagram](images\db.png)

The data model consists of the following tables:
- **TECH_PLACE** - Directory of Technical Places
- **PARAMETER** - Directory of parameters
- **PARAMETER_TYPE** - Parameter setting type
- **PARAMETER_SERIES** - Parameter values
- **PARAMETER_STATUS** - Parameter values
- **PARAMETER_BORDER** - Parameter Setpoints
- **PARAMETER_BORDER_TYPE** - Type of parameter settings
- **PARAMETER_REASON** - Possible causes of setpoint activation
- **PARAMETER_OUTCOME** - Possible consequences of setpoint operation
- **VIEW_PARAMETER_SYNC_FOR_DG** - Parameter statuses for synchronisation back to DG
The data model also consists of the following views:
- **VW_TREE_DATA** - Tree of technical locations and parameters

# Frontend architecture
The presentation layer is implemented using the following technologies:
- **React library** - React official site
- **TypeScript** - TypeScript official site
- **Ant.Design visual components library** - Ant Design official site

The frontend architecture is implemented according to the Feature-sliced structural methodology

# Solution Deployment
Deployment scheme of the application components:
1. **Development** - system development environment
2. **Stage** - system testing environment
3. **Production** - system operation environment

# Observability

The observability of the system is provided by the following components:
- Monitoring
- Alerting
- Logging

## Monitoring
Prometheus centralised metrics collection system is used for system monitoring. Grafana is used to visualise the metrics. 
The following infrastructure metrics are generated:

| Name                     | Description                                              |
| ------------------------ | -------------------------------------------------------- |
| Uptime                   | Operating time after boot                                |
| CPU                      | Usage CPU Usage                                          |
| Threads                  | Number of CPU threads                                    |
| Thread                   | States State of CPU threads                              |
| JVM Heap                 | State of JVM heap                                        |
| JVM                      | Non-Heap                                                 |
| HTTP Rate                | Frequency of HTTP requests                               |
| HTTP Duration            | Duration of HTTP requests                                |
| HTTP Errors              | Frequency of erroneous HTTP requests                     |
| JDBC Pool                | Number of open database connection pools                 |
| Connections Size         | Number of open database connection pools                 |
| Connection Timeout       | Number of open pools of connections to the database      |
| Connections              | Dynamics of open pools of connections to the database    |
| Connection Creation Time | Dynamics of database connection creation                 |
| Connection Usage Time    | Dynamics of database connection usage                    |
| Connection Acquire Time  | Dynamics of database connections acquired                |
| HTTP CLIENT_ERROR        | Dynamics of erroneous HTTP requests to the service       |
| HTTP INFORMATIONAL       | Dynamics of informational HTTP requests from the service |
| HTTP SUCCESS             | Dynamics of successful HTTP requests from the service    |

The following business metrics are generated:
| Name                                        | Description                                                       |
| ------------------------------------------- | ----------------------------------------------------------------- |
| Topics Number                               | The number of Kafka topics to which asmd_etl_kafka is subscribed. |
| Average Number Of Messages Per Second       | Average number of messages per second for a topic                 |
| Max Number Of Messages Per Topic Per Second | The maximum number of messages per second for a topic.            |
| Total Number Of Users                       | The number of users in the system at the moment                   |

## Alerting

The following alerts are generated:
| Name                 | Description                                                                         | Alarm threshold   |
| -------------------- | ----------------------------------------------------------------------------------- | ----------------- |
| Exceeded CPU Usage   | CPU Usage threshold exceeded                                                        | CPU Usage > 70%   |
| Exceeded HTTP Errors | The threshold for the number of erroneous requests to the service has been exceeded | HTTP Errors > 0   |
| Exceeded JBDC        | Conections Exceeded threshold for number of connectionn                             | Connections > 100 |

## Logging

GrayLog (a service designed to collect, store and process logs from various sources) is used for centralised work with logs.
It allows you to:
- Simplify access to logs of various system modules.
- Ability to build reports and alerts.
- Logs can be viewed in case of failure of the system as a whole or one of the modules.
- In order to be able to track distributed operations on events from Kafka tops, a DataKey entry in the logs is required. This ID should be thrown through the chain of calls within the service or between services.

Log generation requirements
In order to correctly collect logs in Graylog:
- Each system model must output logs to the standard stdout\stderr output streams.
- Each new line must be a separate log message. It is not allowed to output multi-line logs to standard streams.
- It is recommended to output logs in JSON format.

# Network architecture
The project requires good network bandwidth on its various segments as well as a high level of fault tolerance.

| Segment name                                                                                         | Network characteristics and development forecasts   |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| Network segment from data sources to OPC-Servers                                                     | No exact data available                             |
| On-campus data centre                                                                                | 1GB per site                                        |
| Between the Data Centre (including OPC-Servers, IOT-Server) and the Kafka Cluster in the Data Centre | 2 channels from different mobile network operators: |
|                                                                                                      | - 200Mbit with channel load - 40Mbit                |
|                                                                                                      | - Backup channel - 200Mbit                          |
|                                                                                                      | Planned - 500Mbit (internet access)                 |
|                                                                                                      | Fault tolerance: 130 min per month                  |
|                                                                                                      | Latency: 33msec                                     |
| Between Kafka and etl-kafka component                                                                | Throughput - 20GB                                   |
| Between core, etl-kafka, etl-dg and db                                                               | Throughput - 20GB                                   |
| Between etl-dg and db-dg                                                                             | Throughput - 20GB                                   |

# Architecture Decision Records
## ADR-1 Microservices vs Monolith
**Context**

A backend implementation option needs to be defined. The following options are considered:

**1 variant - microservices**

Prerequisites for choosing a microservices architecture:
1.	Parts of the functionality described by the Customer are poorly connected with each other (especially in terms of integration with related systems), so it is reasonable to divide them into separate services
2.	It is easier to scale horizontally when a large load occurs in the part of data processing from Kafka. In the case of our project, a significant increase in load is possible.
3. It is possible to use different stack for different modules. For example, Python is more suitable for ML.
4. Failure of one microservice will not lead to failure of the whole system.

**2 option - modular monolith**

Prerequisites for choosing monolithic architecture:
1.	High load and avalanche growth of data and users is not expected.
2.	DB partitioning is not planned
3.	A plus will be the ease of deploying and configuring the application in the development environment as a single container
4.	No need for very frequent deliveries
5.	Easier to test business logic
6.	Monolith scaling options are also present

**Decision**

As a result of the analysis, the decision was made to apply a microservice Backend architecture, as weak connectivity at the ETL service level is assumed and significant load growth is possible in the future (horizontal scaling will be required). 

**Implications**

Possible risks are related to the complication of system maintenance and higher requirements to the qualification of developers.

## ADR-1 Selecting a DBMS for storing operational information

**Context**

It is necessary to define a DBMS for storing messages from Kafka and system help information.

**1 option - PostgreSQL**

Prerequisites for selecting PostgreSQL:
1.	Parts of the functionality described by the Customer are poorly connected with each other (especially in terms of integration with related systems), so it is reasonable to divide into separate services.
2.	It is easier to scale horizontally when there is a heavy load in the part of data processing from Kafka. In the case of our project, a significant increase in load is possible.
3. It is possible to use different stack for different modules. For example, Python is more suitable for ML.
4. Failure of one microservice will not lead to failure of the whole system.
2 option - Oracle Database
Prerequisites for choosing Oracle DB:
1.	High load and avalanche growth of data volume and users are not expected.
2.	DB partitioning is not planned
3.	A plus will be the ease of deploying and configuring the application in the development environment as a single container
4.	No need for very frequent deliveries
5.	Easier to test business logic
6.	Monolith scaling options are also present

**Decision**

As a result of the analysis, it is decided to choose Oracle Database because of the rich in-house expertise and good support. In addition, Oracle DB will provide all the required DBMS functionality for our project.

**Implications**

Possible risks are related to the peculiarities of Oracle technologies.