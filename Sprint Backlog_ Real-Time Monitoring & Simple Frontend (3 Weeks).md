# Sprint Backlog: Real-Time Monitoring & Simple Frontend (3 Weeks)

**Sprint Goal:** Establish a basic end-to-end flow for receiving ATM data (status, config, counters), processing it, storing the real-time state, and displaying the status of ATMs on a simple web interface. Provide foundational APIs for accessing this real-time data.

**Duration:** 3 Weeks

---

## Epic: Core Backend Services Setup

**User Story 1: As a System Operator, I want the system to receive and process basic ATM status updates so that I know the operational state of each ATM.**
*   **Task:** Implement Pulsar consumer logic in Data Ingestion Service for `StatusMessage` type.
*   **Task:** Implement basic validation and parsing for `StatusMessage`.
*   **Task:** Implement `StatusMessageService` logic to process parsed status data.
*   **Task:** Define internal `AtmStatusUpdated` event structure.
*   **Task:** Implement Pulsar producer logic in Data Ingestion Service to publish `AtmStatusUpdated` events.
*   **Task:** Set up `ATM Status Service` project structure (Spring Boot).
*   **Task:** Implement Pulsar consumer in `ATM Status Service` to receive `AtmStatusUpdated` events.
*   **Task:** Define database schema for storing current ATM status (ATM ID, state, last seen, etc.) in PostgreSQL.
*   **Task:** Implement logic in `ATM Status Service` to update the database upon receiving events.
*   **Task:** Create a basic REST API endpoint in `ATM Status Service` (`/api/atms/{atmId}/status`) to retrieve the current status of a single ATM.
*   **Task:** Create a basic REST API endpoint in `ATM Status Service` (`/api/atms/status`) to retrieve the status of all ATMs.
*   **Task:** Write unit tests for status processing logic.
*   **Task:** Write basic integration tests for the status update flow (Ingestion -> Status Service -> API).

**User Story 2: As a System Operator, I want the system to receive and process basic ATM hardware configuration updates so that I have a general idea of component health.**
*   **Task:** Implement Pulsar consumer logic in Data Ingestion Service for `ConfigurationMessage` type.
*   **Task:** Implement basic validation and parsing for `ConfigurationMessage`.
*   **Task:** Implement `ConfigurationMessageService` logic to process parsed configuration data.
*   **Task:** Define internal `AtmConfigurationChanged` event structure.
*   **Task:** Implement Pulsar producer logic in Data Ingestion Service to publish `AtmConfigurationChanged` events.
*   **Task:** Set up `ATM Configuration Service` project structure (Spring Boot).
*   **Task:** Implement Pulsar consumer in `ATM Configuration Service` to receive `AtmConfigurationChanged` events.
*   **Task:** Define database schema for storing current hardware status summary (e.g., ATM ID, overall health indicator - Green/Orange/Red) in PostgreSQL.
*   **Task:** Implement logic in `ATM Configuration Service` to update the database (simple health status for now).
*   **Task:** Create a basic REST API endpoint in `ATM Configuration Service` (`/api/atms/{atmId}/config/summary`) to retrieve the current hardware health summary.
*   **Task:** Write unit tests for configuration processing logic.

**User Story 3: As a System Operator, I want the system to receive and process basic ATM cash counter updates so that I have a general idea of cash levels.**
*   **Task:** Implement Pulsar consumer logic in Data Ingestion Service for `CounterMessage` type.
*   **Task:** Implement basic validation and parsing for `CounterMessage`.
*   **Task:** Implement `CounterMessageService` logic to process parsed counter data.
*   **Task:** Define internal `AtmCountersUpdated` event structure.
*   **Task:** Implement Pulsar producer logic in Data Ingestion Service to publish `AtmCountersUpdated` events.
*   **Task:** Set up `ATM Counter Service` project structure (Spring Boot).
*   **Task:** Implement Pulsar consumer in `ATM Counter Service` to receive `AtmCountersUpdated` events.
*   **Task:** Define database schema for storing current cash summary (e.g., ATM ID, total cash, low cash flag) in PostgreSQL.
*   **Task:** Implement logic in `ATM Counter Service` to update the database (simple summary for now).
*   **Task:** Create a basic REST API endpoint in `ATM Counter Service` (`/api/atms/{atmId}/counters/summary`) to retrieve the current cash summary.
*   **Task:** Write unit tests for counter processing logic.

