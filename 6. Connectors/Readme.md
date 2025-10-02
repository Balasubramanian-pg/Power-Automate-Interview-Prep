# The Ultimate Masterclass on Power Automate Connectors

## Part 1: The Conceptual Foundation - The Integration Fabric

### Introduction: The API Revolution Made Simple

*   At its core, the modern digital enterprise is a collection of specialized applications and services. The CRM system is separate from the HR system, which is separate from the document storage system and the email system. The greatest challenge—and opportunity—in business technology is making these disparate systems talk to each other. This is traditionally done via **APIs (Application Programming Interfaces)**.
*   A **Connector** in Power Automate is a user-friendly, low-code wrapper around a service's API. It acts as a pre-built adapter, translating complex API operations (like HTTP requests, authentication handshakes, and JSON parsing) into simple, understandable "actions" and "triggers" that a citizen developer can use.
*   Connectors are the integration fabric of the Power Platform. They are what allow you to create a flow that is triggered by an email in **Outlook**, gets an approval in **Teams**, updates a record in **Dataverse**, saves a file to **SharePoint**, and logs the transaction in a **SQL Server** database. Without connectors, Power Automate would be a closed box, unable to interact with the outside world.

### The Anatomy of a Connection

*   When you use a connector for the first time, Power Automate prompts you to create a **Connection**. It is crucial to understand what this represents.
*   A Connection is the stored set of **credentials** that Power Automate will use to authenticate with the target service *on your behalf*.
*   **Authentication Types:**
    *   **OAuth 2.0 (Most Common):** When you connect to SharePoint, Outlook, or Teams, you are presented with a Microsoft sign-in dialog. This is an OAuth flow. You are granting the Power Automate service principal permission to perform actions using your identity. The connection stores a secure token, not your password.
    *   **API Key:** For many third-party services, you will go to that service's developer portal, generate a secret key, and paste that key into the connection settings in Power Automate.
    *   **SQL Server Authentication:** You provide a username and password for a SQL login.
*   **Connection References (The Modern Best Practice):**
    *   In the past, a flow was directly tied to a specific user's connection. When moving a flow to a new environment (from development to production), you had to manually edit the flow to point it to the production connections. This was error-prone.
    *   **Connection References** are a modern feature used within **Solutions**. A connection reference is a pointer. Your flow points to the reference, and the reference points to the actual connection. When you deploy your solution to a new environment, you simply update the connection reference to point to the correct production connection for that environment. This decouples the flow logic from the specific credentials and is essential for healthy Application Lifecycle Management (ALM).

> [!IMPORTANT]
> The permissions of the **connection** determine what the flow can do. If you create a SharePoint flow using *your* connection, that flow runs with *your* SharePoint permissions. It can create and delete files in any library that *you* have access to. If your account is a Site Collection Admin, your flow is also a Site Collection Admin. This has significant security implications that must be managed carefully.

### Standard vs. Premium: The Licensing Gateway

*   The single most important factor that governs which connectors you can use is **licensing**. Microsoft categorizes every connector into one of two tiers.
*   **Standard Connectors:**
    *   **Included:** With most Microsoft 365 and Office 365 business and enterprise plans.
    *   **Scope:** Primarily focused on services within the Microsoft 365 ecosystem.
    *   **Examples:** SharePoint, Outlook, Teams, OneDrive, Forms, Planner, Notifications.
    *   **Target Audience:** Enables broad "citizen developer" automation of everyday productivity tasks.
*   **Premium Connectors:**
    *   **Requires:** A premium Power Automate license (e.g., "Power Automate Per User" plan, "Per Flow" plan, or seeded rights from a Dynamics 365 or Power Apps premium license).
    *   **Scope:** Connects to enterprise systems, external services, and provides advanced capabilities.
    *   **Examples:** Dataverse, SQL Server, Azure DevOps, Salesforce, Adobe Sign, HTTP, AI Builder, and all **Custom Connectors**.
    *   **Target Audience:** For building mission-critical, enterprise-grade automations that connect to core business systems.

