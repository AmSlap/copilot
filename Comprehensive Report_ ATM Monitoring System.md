# Comprehensive Report: ATM Monitoring System

**Date:** May 29, 2025

**Version:** 1.0

---

## 1. Introduction

### 1.1. Project Vision and Goals

The primary vision for the ATM Monitoring System project is to establish a robust, real-time platform for overseeing a fleet of Automated Teller Machines (ATMs). The core goals include:

*   **Enhanced Availability:** Minimize ATM downtime by proactively detecting and addressing issues.
*   **Streamlined Maintenance:** Automate incident creation and management through defined workflows, improving response times and efficiency.
*   **Real-time Visibility:** Provide operators and managers with immediate insights into ATM status, hardware health, and cash levels.
*   **Data-Driven Decisions:** Enable better operational planning through comprehensive reporting and analytics on ATM performance and incident trends.
*   **Modern Architecture:** Leverage a scalable and resilient microservices architecture to support future growth and adaptability.

### 1.2. Scope of the System

The system encompasses the end-to-end monitoring and management lifecycle for ATMs, including:

*   Ingestion and processing of real-time data streams from ATMs (status, configuration, counters, transactions).
*   Management of static ATM information (registry).
*   Tracking of dynamic ATM states (operational status, hardware health, cash levels).
*   Automated alerting based on predefined rules and thresholds.
*   Workflow-driven incident management (ticketing, assignment, tracking, resolution) based on maintenance or insurance contracts.
*   Reporting and analytics capabilities.
*   A secure API layer for integration and user interface access.
*   User management and authentication.

### 1.3. Purpose of the Report

This document serves as a comprehensive overview of the ATM Monitoring System project. It consolidates key information regarding the system's architecture, functional components (microservices), operational workflows, project planning artifacts (backlogs), and initial implementation details. It is intended to provide stakeholders with a thorough understanding of the system design and development progress.

---



## 2. System Architecture

### 2.1. Architecture Style

The system employs a **Microservices Architecture**. This style was chosen to promote:

*   **Scalability:** Individual services can be scaled independently based on load.
*   **Resilience:** Failure in one service is less likely to impact others.
*   **Maintainability:** Smaller, focused codebases are easier to understand, modify, and test.
*   **Technology Diversity:** Different services could potentially use different technologies best suited for their task (though this project standardizes on Java/Spring Boot).
*   **Independent Deployability:** Services can be updated and deployed without requiring a full system redeployment.

Communication between services is primarily event-driven, utilizing **Apache Pulsar** as the message broker for asynchronous data flow. Synchronous communication for direct requests (e.g., from the UI or between specific services) is handled via REST APIs exposed through an **API Gateway**.

### 2.2. High-Level Architecture Overview

The system consists of several layers: data sources (ATMs), a data infrastructure layer (Pulsar, Databases), an application layer (microservices and API Gateway), and external interfaces (UI, Notification Systems).

*(Refer to the diagram in `system_architecture.puml` for a visual representation)*

Key interactions include:

1.  **Data Ingestion:** ATMs send raw data to Pulsar. The Data Ingestion Service consumes, processes, and publishes standardized events back to internal Pulsar topics.
2.  **Event Processing:** Dedicated microservices (Status, Configuration, Counter, Alerting, Reporting) consume these internal events to update state, trigger alerts, or perform analysis.
3.  **API Access:** A User Interface or external systems interact with the platform via the API Gateway, which routes requests to the appropriate microservices.
4.  **Incident Management:** The Alerting Service can trigger the Incident Management Service, which uses jBPM to orchestrate predefined resolution workflows.
5.  **Data Storage:** Services persist their state in dedicated databases (Operational PostgreSQL DB for registry, incidents, users; Time-Series DB for status, configuration, counters).

### 2.3. Core Technologies

*   **Backend Framework:** Java 17, Spring Boot 3.x
*   **Messaging Broker:** Apache Pulsar 3.x
*   **Workflow Engine:** jBPM
*   **Databases:** PostgreSQL (Operational Data), TimescaleDB (Time-Series Data - Recommended)
*   **API Gateway:** Spring Cloud Gateway
*   **Containerization:** Docker
*   **Orchestration (Recommended):** Kubernetes

### 2.4. Best Practices Incorporated

The architecture incorporates several best practices for distributed systems:

