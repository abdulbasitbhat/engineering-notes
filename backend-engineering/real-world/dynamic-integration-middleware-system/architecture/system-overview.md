# 🧩 System Overview — Dynamic Integration Middleware

This system is a **dynamic integration middleware**. It was designed as a successor to an older system that was implemented in a highly **client-specific** manner. So every client had a codebase and separate documentation.

In the legacy system, any change required a developer to closely work with client-specific documents and client-specific code. Although components were reused, the system gradually became difficult to maintain. Over time, everything became messy, and even a small change required **code-level modifications**.

The current system is **completely driven by configuration stored in the database**. Any new integration requires only configuration changes and **almost zero code change or hardcoding**.

The main aim was to create a **standard way of integration** while still supporting **custom client-specific logic** where required.

---

## 🎯 Design Goals

- Reduce dependency on client-specific code  
- Move system behavior from code to **configuration**  
- Enable faster onboarding of new integrations  
- Improve maintainability and scalability  
- Support customization without breaking standardization
- Support common audit logic and error handling mechanism

---

## 🧱 System Design

The system is divided into **thin JARs**, each designed with a **plug-and-play nature**.

- Each JAR has standardized input and output
- JARs are **independent of each other**
- Any single JAR can be upgraded without impacting others

This design allows the system to evolve over time without large-scale refactoring. The whole system was designed on orchestration of these thin jars for business logic.

---

## ⏱️ Workflow & Scheduling

The main workflow of the system is based on **CRON job scheduling**.

### 📦 Queues (Pipelines)

- The system runs using pipelines called **queues** define in **Task Queues Table**
- Each queue contains multiple **tasks**
- Tasks execute **sequentially**, forming a controlled workflow

Queues define *when* things run, while tasks define *what* runs. These queues are run by a main runner, that utilizes command line argument as input to define the queue to run. The command is then scheduled in a CRON manner using crontab. Where frequency and syncronization between different queues/jobs is maintained.

---

## ⚙️ Task Parameter Table

Each task has an entry in a **Task Parameter Table**.

This table defines:
- Which adapter should be used
- How the task should execute
- Input and output behavior
- Source and target systems
- Which external system to connect and which protocol to use.
- Which connection credentials to use
- Any API filters needed
- Is the flow from external system to database, database to database, or database to external

You can think of this table as a **feeder or controller**, similar to a `main()` function, which supplies configuration and decides how a task runs at runtime.

---

## 🔄 Data Mapping

Since tasks operate on data, each task can have **multiple entries** in a **Task Mapping Table**.

### 🗂️ Mapping Table Responsibilities

- Defines how **external data** maps to **internal database structures**
- Supports mapping of nested external data
- Specifies which internal table and column data should be stored in
- Target tables are defined by the task parameter entry
- Has transformation logic in a JSON format for each column

Each row in the mapping table represents a **single column-level mapping**.

This mapping mechanism supports:
- External system ➝ Internal database  
- Internal database ➝ Internal database (inbound/outbound tables)  
- Internal database ➝ External system  

---

## 🔧 Transformation Logic

Each mapping row contains a **Transformation column**, where transformation logic is defined in **JSON format**.

### 🔁 How Transformations Work

- The JSON specifies which transformation function to apply
- A dedicated **Transformers JAR** reads this JSON
- Based on the configuration, the correct transformation function is invoked
- The transformed data is returned and passed to the next stage

---

## 🔌 Adapters

All tasks are implemented as **adapters**.

Adapters do not contain client-specific logic. Instead, they:
- Orchestrate thin JARs
- Route data based on configuration
- Act as a bridge between configuration and execution

---

### 📄 Example: Parser Adapter

A `ParserAdapter` contains logic to route incoming data to the appropriate parser:
- JSON
- XML
- CSV
- Fixed Width

Which parser to use, how data is ingested, and which database table data should be written to are all defined in the **Task Parameter Table**.

A task is effectively an **instance of an adapter**, configured at runtime. Everything that needs to be fed using a main function is fed by database configs from parameters table

---

## 🧬 Column-Level Control

- Column mappings are defined in the **Task Mapping Table**
- Each column can have its own transformation logic
- Mapping is driven entirely by task configuration
- The same adapter can be reused across multiple tasks and queues

This provides fine-grained control over data handling while keeping the execution engine generic.

---

## 🔁 Configuration-Driven Execution

Multiple configurations can use the **same adapter**, even within the same workflow sequence.

All behavior is defined through database configuration, allowing:
- Reuse of adapters
- Easy reordering of tasks
- Changes without redeploying code

---

## ▶️ Runtime Execution & Client Segregation

The system is executed through a **main runner**, which exposes a `main` function that accepts **two command-line arguments**:

1. **Client Name**
2. **Queue Name**

These arguments determine which client the system runs for and which pipeline (queue) is executed. This command is later configured as a **CRON job** using `crontab`, enabling scheduled and automated execution. A connection and logging session is create at main and reused throughout the workflow. Its send deep down to each and every needful function. The connection is closed after execution in main itself.