> [!WARNING]
> Before you begin designing any solution, you MUST identify if your required connectors are Standard or Premium. A brilliant solution design is useless if the organization does not have the budget for the necessary premium licenses. This should be one of the first questions you answer in the discovery phase of any project.

---

## Part 2: The Core Microsoft 365 Connectors (Standard)

*   These are the connectors that form the foundation of most automations for information workers. They are readily available and incredibly powerful for streamlining collaboration and productivity.

### Connector 1: SharePoint

*   **Core Use Case:** Automating processes around lists (structured data) and document libraries (files).
*   **Strengths:** Included with M365, excellent for document-centric workflows, highly versatile.
*   **Triggers (Most Common):**
    *   `When an item is created`: Starts a flow when a new item is added to a list.
    *   `When a file is created in a folder`: Starts a flow when a new file is uploaded to a library.
    *   `When an item is created or modified`: Fires for both new and updated items (caution: infinite loops).
    *   `For a selected item`: A manual trigger that appears in the SharePoint UI.
*   **Actions (Most Common):**
    *   `Get items`: Retrieves a set of items from a list. The **Filter Query** is the most important field for performance.
    *   `Create item`: Adds a new item to a list.
    *   `Update item`: Modifies an existing item. Requires the item's `ID`.
    *   `Get file content`: Reads the binary content of a file, necessary before sending it as an email attachment or saving it elsewhere.
    *   `Create file`: Saves content (e.g., from an attachment) as a new file in a document library.
