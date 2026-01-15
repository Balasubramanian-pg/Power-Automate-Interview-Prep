# The Ultimate Masterclass on Power Automate Actions

## Part 1: The Foundational Concepts of Actions

### Introduction: The Engine of Automation

*   If the trigger is the "when," then actions are the "what." Actions are the individual, sequential steps that make up the business logic of your Power Automate flow. They are the engine of automation, the building blocks that you assemble to perform a process.
*   Every task, from sending a simple email to performing a complex data transformation and calling an external API, is accomplished through one or more actions. A flow can be as simple as a single action or contain hundreds of actions organized into complex branches of conditional logic and loops.
*   Mastering the common actions and, more importantly, the *patterns* of how they work together, is the key to unlocking the full potential of Power Automate. This guide will move beyond a simple list of actions and delve into the strategies and best practices that define professional-grade workflow development.

### The Anatomy of an Action: Inputs, Outputs, and Settings

*   Every action, regardless of which connector it belongs to, has a similar underlying structure.

*   **1. Inputs:**
    *   These are the parameters or data that the action requires to do its job.
    *   Inputs can be **static** (hardcoded values, like typing "invoices@company.com" into the "To" field of an email) or **dynamic**.
    *   **Dynamic Inputs:** This is where the power lies. The inputs for an action are most often populated with the outputs from the trigger or a preceding action, using either the **Dynamic Content** picker or an **Expression**.
    *   **Example (`Create item` in SharePoint):** The inputs would be the `Site Address`, the `List Name`, and a field for each column in that list (e.g., `Title`, `DueDate`). You would typically populate the `Title` input with the `Subject` dynamic content from an email trigger.

*   **2. Outputs:**
    *   After an action successfully executes, it produces a set of outputs. These outputs contain the result of the action's operation.
    *   The outputs of one action immediately become available as dynamic content for all subsequent actions.
    *   **Example (`Create item` in SharePoint):** The outputs would include the `ID` of the newly created item, a `Link to item`, and all the values that were just saved to its columns. This `ID` is crucial, as you might need it in a later `Update item` action.

*   **3. Settings (The "..." Menu):**
    *   This often-overlooked menu contains advanced configuration options that are critical for building robust flows.
    *   **Connection References:** Allows you to change the user connection an action is using.
    *   **Retry Policy:** By default, if an action fails due to a temporary issue (like a brief network outage), Power Automate will try to run it again up to four times with an exponential back-off delay. You can customize this policy.
    *   **Configure Run After:** This is a fundamental concept for error handling. By default, Action B will only run if Action A succeeds. "Configure Run After" lets you change this. You can configure Action B to run *only if* Action A **has failed**, **is skipped**, or **has timed out**. This is how you build a `try-catch` block, for example, to send an error notification email if a SharePoint action fails.
    *   **Secure Inputs/Outputs:** When toggled on, the inputs and outputs of an action will be redacted and hidden from the flow's run history. This is **essential** for any action that handles sensitive data like passwords, API keys, or personal information.

> [!IMPORTANT]
> The single most powerful debugging technique in Power Automate is to inspect the **raw inputs and outputs** of a failed (or successful) action in the run history. This will show you the exact JSON data that was sent to the connector and the exact JSON data that was received back, allowing you to pinpoint issues with formatting, null values, or incorrect data types.

---

## Part 2: The Core Toolbox - Data Operations Actions

*   Data Operations are a special group of built-in actions that do not connect to any external service. Their purpose is to manipulate, transform, and shape data *within your flow*. They are the Swiss Army knife for a flow developer.

### Action 1: `Compose`

*   **Core Purpose:** The simplest yet most versatile Data Operations action. Its official purpose is to construct a string or object, but it is universally used as a **debugging tool** and a **placeholder for dynamic content**.
*   **How it Works:** It has a single input field. Whatever you put in that input is what it produces as its output.
*   **Common Use Cases:**
    1.  **Debugging:** When your flow is failing, and you're not sure what a particular piece of dynamic content or an expression is evaluating to, you can add a `Compose` action right before the failing step. Put the problematic expression into the `Compose` input. Rerun the flow. Now you can look at the output of the `Compose` action in the run history and see the exact, final value, which often makes the problem obvious.
    2.  **Creating Complex Strings:**
        *   `concat('Task: ', triggerBody()?['Title'], ' is due on ', formatDateTime(triggerBody()?['DueDate'], 'd'))`
    3.  **Referencing Dynamic Content with a Clean Name:** Sometimes the name of a dynamic token is long and ugly (e.g., `'Get_item_details'['Editor']['DisplayName']`). You can put this token into a `Compose` action named "Compose Editor Name". All subsequent actions can now refer to the cleaner output of `outputs('Compose_Editor_Name')`.