---

## 🧭 Client Context Resolution

The **Client Name** defines the execution context for the run. It determines:

- Which database schema should be used
- Where audit files are written
- Where logging files are stored on the server

This ensures that each client’s data and execution remain isolated from others.

---

## 📁 Client Folder Structure

Each client is assigned a dedicated folder on the server to ensure segregation of data, logs, and client-specific logic.

    /<client-name>/
    ├── config        # Logging and client-specific configurations
    ├── logs          # Runtime logs
    ├── lib           # Client-specific thin JARs
    ├── archive       # Archived files
    ├── inbound       # Incoming data files
    └── outbound      # Outgoing data files

### Folder Details

- **config**  
  Contains client-specific configuration files, primarily for logging and runtime behavior.

- **logs**  
  Stores runtime logs generated during execution for the client.

- **lib**  
  Holds client-specific thin JARs such as custom adapters or extensions.

- **archive**  
  Stores processed or historical files for auditing and traceability.

- **inbound**  
  Entry point for incoming data from external systems.

- **outbound**  
  Stores files generated for external systems.



This folder structure keeps client data, logs, and custom logic clearly segregated.

---

## 🔁 Adapter Execution Contract

All adapters follow a **standard execution contract**.

Each adapter exposes a **`processTask`** function, which acts as the **entry signature** for task execution. This function is triggered by the **main runner** during runtime.

The `processTask` function is responsible for:
- Receiving task-level configuration
- Setting up execution context
- Invoking the internal processing flow

Internally, `processTask` calls the **`process`** function.

The `process` function contains:
- The actual task logic
- Orchestration of thin JARs
- Routing of data through parsers, transformers, and other components

This separation ensures that:
- All adapters have a **consistent entry point**
- The main runner interacts with adapters in a uniform way
- Business logic remains encapsulated inside the adapter implementation

By standardizing `processTask` as the entry signature, the system maintains a clean and predictable execution flow while allowing flexibility inside each adapter.


## 🔗 Shared Common JARs (EFS)

All **common JARs** are deployed on an **EFS (Elastic File System)** shared across all clients.

- Common adapters and utilities are placed here
- These JARs are maintained centrally
- All clients reuse these components without duplication

This allows consistent behavior across clients while minimizing maintenance effort.

---

## 🔌 Client-Specific Logic

When client-specific behavior is required, a **client-specific adapter** can be created.

- These adapters may contain custom logic
- They can orchestrate existing common JARs
- They may also introduce new client-only JARs if needed

Client-specific adapters are placed within the respective client’s folder to maintain isolation.

---

## ⚙️ Adapter Resolution via Configuration

For tasks that require client-specific logic, the **Task Parameter Table** allows overriding the adapter selection.

Instead of using a common adapter, the task configuration sets:

- `adapter_name_full_path` → pointing to the client-specific adapter rather than common adapter jar at EFS

This enables client-specific customization without impacting other clients or modifying shared code.

## 🗃️ Supporting Configuration & Metadata Tables

In addition to task parameters and mappings, the system uses a few supporting tables to handle **execution tracking, connectivity, and client-level configuration**.

---

### 📊 Task History Table

The **Task History Table** is used to track execution details for every task run.

It stores:
- Task execution **status** (success / failure)
- **Run start time**
- **Run end time**
- **Error message**, if any occurred during execution

This table helps in monitoring, debugging, and auditing task executions over time. Each task run creates a corresponding history entry, making execution traceable and transparent.

---

### 🔐 Connection Table

The **Connection Table** is used to store connection-related details in a generic and extensible way.

- Each entry has a **connection name**
- The value is stored as a **JSON object**

The JSON structure varies based on connection type:
- API connections may contain `url`, `clientSecret`, tokens, tokenExpiry and regeneration logic etc.
- SFTP connections may contain host, port, username, and authentication details
- Other connection types can store their respective configurations

The **connection name** acts as a key and is referenced inside the **Task Parameter Table**, allowing tasks to dynamically resolve connection details at runtime without hardcoding them.

---

### ⚙️ Client Configuration Table

The **Configuration Table** stores client-specific configuration values that are used across the system.

This includes:
- Client-specific **timezone** details
- Constants required during execution
- Any reusable configuration values needed by tasks or adapters

This table helps centralize client-level settings and avoids duplication across multiple task configurations.

---

### 🔁 Configuration-Driven Resolution

Together, these tables ensure that:
- Task execution is traceable
- Connections are reusable and configurable
- Client-specific behavior is centralized
- No sensitive or environment-specific details are hardcoded

All runtime behavior continues to remain **fully configuration-driven**.

## 📌 Summary

This system converts integration from a **code-heavy, client-specific problem** into a **configuration-driven platform** that is:

- 🧩 Modular  
- 🔄 Flexible  
- 🛠 Maintainable  
- 🚀 Scalable  

While keeping the core execution logic stable and reusable.