**User Story 4: As a Developer, I need an API Gateway configured so that frontend and external systems have a single entry point.**
*   **Task:** Set up API Gateway project (Spring Cloud Gateway).
*   **Task:** Configure basic routing rules for `ATM Status Service`, `ATM Configuration Service`, and `ATM Counter Service` API endpoints.
*   **Task:** Implement basic request logging in the Gateway.

---

## Epic: Simple Frontend Interface

**User Story 5: As a System Operator, I want a simple web page that lists all monitored ATMs and their current operational status so that I can quickly see which ATMs are online or offline.**
*   **Task:** Set up a basic frontend project (e.g., React using Create React App, or simple server-rendered framework like Thymeleaf).
*   **Task:** Create a main page/view for the ATM dashboard.
*   **Task:** Implement frontend logic to periodically fetch data from the `ATM Status Service` API (`/api/atms/status`) via the API Gateway.
*   **Task:** Display the list of ATMs with their IDs and current operational status (e.g., "In Service", "Offline", "Out of Service") based on the fetched data.
*   **Task:** Implement visual indicators for status (e.g., green dot for online, red dot for offline).
*   **Task:** Ensure the list updates automatically at a regular interval (e.g., every 30 seconds).

**User Story 6: As a System Operator, I want the ATM list to also show a basic hardware health indicator so I can quickly identify ATMs with potential component issues.**
*   **Task:** Modify the frontend data fetching logic to also call the `ATM Configuration Service` API (`/api/atms/{atmId}/config/summary`) for each ATM (or enhance the backend API to provide combined data).
*   **Task:** Display a simple color-coded indicator (Green/Orange/Red) next to each ATM based on the hardware health summary.

**User Story 7: As a System Operator, I want the ATM list to show a low cash warning indicator so I can prioritize replenishment.**
*   **Task:** Modify the frontend data fetching logic to also call the `ATM Counter Service` API (`/api/atms/{atmId}/counters/summary`) for each ATM (or enhance backend APIs).
*   **Task:** Display a warning icon (e.g., a fuel gauge symbol) next to ATMs flagged as having low cash.

---

## Epic: Development Environment & Setup

**User Story 8: As a Developer, I need a local development environment using Docker Compose so that I can easily run the necessary infrastructure (Pulsar, PostgreSQL) and services.**
*   **Task:** Create a `docker-compose.yml` file.
*   **Task:** Add service definitions for Apache Pulsar (standalone mode is sufficient).
*   **Task:** Add service definition for PostgreSQL.
*   **Task:** Add service definitions for Data Ingestion, Status, Configuration, Counter services, and API Gateway (building from source or using pre-built images).
*   **Task:** Configure necessary environment variables and network settings within the compose file.
*   **Task:** Document how to start and stop the local environment.

**User Story 9: As a Team, we need basic project structure and build configurations for the new services so that we can start development efficiently.**
*   **Task:** Initialize Git repositories for the new services (Status, Configuration, Counter, Frontend, API Gateway).
*   **Task:** Create standard Maven/Gradle build files (`pom.xml` / `build.gradle`) for backend services.
*   **Task:** Create standard build/dependency management files (`package.json`) for the frontend service.
*   **Task:** Define basic README files for each service.

---

## Potential Stretch Goals (If time permits)

*   **Stretch:** Implement a very basic `Alerting Service` that consumes failure/low cash events and logs them to the console.
*   **Stretch:** Add a simple section to the frontend UI to display the last 5 critical alerts logged by the basic Alerting Service.
*   **Stretch:** Implement basic authentication/authorization placeholder on the API Gateway.
*   **Stretch:** Set up a basic CI pipeline (e.g., GitHub Actions) to build and test the backend services on commit.

---