### Action 2: `Initialize Variable` & Friends (`Set Variable`, `Increment/Decrement Variable`, `Append to...`)

*   **Core Purpose:** To create and manipulate variables that exist only for a single run of the flow. Variables are essential for scenarios requiring counters, aggregations, or temporary data storage.
*   **Key Rules:**
    *   You must `Initialize Variable` at the top level of your flow (i.e., not inside a loop or condition). This action declares the variable and sets its initial value and type.
    *   **Available Types:** `Boolean`, `Integer`, `Float` (number with decimals), `String`, `Object`, `Array`.
*   **The "Variable Family" of Actions:**
    *   **`Set Variable`:** Assigns a completely new value to a variable, overwriting the previous one.
    *   **`Increment Variable`:** Used for counters. Takes an `Integer` or `Float` variable and adds a specific value to it.
    *   **`Decrement Variable`:** Subtracts a value from a number variable.
    *   **`Append to string variable`:** Adds text to the end of an existing string variable.
    *   **`Append to array variable`:** Adds an item to an array.
*   **Realistic Scenario: Counting Invoice Line Items**
    1.  **`Initialize variable`:** `Name: varItemCount`, `Type: Integer`, `Value: 0`.
    2.  **`Get items` (from an 'Invoice Line Items' list):** Fetches all line items related to an invoice.
    3.  **`Apply to each` loop:** Runs for each line item returned from `Get items`.
    4.  **Inside the loop (`Increment variable`):** `Name: varItemCount`, `Increment by: 1`.
    5.  **After the loop:** Now you have the total count in `varItemCount` which you can use to update a "Total Items" column on the main invoice record.

### Action 3: `Parse JSON`

*   **Core Purpose:** To take a string of JSON-formatted text and convert it into a fully typed object with dynamic content tokens. This is the bridge from unstructured text to structured data.
*   **When It's Essential:**
    *   When an `HTTP` request action returns a JSON payload.
    *   When a trigger (like from Microsoft Forms) provides data in a JSON string (e.g., details from a complex question type).
    *   When a column in your data source (like a multi-line text field in SharePoint) is being used to store JSON data.
*   **How it Works:**
    1.  **`Content` Input:** You provide the string variable or dynamic content that contains the JSON text.
    2.  **`Schema` Input:** This is the magic. You need to provide a schema that describes the structure of the JSON. You don't need to write this by hand! Simply click **"Generate from sample"** and paste in a sample of the JSON you expect to receive. Power Automate will analyze it and generate the correct schema for you.
*   **Result:** After the `Parse JSON` action, all the properties defined in the schema (e.g., `customerName`, `orderID`, `items`) become available as first-class dynamic content tokens for you to use in subsequent actions.

### Flashcard Q&A: Data Operations

*   **Q:** You have a complex expression in your flow that isn't working correctly. What is the quickest and easiest action to use for debugging its value?
*   **A:** The `Compose` action. Put the expression in its input, run the flow, and check its output in the run history.

*   **Q:** What is the critical rule about where the `Initialize Variable` action must be placed in a flow?
*   **A:** It must be at the **top level** of the flow logic; it cannot be placed inside a loop or a conditional branch.

*   **Q:** When an action (like an HTTP request) gives you a single string of text that contains structured JSON data, what action must you use to be able to access the individual properties (like `name` or `id`) from that data?
*   **A:** The `Parse JSON` action.

---

## Part 3: The Workhorses - Logic and Control Actions

*   These actions don't talk to external systems; they direct the traffic and logic *inside* your flow. They are the `if` statements and `for` loops of Power Automate.

### Action 1: `Condition`

*   **Core Purpose:** To create an if/then/else block of logic.
*   **How it Works:** It provides a simple UI to build a logical condition. If the condition evaluates to `true`, the actions in the **"If yes"** branch are executed. If it evaluates to `false`, the actions in the **"If no"** branch are executed.
*   **Building a Condition:**
    *   **Value 1 (Left Side):** Typically a piece of dynamic content from a previous step (e.g., the `Status.Value` from a SharePoint item).
    *   **Operator:** A comparison operator (`is equal to`, `is greater than`, `contains`, `starts with`, etc.).
    *   **Value 2 (Right Side):** Typically a static value you are checking against (e.g., "Approved").