*   **Database per Service:** Each microservice manages its own data.
*   **Asynchronous Communication:** Event-driven flow via Pulsar enhances decoupling and resilience.
*   **API Gateway:** Centralizes access control, routing, and cross-cutting concerns.
*   **Stateless Services:** Where feasible, services are designed stateless for scalability.
*   **Externalized Configuration:** Configuration managed outside the application code.
*   **Service Discovery:** Mechanisms for services to find each other dynamically.
*   **Resilience Patterns:** Use of retries, timeouts, and potentially circuit breakers.
*   **Observability:** Emphasis on logging, metrics, and tracing (though implementation details are outside the current scope).
*   **Security:** Secure endpoints and communication channels.

---



## 3. Microservice Components

This section details the individual microservices that constitute the application layer of the ATM Monitoring System.

*(Refer to the diagram in `component_diagram.puml` for a visual representation of component interactions)*

### 3.1. ATM Data Ingestion Service
*   **Responsibility:** Consumes raw data streams (status, configuration, counters, transactions) published by ATMs to Apache Pulsar topics. Validates, parses, and potentially enriches the data before publishing standardized events to internal Pulsar topics for downstream processing.
*   **Technology:** Java Spring Boot, Apache Pulsar Client.
*   **Communication:** Consumes from external Pulsar topics, publishes to internal Pulsar topics.
*   **Data Storage:** Primarily stateless processing; no persistent storage.

### 3.2. ATM Registry Service
*   **Responsibility:** Manages the master data for each ATM, including GAB Number, Serial Number, Brand, Model, Label, IP Address, Region, Agency, and Responsible Person details.
*   **Technology:** Java Spring Boot, Relational Database (PostgreSQL recommended).
*   **Communication:** Provides REST APIs for CRUD operations on ATM data. May consume events for specific updates.
*   **Data Storage:** Relational database (Operational DB).

### 3.3. ATM Status Service
*   **Responsibility:** Tracks the real-time and historical operational status of ATMs (offline, out of service, supervisor mode, alert, in service), maintenance status, last connection date, and last transaction date.
*   **Technology:** Java Spring Boot, Time-Series Database (TimescaleDB recommended) or Relational DB.
*   **Communication:** Consumes `AtmStatusUpdated` events from Pulsar. Provides REST APIs for querying current/historical status. Publishes significant status change events (e.g., `AtmOfflineEvent`).
*   **Data Storage:** Time-Series Database (or Operational DB).

### 3.4. ATM Configuration Service
*   **Responsibility:** Monitors the status of individual ATM hardware peripherals (Cassettes, Reject Bin, Safe, Keyboard/EPP, Card Readers, UPS, Camera, Encryptor, Printers, Journal, Dispenser, etc.). Assigns health status codes/colors (Red, Orange, Green, Gray).
*   **Technology:** Java Spring Boot, Time-Series Database (TimescaleDB recommended) or Relational DB.
*   **Communication:** Consumes `AtmConfigurationChanged` events from Pulsar. Provides REST APIs for querying hardware status. Publishes `HardwareFailureEvent` when issues are detected.
*   **Data Storage:** Time-Series Database (or Operational DB).

### 3.5. ATM Counter Service
*   **Responsibility:** Tracks cash levels for each cassette: available notes, total amount, denomination, and cassette status (OK, empty, problem, not configured).
*   **Technology:** Java Spring Boot, Time-Series Database (TimescaleDB recommended) or Relational DB.
*   **Communication:** Consumes `AtmCountersUpdated` events from Pulsar. Provides REST APIs for querying cash levels. Publishes `LowCashEvent`, `CassetteEmptyEvent`, etc.
*   **Data Storage:** Time-Series Database (or Operational DB).

### 3.6. Incident Management Service
*   **Responsibility:** Manages the full lifecycle of ATM incidents using jBPM workflows. Handles ticket creation (manual/automatic), assignment, tracking, SLA monitoring, and resolution based on defined processes (Maintenance Contract, Insurance, etc.).
*   **Technology:** Java Spring Boot, jBPM Engine, Relational Database (PostgreSQL recommended).
*   **Communication:** Provides REST APIs for incident management. Consumes triggers from the Alerting Service. Interacts with the jBPM engine.
*   **Data Storage:** Relational database (Operational DB) for incident details and workflow state.