*   **Realistic Scenario: Document Review Process**
    1.  **Trigger:** `When a file is created in a folder` (Folder: "Drafts").
    2.  **Action:** `Start and wait for an approval` (sends approval to the file creator's manager).
    3.  **Action:** `Condition` on the approval `Outcome`.
    4.  **If Approved:**
        *   `Get file content` using the `Identifier` from the trigger.
        *   `Create file` in the "Published" folder using the content.
        *   `Delete file` from the "Drafts" folder.

### Connector 2: Office 365 Outlook

*   **Core Use Case:** Automating any process that involves email, from notifications to data extraction.
*   **Strengths:** Ubiquitous in the corporate world, powerful server-side filtering, fast webhook triggers.
*   **Triggers (Most Common):**
    *   `When a new email arrives (V3)`: The webhook-based trigger that fires instantly. Can be configured to watch specific folders, watch for attachments, or filter by subject.
    *   `When an event is added, updated or deleted (V2)`: For calendar automation.
*   **Actions (Most Common):**
    *   `Send an email (V2)`: The fundamental notification action. Can send from your mailbox or a shared mailbox.
    *   `Send an email with options`: Sends an email with actionable buttons (e.g., "Approve," "Reject"). The flow waits for the user to click a button. A simpler alternative to the full Approvals connector.
    *   `Move email (V2)`: For triaging your inbox. After processing an email, move it to an "Archive" or "Processed" folder.
    *   `Create event (V4)`: Adds an appointment to a user's calendar.
*   **Realistic Scenario: Processing Invoice Attachments**
    1.  **Outlook Rule:** Create a server-side rule to move all emails with "Invoice" in the subject to a folder named "Flow - Invoices".
    2.  **Trigger:** `When a new email arrives (V3)` configured to monitor the "Flow - Invoices" folder.
    3.  **Action (`Apply to each`):** A loop to iterate through the `Attachments` from the trigger.
    4.  **Action (Inside loop):** `Create file` (SharePoint) to save the attachment `Content` to a library.

### Connector 3: Microsoft Teams

*   **Core Use Case:** Integrating automations directly into the collaborative workspace of Teams.
*   **Strengths:** Meets users where they work, powerful notification options (adaptive cards), team and channel management.
*   **Triggers (Most Common):**
    *   `For a selected message`: Allows a user to manually trigger a flow from a specific Teams message.
    *   `When a new channel message is added`: Fires when someone posts in a monitored channel.
*   **Actions (Most Common):**
    *   `Post message in a chat or channel`: The main notification action.
    *   `Post adaptive card and wait for a response`: The killer feature. Allows you to post a rich, interactive card with fields and buttons directly into a Teams chat. The flow then pauses and waits for a user to fill out the card and click a submit button. This is excellent for "in-context" approvals and data gathering.
    *   `Create a team`, `Add a member to a team`: For automating the provisioning of new teams and channels.
*   **Realistic Scenario: Automated Team Provisioning**
    1.  **Trigger:** `When a new item is created` in a SharePoint list called "New Project Requests".
    2.  **Action:** `Create a team`. Uses the project name from the SharePoint item as the new team name.
    3.  **Action:** `Add a member to a team`. Adds the person who submitted the SharePoint request to the new team as an owner.
    4.  **Action:** `Post message in a chat or channel`. Posts a welcome message to the "General" channel of the newly created team.

### Flashcard Q&A: Standard M365 Connectors

*   **Q:** What is the primary connector for automating processes around business files and metadata?
*   **A:** **SharePoint**.

*   **Q:** What is the most powerful and interactive action available in the Teams connector?
*   **A:** `Post adaptive card and wait for a response`, which allows you to embed forms and actions directly into a Teams chat.

*   **Q:** You want to build a flow that is triggered instantly when an email arrives. Which Outlook trigger should you use?
*   **A:** `When a new email arrives (V3)`, as it is webhook-based and provides near-instantaneous triggering.

---

## Part 3: The Premium Connectors - Enterprise-Grade Integration

*   These connectors require premium licensing but unlock the ability to connect to core business systems, relational databases, and custom services.

### Connector 4: Dataverse

*   **Core Use Case:** Building robust, scalable, and secure automations on top of the premier Power Platform data backend.
*   **Strengths:** Unmatched security, true relational data, excellent performance, deep integration with model-driven apps and Dynamics 365.
*   **Triggers (Most Common):**
    *   `When a row is added, modified or deleted`: The all-in-one trigger. Highly configurable.
    *   **Filtering Attributes:** You can configure the trigger to only fire if a specific field (attribute) is changed. This is the built-in, no-code way to prevent infinite loops, and is far superior to SharePoint's trigger conditions.
    *   **Run As:** Can be configured to run in the context of the user who made the change, the flow's owner, or a system process.
*   **Actions (Most Common):**
    *   `Add a new row`, `Update a row`, `Delete a row`: Standard CRUD operations.
    *   `List rows`: The equivalent of SharePoint's `Get items`. It has a very powerful and extensive OData filter syntax that supports filtering on related table values.
    *   `Relate rows` / `Unrelate rows`: Actions specifically for managing Many-to-Many relationships.
    *   `Perform an unbound action` / `Perform a bound action`: Allows you to call custom server-side business logic (Custom APIs) built directly into Dataverse.
*   **Realistic Scenario: Advanced Sales Process Automation**
    1.  **Trigger:** `When a row is modified` on the "Opportunities" table.
    2.  **Filter Attribute:** `statuscode` (Only fire when the Status field changes).
    3.  **Action (`Switch`):** A switch on the new `statuscode`.
    4.  **Case "Won":**
        *   `Update a row`: Update the parent "Account" record's `Last Purchase Date` field.
        *   `Relate rows`: Associate the Opportunity record with the "Active Customers" Marketing List record.
    5.  **Case "Lost":**
        *   `Add a new row`: Create a new "Follow-up Activity" record assigned to the account manager, scheduled for 90 days in the future.

### Connector 5: SQL Server

*   **Core Use Case:** Integrating Power Automate with new or existing SQL databases, whether on-premises or in Azure.
*   **Strengths:** High performance, transactional integrity, ability to leverage existing SQL logic like stored procedures and views.
*   **Triggers:**
    *   `When an item is created / modified (V2)`: Polling-based triggers that watch a specific table for changes. Requires the table to have a `TIMESTAMP` column for reliable change detection.
*   **Actions (Most Common):**
    *   `Get rows (V2)`: Retrieves multiple rows. Supports OData `Filter Query` and `Order By` clauses.
    *   `Insert row (V2)`, `Update row (V2)`, `Delete row (V2)`: Standard CRUD operations.
    *   `Execute a SQL query (V2)`: The power user's action. Allows you to write and execute any raw SQL statement (including `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and more complex joins). This is for read operations.
    *   `Execute a stored procedure (V2)`: Call a pre-existing stored procedure in your database, passing in parameters. This is the most secure and maintainable way to perform complex write operations.
*   **Realistic Scenario: Logging Audit Data**
    1.  **Trigger:** Dataverse - `When a row is modified` on a sensitive "Financials" table.
    2.  **Action:** Office 365 Users - `Get my profile (V2)` to get the name of the user who made the change from the trigger's output.
    3.  **Action:** SQL Server - `Execute a stored procedure (V2)`.
    4.  **Parameters Passed to Stored Proc:** The `FinancialRecordID`, the user's name, the new and old values, and `utcNow()` for a timestamp. The stored procedure handles the logic of writing a new row to a separate `Financials_AuditLog` table.

### Connector 6: HTTP

*   **Core Use Case:** The universal adapter. Allows your flow to communicate with any service in the world that exposes a REST API. This is a premium connector.
*   **Strengths:** Infinitely extensible. If a service has an API, this connector can talk to it.
*   **Actions (There is only one primary action):**
    *   `HTTP`:
        *   **`Method`:** `GET` (read data), `POST` (create data), `PUT`/`PATCH` (update data), `DELETE`.
        *   **`URI`:** The URL of the API endpoint.
        *   **`Headers`:** Where you put information like `Content-Type` and `Authorization` keys.
        *   **`Body`:** The data payload for POST/PUT requests, typically written using a `json()` expression.
        *   **`Authentication`:** Supports various schemes for connecting to secure APIs.
*   **Scenario:** This is covered in depth in the "Custom Connectors" section, as the HTTP action is the engine that powers custom connectors under the hood.

### Connector 7: Custom Connectors

*   **Core Use Case:** To take a raw REST API and make it a first-class, reusable, and user-friendly connector for your entire organization.
*   **Why use it over the raw HTTP action?**
    1.  **Reusability:** Instead of every developer re-creating the same complex HTTP request, they can just add your pre-built custom connector and use a simple action like `'MyCompanyAPI'.SubmitTimesheet()`.
    2.  **Ease of Use:** A well-defined custom connector provides strongly typed dynamic content. Instead of using `parseJSON` and complex expressions to get data out of the response body, you get clean dynamic tokens like `'Employee Name'` and `'Project ID'`.
    3.  **Governance and Security:** You can share the custom connector with specific users and groups, and you manage the authentication in one central place.
*   **How to Build:**
    1.  Start from blank, or import an **OpenAPI (Swagger)** file which is the industry standard for defining REST APIs.
    2.  Use the UI to define the general information (icon, color, description).
    3.  Define the authentication method (e.g., API Key).
    4.  Define each **Action** (e.g., "GetCustomerByID").
        *   Provide a request URL (e.g., `https://api.mycompany.com/customers/{id}`). The `{id}` part defines an input parameter.
        *   Provide a sample response body. Power Automate analyzes this and automatically generates the schema for the output dynamic content.
    5.  Save and test the connector.

### Flashcard Q&A: Premium Connectors

*   **Q:** You need to trigger a flow *only when* the "Status" field of a Dataverse record is changed, ignoring all other field modifications. How do you achieve this efficiently?
*   **A:** In the Dataverse trigger `When a row is added, modified or deleted`, you add the logical name of the "Status" field to the **`Filter attributes`** (also known as Filtering Columns) setting.

*   **Q:** What is the most secure and maintainable method for performing complex, multi-statement data updates in a SQL Server database from a flow?
*   **A:** Create a **stored procedure** in SQL Server that encapsulates the logic and call that stored procedure from the flow using the `Execute a stored procedure (V2)` action.

*   **Q:** What is the primary benefit of creating a **Custom Connector** instead of just using the generic `HTTP` action in every flow?
*   **A:** **Reusability and Ease of Use**. A custom connector makes the API available to all makers as a simple set of pre-defined actions with user-friendly dynamic content, removing the need for everyone to understand HTTP and JSON.

---

## Part 4: Special and Utility Connectors

### Connector 8: Power Apps

*   **Core Use Case:** To bridge the gap between a Power App canvas app and a Power Automate flow.
*   **Primary Trigger:** `Power Apps (V2)`. This trigger exposes the flow to be "callable" from a Power App.
*   **Key Action in Flow:** `Respond to a PowerApp or flow`. This action sends data *back* to the canvas app that called it.
*   **Integration Pattern:**
    1.  App needs to perform a task it can't do well (e.g., generate a PDF, complex loop).
    2.  User clicks a button in the app, which calls `MyFlow.Run(parameters)`.
    3.  The flow runs, performs the task.
    4.  The `Respond to...` action sends a result back (e.g., a link to the generated file, or a status of "Success").
    5.  The app captures this result in a variable (`Set(gblFlowResult, MyFlow.Run(...))`) and uses it to update the UI.

### Connector 9: AI Builder

*   **Core Use Case:** To infuse your automations with artificial intelligence without being a data scientist. This is a premium connector.
*   **Strengths:** Provides access to a suite of pre-built and custom-trainable AI models.
*   **Common Pre-built Model Actions:**
    *   **`Extract information from forms`:** Scans a document (like a PDF invoice) and automatically extracts key-value pairs (Invoice Number, Total Amount, Vendor Name). This is a cornerstone of intelligent automation.
    *   **`Recognize text in an image or a PDF document (OCR)`:** Performs Optical Character Recognition to extract all text from an image.
    *   **`Predict`:** This is the action for using a custom AI Builder model that you have trained on your own data (e.g., a prediction model to forecast sales, a classification model to categorize support tickets).
    *   **`Analyze sentiment`:** Takes a block of text and determines if its sentiment is Positive, Negative, or Neutral.
*   **Realistic Scenario: Intelligent Invoice Processing**
    1.  **Trigger:** `When a new email arrives` with "Invoice" in the subject.
    2.  **Action (`Apply to each`):** Loop through attachments.
    3.  **Action (AI Builder):** `Extract information from forms`, using the attachment `Content`.
    4.  **Action (`Create item`):** Create a new item in a SharePoint list, using the dynamic content outputs from the AI Builder action (`Invoice number value`, `Total value`) to populate the columns.

### Connector 10: Approvals

*   **Core Use Case:** To create and manage formal, auditable human approval workflows.
*   **Key Action:** `Start and wait for an approval`.
    *   This is a special "long-running" action. The flow will pause its execution—potentially for days—until all required users have responded.
*   **Approval Types:**
    *   **First to respond:** The first person in the list of approvers to respond determines the outcome for everyone.
    *   **Everyone must approve:** The approval is only considered "Approve" if every single person in the list approves. If anyone rejects, the whole process is rejected.
*   **Integration with Teams/Outlook:** The approval requests show up as actionable messages directly in Teams and Outlook, allowing users to approve or reject with comments without ever leaving their primary applications.
*   **Outputs:** The action produces rich outputs, including the overall `Outcome` and an array of `Responses` containing the individual comment and response from each approver.

### Flashcard Q&A: Special Connectors

*   **Q:** What is the name of the flow action that sends data back to a calling Power App?
*   **A:** `Respond to a PowerApp or flow`.

*   **Q:** You need a flow to automatically read the invoice number and total amount from thousands of PDF invoices your company receives. Which connector is designed specifically for this task?
*   **A:** The **AI Builder** connector, using the `Extract information from forms` action.

*   **Q:** A flow action needs to pause and wait for a response from one or more users before it can continue. What connector provides this capability?
*   **A:** The **Approvals** connector.