*   **Advanced Mode:** You can switch to advanced mode to write the entire condition as a single WDL expression. ` @equals(triggerBody()?['Status'], 'Approved')`.
*   **Grouping (AND/OR):** The UI allows you to add multiple rows to your condition and group them with `AND` (all rows must be true) or `OR` (any row can be true) logic.

### Action 2: `Switch`

*   **Core Purpose:** An alternative to a series of nested `Condition` actions. A `Switch` is ideal when you need to check a *single value* against multiple possible outcomes.
*   **How it Works:**
    *   **`On` Input:** You provide the single piece of dynamic content you want to evaluate (e.g., the `Outcome` of an approval action).
    *   **`Case` Blocks:** You then create a `Case` block for each possible value of the `On` input. For example, a `Case` where `Outcome` equals "Approve", another `Case` where `Outcome` equals "Reject", and so on.
    *   **`Default` Block:** A `Default` case will run if the `On` input's value doesn't match any of the defined `Case` blocks.
*   **Benefit:** `Switch` is much cleaner, more readable, and more performant than building a "Christmas tree" of nested if/else conditions.

### Action 3: `Apply to each` (Loop)

*   **Core Purpose:** To perform one or more actions for every single item in an **array** (a list of items).
*   **When it's Used (Automatically):** Power Automate is smart. If you are inside an action and you select a piece of dynamic content that is an array, the designer will **automatically** wrap your action in an `Apply to each` loop.
    *   **Example:** After a `When a new email arrives` trigger, if you select the `Attachments` token (which is an array, as an email can have multiple attachments) and try to put its `Name` into a `Create file` action, Power Automate will instantly create an `Apply to each` loop and place your `Create file` action inside it. The loop will run once for each attachment.
*   **Input:** The input to an `Apply to each` loop must be an array. Common sources of arrays are:
    *   The `value` output from a `Get items` (SharePoint) or `List rows` (Dataverse) action.
    *   The `Attachments` output from an email trigger.
    *   An array variable you have built yourself.
*   **Concurrency Control:** In the settings of an `Apply to each` loop, you can turn on "Concurrency Control." This allows the loop to run multiple iterations in parallel (up to 50 at a time), which can dramatically speed up your flow if the actions inside the loop are independent of each other. If order matters, leave concurrency turned off.

### Flashcard Q&A: Logic and Control

*   **Q:** What action would you use to create a simple two-path `if/then/else` logic in your flow?
*   **A:** The `Condition` action.

*   **Q:** You have an approval flow and need to perform different actions based on whether the outcome is "Approve," "Reject," "Needs More Info," or "Delegate." Which control action would be cleaner and more efficient than using nested `Condition`s?
*   **A:** The `Switch` action.

*   **Q:** Why does Power Automate sometimes automatically add an `Apply to each` loop to your flow?
*   **A:** It does this when you use a dynamic content token that represents an **array** (a list of items), as it needs to perform the action on each item in that list individually.

---

## Part 4: The Pillars of Business Apps - Common Connector Actions

*   These are the actions from the most common connectors that form the backbone of countless business process automations.

### Category 1: SharePoint Connector Actions

*   **`Get items`:**
    *   **Purpose:** Retrieves multiple items from a SharePoint list.
    *   **Key Inputs:** `Site Address`, `List Name`.
    *   **Power Feature (`Filter Query`):** The most important input is the `Filter Query`, where you can write an OData query to filter the data on the server *before* it's sent to your flow. This is the #1 way to make your flow efficient.
        *   `Status eq 'Pending'` (Note: OData uses `eq` not `=`)
        *   `DueDate lt '2023-01-01'` (`lt` is "less than")
    *   `Top Count`: Limits the number of items returned.
*   **`Create item`:**
    *   **Purpose:** Adds a new single item (row) to a SharePoint list.
    *   **Inputs:** `Site Address`, `List Name`, and a field for each column.
*   **`Update item`:**
    *   **Purpose:** Modifies an existing single item in a SharePoint list.
    *   **Key Inputs:** `Site Address`, `List Name`, `Id` (The unique ID of the item to update), and the fields you want to change.
    *   > [!IMPORTANT]
        > If you leave a field blank in `Update item`, it will erase any existing value in that field! If you only want to update one field, only fill out that field (and the required ID).