### 3.7. Alerting Service
*   **Responsibility:** Consumes events from Status, Configuration, and Counter services. Applies rules to detect conditions requiring alerts. Sends notifications via configured channels (Email, SMS, UI) and triggers incident creation in the Incident Management Service for critical alerts.
*   **Technology:** Java Spring Boot, potentially a Rules Engine (e.g., Drools).
*   **Communication:** Consumes events from Pulsar. Publishes notifications. Calls Incident Management Service API.
*   **Data Storage:** Optional database for tracking generated alerts.

### 3.8. Reporting Service
*   **Responsibility:** Provides aggregated reports and analytics on ATM availability, performance, incident statistics, SLA compliance, etc., by gathering data from other services or dedicated data stores.
*   **Technology:** Java Spring Boot, potentially data warehousing tools.
*   **Communication:** Provides REST APIs for accessing reports. Queries other services or reporting databases.
*   **Data Storage:** May utilize its own reporting database or data warehouse.

### 3.9. API Gateway
*   **Responsibility:** Single entry point for external clients. Handles routing, authentication, rate limiting, SSL termination, and other cross-cutting concerns.
*   **Technology:** Spring Cloud Gateway (or similar like Kong, Apigee).
*   **Communication:** Routes external requests to internal microservices.

### 3.10. User Management / Authentication Service
*   **Responsibility:** Manages user accounts, roles, permissions. Handles user authentication (e.g., login) and issues security tokens (e.g., JWT) for authorization.
*   **Technology:** Java Spring Boot, Spring Security, OAuth2/JWT (potentially integrated with an Identity Provider like Keycloak).
*   **Communication:** Provides authentication endpoints. Interacts with the API Gateway and other services for authorization checks.
*   **Data Storage:** Relational database (Operational DB) for user credentials and roles.

---



## 4. Key Workflows

This section describes the primary operational workflows within the ATM Monitoring System.

### 4.1. Data Ingestion Flow

This flow describes how data originating from the ATMs is processed and made available within the system:

1.  **Transmission:** Individual ATMs in the fleet publish their status, configuration, counter, and transaction data as messages to predefined, specific topics within the Apache Pulsar cluster.
2.  **Consumption:** The `ATM Data Ingestion Service` subscribes to these raw data topics in Pulsar.
3.  **Processing:** Upon receiving a raw message, the Ingestion Service:
    *   Validates the message format and structure.
    *   Parses the content to extract relevant data fields.
    *   Potentially enriches the data (e.g., adding standardized timestamps).
4.  **Internal Publishing:** The Ingestion Service publishes distinct, processed event messages (e.g., `AtmStatusUpdated`, `AtmConfigurationChanged`, `AtmCountersUpdated`, `AtmTransactionRecorded`) to dedicated internal Pulsar topics. These events follow a standardized schema (e.g., Avro or JSON).
5.  **Downstream Consumption:** Relevant microservices subscribe to these internal topics:
    *   `ATM Status Service` consumes `AtmStatusUpdated` events.
    *   `ATM Configuration Service` consumes `AtmConfigurationChanged` events.
    *   `ATM Counter Service` consumes `AtmCountersUpdated` events.
    *   `Alerting Service` and `Reporting Service` may consume multiple event types.
6.  **State Update:** Each consuming service updates its internal state (database records) based on the received event data.

### 4.2. Alerting and Incident Trigger Flow

This flow outlines how potential issues are detected and escalated:

1.  **Issue Detection:** An event processing service (`ATM Status`, `Configuration`, or `Counter`) detects a condition that might warrant attention based on incoming data (e.g., status changes to 'offline', a critical hardware component reports an error, a cash cassette becomes empty).
2.  **Event Publication:** The detecting service publishes a specific, granular event message (e.g., `AtmOfflineEvent`, `HardwareFailureEvent`, `LowCashEvent`, `CassetteEmptyEvent`) to a designated internal 'events' topic in Pulsar.
3.  **Alert Evaluation:** The `Alerting Service` consumes these granular events.
4.  **Rule Application:** The Alerting Service applies predefined rules (which can range from simple thresholds to more complex logic) to the event data to determine if an alert should be generated.
5.  **Notification:** If an alert condition is met, the `Alerting Service` sends notifications through configured channels (e.g., Email to the responsible person, SMS to a maintenance group, updates to a UI dashboard).
6.  **Incident Creation (Optional):** Based on the severity defined in the alerting rules, the `Alerting Service` can make a synchronous API call to the `Incident Management Service`'s REST endpoint to automatically create a new incident ticket, passing relevant details from the triggering event.