### Category 2: Office 365 Outlook Connector Actions

*   **`Send an email (V2)`:**
    *   **Purpose:** The workhorse for all notifications. Sends an email from your (or a shared) mailbox.
    *   **Key Inputs:** `To`, `Subject`, `Body`.
    *   **Advanced Options:** Lets you set `From (Send as)`, `Cc`, `Bcc`, `Attachments`, and `Importance`.
    *   The body can be rich HTML, allowing you to include tables, links, and formatted text for professional-looking notifications.
*   **`Get Emails (V3)`:**
    *   **Purpose:** Retrieves emails from a mailbox. This is an *action*, not a trigger. You might use it in a scheduled flow to process emails that arrived earlier.
    *   **Key Inputs:** `Folder`, `Fetch Only Unread Messages`, `Search Query` (uses the Outlook search syntax to perform powerful server-side filtering).

### Category 3: Approvals Connector Actions

*   **`Start and wait for an approval`:**
    *   **Purpose:** The standard action for creating a formal, single-stage approval process.
    *   **Behavior:** This action **pauses** the flow's execution until the approver(s) respond.
    *   **Key Inputs:**
        *   `Approval type`: "Approve/Reject - Everyone must approve," "Approve/Reject - First to respond," "Custom Responses."
        *   `Assigned To`: A semicolon-separated list of approver emails.
        *   `Title`: The title of the approval request.
        *   `Details`: The main body text, where you should include all necessary information using dynamic content.
        *   `Item Link`: A URL link back to the item being approved, which is crucial for usability.
    *   **Outputs:** `Outcome` (e.g., "Approve"), `Responses` (an array containing comments from each approver).

### Category 4: Dataverse Connector Actions

*   **`Add a new row`:**
    *   **Purpose:** Creates a new record in a Dataverse table. Equivalent to `Create item`.
*   **`Update a row`:**
    *   **Purpose:** Modifies an existing record.
    *   **Key Input (`Row ID`):** Requires the unique GUID of the Dataverse record to update.
*   **`List rows`:**
    *   **Purpose:** Retrieves multiple rows. Equivalent to `Get items`.
    *   **Power Features:** `Filter rows`, `Sort by`, `Select columns`. Dataverse supports a more extensive and powerful OData query syntax than SharePoint. You can filter based on values in related tables (e.g., `_parentcustomerid_value eq 'GUID'`).
*   **`Relate rows` / `Unrelate rows`:**
    *   **Purpose:** Specifically for managing the N:N (many-to-many) relationships in Dataverse. Instead of updating a record, you use these actions to create or remove the link between two existing records.

### Category 5: HTTP Connector Actions

*   **Core Purpose:** To communicate with any REST API that doesn't have a dedicated connector. This is a premium action.
*   **Key Inputs:**
    *   **`Method`:** The HTTP verb to use: `GET` (retrieve data), `POST` (create data), `PUT`/`PATCH` (update data), `DELETE` (remove data).
    *   **`URI`:** The full URL of the API endpoint you want to call.
    *   **`Headers`:** Key-value pairs for request headers (e.g., `Content-Type: application/json`, `Authorization: Bearer <token>`).
    *   **`Body`:** The data payload to send with a `POST`, `PUT`, or `PATCH` request, typically a JSON string constructed using the `JSON()` function or `Compose` action.
    *   **`Authentication`:** A built-in way to handle various authentication schemes (Basic, Client Certificate, Azure AD OAuth).
*   **Output:** The main output is the `Body` from the API's response, which often needs to be passed to a `Parse JSON` action to be used.

### Flashcard Q&A: Common Connector Actions

*   **Q:** In the SharePoint `Get items` action, what is the most important parameter to configure for performance when working with large lists?
*   **A:** The `Filter Query` parameter, where you write an OData query to have SharePoint filter the data on the server *before* sending it to your flow.

*   **Q:** Your flow needs to pause and wait for a manager to approve a request before proceeding. What is the standard action to achieve this?
*   **A:** The `Start and wait for an approval` action from the Approvals connector.

*   **Q:** Your flow receives a response from an external API via the `HTTP` action. What action is almost always required immediately afterward to make the response data usable?
*   **A:** The `Parse JSON` action, to convert the JSON response body from a text string into usable dynamic content tokens.