### 4.3. Incident Management Workflow (jBPM)

This workflow details how incidents are managed using the jBPM engine:

1.  **Incident Creation:** An incident ticket is created in the `Incident Management Service`, either automatically via an API call from the `Alerting Service` or manually through a user interface interacting with the Incident Management API.
2.  **Workflow Initiation:** Upon incident creation, the `Incident Management Service` identifies the appropriate workflow definition (based on incident type, ATM contract status, etc.) and initiates a new jBPM process instance, passing necessary data (like ATM ID, issue description) as process variables.
3.  **Process Execution:** The jBPM engine takes control and executes the steps defined in the corresponding BPMN 2.0 model. The specific path depends on the workflow type (e.g., Sous Contrat, Hors Contrat/Assurance).
4.  **Task Assignment:** The workflow assigns human tasks (e.g., 'Diagnose Issue', 'Dispatch Technician', 'Validate Repair') to specific user roles or groups (Helpdesk, Centre Suivi GAB, Fournisseur, Validator).
5.  **User Interaction:** Users interact with the system (likely via a UI calling the Incident Management API) to view their assigned tasks, claim tasks, provide necessary input (comments, repair details, costs), and complete tasks.
6.  **Service Tasks:** The workflow may include automated service tasks executed by the jBPM engine, potentially calling other microservice APIs if needed.
7.  **State Tracking:** The `Incident Management Service` listens for events from the jBPM engine or queries its state to update the incident record in its database, reflecting the current workflow status, task assignments, and captured timestamps (Date ouverture/fermeture for different stages).
8.  **SLA Monitoring:** Timers and logic within the workflow (or the Incident Management Service) track progress against defined Service Level Agreements (SLAs) or Operational Level Agreements (OLAs).
9.  **Resolution & Closure:** Once the workflow reaches an end state (e.g., 'Resolved', 'Closed', 'Rejected'), the incident ticket is updated accordingly.

### 4.4. Sequence Diagram Example: Hardware Failure

The following sequence illustrates the interactions for a typical hardware failure event:

*(Refer to the diagram in `sequence_diagram_hw_failure.puml` for a visual representation)*

1.  An ATM detects a printer failure and sends a configuration message to Pulsar.
2.  The `Data Ingestion Service` consumes the raw message, processes it, and publishes a `ConfigurationEvent` to an internal Pulsar topic.
3.  The `ATM Configuration Service` consumes this event, updates the printer status to 'ERROR' in its database (likely the Time-Series DB), and publishes a `HardwareFailureEvent` to another internal topic.
4.  The `Alerting Service` consumes the `HardwareFailureEvent`, evaluates its rules, determines an alert and incident are needed.
5.  The `Alerting Service` sends an email/SMS notification via the `Notification Service`.
6.  The `Alerting Service` calls the `Incident Management Service` API to create an incident.
7.  The `Incident Management Service` creates the incident record and starts the appropriate jBPM workflow instance.

---



## 5. Project Backlog

This section provides an overview of the project planning artifacts, including the overall Product Backlog and an example Sprint Backlog.

### 5.1. Product Backlog Overview

The Product Backlog captures the complete set of desired features, functionalities, and requirements for the ATM Monitoring System. It is organized into Epics, representing major areas of capability. Key epics include:

*   **Core Infrastructure & Platform Setup:** Foundational setup tasks.
*   **ATM Data Ingestion & Processing:** Handling incoming data streams.
*   **ATM Registry Management:** Managing static ATM information.
*   **ATM Status Monitoring:** Tracking operational status.
*   **ATM Hardware Configuration Monitoring:** Tracking peripheral health.
*   **ATM Cash Counter Monitoring:** Tracking cash levels.
*   **Incident Management & Workflow Automation (jBPM):** Managing incident lifecycle.
*   **Alerting & Notification System:** Generating and sending alerts.
*   **Reporting & Analytics:** Providing insights and reports.
*   **API Gateway & Security:** Secure external access point.
*   **User Management & Authentication:** Handling users and security.
*   **User Interface (UI):** Web-based frontend for interaction.
*   **Observability & Monitoring:** Monitoring the platform itself.

*(Refer to the full `product_backlog.md` file for detailed user stories and tasks within each epic.)*

### 5.2. Sprint 1 Backlog Example

To illustrate the iterative development approach, a sample backlog for an initial 3-week sprint was created. 

*   **Sprint Goal:** Establish the core data ingestion pipeline, basic ATM registration, and real-time status tracking capabilities. Provide foundational APIs for accessing this core data.
*   **Key Areas Covered:**
    *   Core Infrastructure Setup (Repositories, CI basics, Local Env).
    *   ATM Data Ingestion Service (Pulsar connection, basic parsing, status event publishing).
    *   ATM Registry Service (Basic CRUD APIs, DB schema).
    *   ATM Status Service (Event consumption, status storage, basic query APIs).
    *   API Gateway (Basic setup and routing for Registry and Status services).
    *   Initial Testing and Documentation.

*(Refer to the `sprint_1_backlog.md` file for the detailed list of tasks and user stories planned for this initial sprint.)*

---



## 6. Implementation Details: Data Ingestion Service

This section provides details on the initial implementation of the `ATM Data Ingestion Service`, which serves as a core component for processing incoming data.

*(Refer to the code provided in `data-ingestion-service.zip` and `pulsar-integration.zip` for the full source.)*

### 6.1. Overview of the Service

The Data Ingestion Service is a Spring Boot application responsible for:

1.  Consuming raw messages from an external Pulsar topic (`atm-raw-data` by default).
2.  Parsing these messages based on a `messageType` field within the JSON payload.
3.  Dispatching the parsed message object to the appropriate processing logic (currently basic logging).
4.  (Future enhancement) Publishing processed events to specific internal Pulsar topics.

### 6.2. Project Structure

The service follows a standard Maven project structure for Spring Boot applications:

*   **`pom.xml`**: Defines project dependencies (Spring Boot, Pulsar Client, Jackson, Jakarta Annotations, Lombok) and build configurations.
*   **`src/main/java/com/atmmonitoring/ingestion`**: Contains the main source code, organized into packages:
    *   `config`: Configuration classes (e.g., `PulsarConfig`).
    *   `exception`: Custom exception classes (e.g., `MessageProcessingException`).
    *   `model`: Data Transfer Objects (DTOs) representing different message types (`BaseAtmMessage`, `StatusMessage`, `ConfigurationMessage`, `CounterMessage`, `TransactionMessage`, `MessageType` enum).
    *   `service`: Service interfaces (`MessageDispatcherService`, `PulsarConsumerService`, `PulsarProducerService`, and specific message type services).
    *   `service/impl`: Concrete implementations of the service interfaces (`MessageDispatcherServiceImpl`, `PulsarConsumerServiceImpl`, `PulsarProducerServiceImpl`, and specific message type service implementations).
*   **`src/main/resources/application.properties`**: Configuration file for application settings, including Pulsar connection details and topic names.
*   **`src/test/java`**: Contains unit tests (e.g., `MessageDispatcherServiceImplTest`, `PulsarProducerServiceImplTest`).

### 6.3. Core Logic: Message Dispatcher

The `MessageDispatcherServiceImpl` is central to the ingestion process:

1.  It receives the raw message payload (String) from the Pulsar consumer.
2.  It uses Jackson `ObjectMapper` to first parse the payload into a `BaseAtmMessage` to identify the `messageType`.
3.  Based on the `messageType` enum value, it re-parses the payload into the specific DTO (`StatusMessage`, `ConfigurationMessage`, etc.).
4.  It then calls the `process()` method of the corresponding injected service implementation (e.g., `StatusMessageService.process(statusMessage)`).
5.  Error handling is included for JSON parsing issues.

### 6.4. Pulsar Integration

Integration with Apache Pulsar is handled by dedicated configuration and service classes:

*   **`PulsarConfig.java`**: Creates the core `PulsarClient` bean required by consumers and producers, configured via `application.properties`.
*   **`PulsarConsumerServiceImpl.java`**: 
    *   Uses `@PostConstruct` to automatically start listening on the configured raw data topic upon application startup.
    *   Uses a `MessageListener` to receive messages asynchronously.
    *   Calls the `MessageDispatcherService` to process received messages.
    *   Handles message acknowledgment (ack/nack) based on processing success or failure.
    *   Includes basic retry logic for processing.
    *   Uses `@PreDestroy` to cleanly close the consumer on application shutdown.
*   **`PulsarProducerServiceImpl.java`**: 
    *   Manages a cache of `Producer` instances for different internal topics.
    *   Provides specific methods (`publishStatusEvent`, `publishConfigurationEvent`, etc.) to send processed events to the correct internal topics (configured via `application.properties`).
    *   Uses asynchronous sending (`sendAsync`) with configurable timeouts.
    *   Configurable batching is enabled by default.
    *   Uses `@PreDestroy` to close all producers on shutdown.

### 6.5. Key Dependencies

*   **Spring Boot Starter Web:** Core Spring Boot web capabilities.
*   **Apache Pulsar Client (`pulsar-client`):** For interacting with Pulsar.
*   **Jackson Databind/Datatype JSR310:** For JSON parsing and handling Java Time objects.
*   **Jakarta Annotation API (`jakarta.annotation-api`):** Provides `@PostConstruct` and `@PreDestroy` annotations (required for Java 11+).
*   **Lombok (Optional):** Reduces boilerplate code (getters, setters, constructors).
*   **Spring Boot Starter Test:** For unit and integration testing.

---



## 7. Diagrams

Visual diagrams are essential for understanding the system structure and flow. The following diagrams have been created using PlantUML syntax:

*   **System Architecture Diagram (`system_architecture.puml`):** Provides a high-level overview of the main components, layers, and their interactions within the platform and with external systems.
*   **Component Diagram (`component_diagram.puml`):** Shows the microservices within the application layer, infrastructure components (Pulsar, Databases), external actors, and their primary relationships.
*   **Sequence Diagram - Hardware Failure (`sequence_diagram_hw_failure.puml`):** Illustrates the step-by-step message flow and interactions between components when a specific event (a hardware failure) occurs, from the ATM to incident creation.

*(These diagrams can be rendered using PlantUML tools or online viewers. For inclusion in a Word document, they would typically be exported as images (e.g., PNG) and embedded.)*

---

## 8. Conclusion

This report provides a comprehensive overview of the ATM Monitoring System project, detailing its vision, architecture, components, workflows, and initial implementation progress. The system is designed using a modern microservices approach, leveraging technologies like Java Spring Boot, Apache Pulsar, and jBPM to create a scalable, resilient, and maintainable platform.

The initial implementation focused on establishing the core data ingestion pipeline using Pulsar and Spring Boot, demonstrating the feasibility of the chosen approach. Key components like the message dispatcher, Pulsar consumer, and producer have been developed.

### 8.1. Summary of Project Status

*   **Architecture Defined:** A clear microservices architecture has been designed.
*   **Core Components Identified:** Key microservices and their responsibilities are outlined.
*   **Workflows Documented:** Major data and process flows are described.
*   **Backlog Created:** A product backlog and sample sprint backlog provide a roadmap for development.
*   **Initial Implementation Done:** The Data Ingestion Service, including Pulsar integration, has been implemented and tested at a basic level.
*   **Diagrams Created:** Visual aids for architecture, components, and sequence flow are available.

### 8.2. Potential Next Steps / Future Enhancements

Building upon the current foundation, future development efforts would likely focus on:

*   Implementing the full business logic within each microservice (Status, Configuration, Counter, Alerting, Incident Management, etc.).
*   Developing the jBPM workflow definitions (BPMN models) and integrating them fully with the Incident Management Service.
*   Building out the API endpoints for all services according to defined contracts (e.g., OpenAPI).
*   Developing the User Interface (UI) for monitoring and interaction.
*   Implementing robust observability (logging, metrics, tracing).
*   Setting up production-grade deployment infrastructure (Kubernetes, CI/CD pipelines).
*   Conducting thorough integration and end-to-end testing.

---

## 9. Appendix

This section lists the key files generated during this project phase, which contain more detailed information:

*   `pasted_content.txt`: Original requirements input.
*   `atm_monitoring_architecture.md`: Initial architecture document.
*   `sprint_1_backlog.md`: Example backlog for the first sprint.
*   `product_backlog.md`: Comprehensive product backlog.
*   `atm_system_workflow.md`: Detailed workflow explanations.
*   `data-ingestion-service.zip`: Zipped codebase for the initial service implementation.
*   `pulsar-integration.zip`: Zipped code specifically for Pulsar consumer/producer classes.
*   `system_architecture.puml`: PlantUML code for the system architecture diagram.
*   `component_diagram.puml`: PlantUML code for the component diagram.
*   `sequence_diagram_hw_failure.puml`: PlantUML code for the hardware failure sequence diagram.
*   `comprehensive_report.md`: This report in Markdown format.

*(Full PlantUML code or backlog details could be included directly here if desired.)*

---
**End of Report**

