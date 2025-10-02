# The Ultimate Masterclass on Working with Data in Power Automate

## Part 1: The Conceptual Foundation - Data as the Lifeblood of Automation

### Introduction: The Flow is the Process, the Data is the Purpose

*   At the heart of every meaningful business process automation is **data**. A flow without data is like a factory with no raw materials—it has the machinery to do work, but nothing to work on. The purpose of a flow is to receive data from one system, transform or enrich it, make decisions based on it, and then deliver it to another system or to a user.
*   "Working with data" is not a single topic; it is the central theme that runs through every trigger, action, and expression you will ever write in Power Automate.
*   This masterclass will go beyond the basics of single actions. It will explore the patterns, strategies, and best practices for efficiently querying, transforming, and manipulating data from the most common business data sources: SharePoint, Dataverse, SQL Server, and Excel. Understanding these patterns is what allows you to build scalable, high-performance flows that are the backbone of digital transformation.

### The Universal Data Language: JSON

*   To understand how Power Automate "thinks" about data, you must understand **JSON (JavaScript Object Notation)**. Underneath the user-friendly interface of dynamic content, every piece of data moving through your flow is structured as JSON.
*   **A JSON Object:** Represents a single item. It's a collection of key-value pairs, enclosed in curly braces `{}`.
    ```json
    {
      "Title": "Project Phoenix",
      "Priority": "High",
      "Budget": 50000
    }
    ```
    *   This is how a single SharePoint item or Dataverse row is represented.
*   **A JSON Array:** Represents a list of items. It's an ordered collection of objects or values, enclosed in square brackets `[]`.
    ```json
    [
      { "Title": "Project Phoenix", "Priority": "High" },
      { "Title": "Project Neptune", "Priority": "Medium" }
    ]
    ```
    *   This is how the output of a `Get items` or `List rows` action is represented.
*   **Why this matters:** When you use an expression like `outputs('Get_item')?['body/Title']`, you are simply navigating a JSON object. This knowledge is your key to debugging and writing advanced expressions. It demystifies what is happening behind the scenes and empowers you to handle any data structure a service might throw at you.

### Server-Side vs. Client-Side Filtering: The Performance Golden Rule

*   This is the most important principle for building efficient data-centric flows.
*   **Server-Side Filtering (Efficient & Scalable):**
    *   **The Principle:** You tell the data source (the server) *exactly* which data you want, and the server does the hard work of finding and returning only that small, relevant subset.
    *   **The Mechanism:** Using the **`Filter Query`** or **`Filter rows`** parameter in your data retrieval action (e.g., SharePoint `Get items` or Dataverse `List rows`). The query is written in a language the server understands (OData).
    *   **Result:** The flow receives only the data it needs. This is extremely fast and consumes minimal resources and API calls. **This is the way of the professional.**
*   **Client-Side Filtering (Inefficient & Brittle):**
    *   **The Principle:** You ask the data source for a large, unfiltered chunk of data. You then bring all that data into the flow's memory and use a Power Automate `Condition` or `Filter array` action to find what you need.
    *   **The Mechanism:** Leaving the `Filter Query` blank in a `Get items` action and then placing a `Condition` inside the `Apply to each` loop.
    *   **Result:** This is slow, wastes API calls (as you are pulling data you will just throw away), and is subject to the connector's limits on the number of records it can return in a single call (often 5,000-10,000, which can be further limited by pagination settings). **This is the path of the novice and should be avoided at all costs.**

> [!IMPORTANT]
> The #1 skill for working with data is learning how to write effective **server-side filter queries**. Never retrieve more data than you absolutely need. Always filter at the source.

---

## Part 2: Working with SharePoint Data

*   SharePoint is often the entry point for citizen developers due to its availability within Microsoft 365. Its quirks and query language are essential knowledge.

### The Key to Performance: OData Filter Queries in `Get items`

*   The SharePoint `Get items` action has a `Filter Query` field in its advanced options. This is where you perform your server-side filtering using the **OData (Open Data Protocol)** syntax.
*   OData syntax is similar to SQL but has its own nuances.

#### OData Cheat Sheet for SharePoint

| Comparison | OData Syntax | Example |
| :--- | :--- | :--- |
| Equals | `eq` | `Status eq 'Approved'` (Note the single quotes for strings) |
| Not Equals | `ne` | `Status ne 'Complete'` |
| Greater Than | `gt` | `Amount gt 1000` |
| Greater Than/Equals | `ge` | `Amount ge 1000` |
| Less Than | `lt` | `DueDate lt '2023-01-01T00:00:00Z'` (Dates must be in ISO 8601 format) |
| Less Than/Equals | `le` | `DueDate le '@{utcNow()}'` (Using an expression to get the current date) |
| And | `and` | `Status eq 'Approved' and Amount gt 1000` |
| Or | `or` | `Priority eq 'High' or IsUrgent eq true` (Yes/No fields are `true` or `false`) |
| Starts With | `startswith(ColumnName, 'Value')` | `startswith(Title, 'INV')` |

*   > [!WARNING]
    > Column names in OData queries are the **internal names**, not the display names. If you create a column named "Due Date", SharePoint's internal name for it will be `DueDate`. If you later rename it to "Target Date", the internal name often remains `DueDate`. You can find the internal name by going to List Settings and clicking on the column. The internal name will be in the URL after `&Field=`.

*   **Complex Columns:** You cannot directly filter on the value of a Person, complex Choice, or Lookup column in a SharePoint OData query from a flow. You can only filter on their **ID**.
    *   `AuthorId eq 15` (Correct) -> Filter by the Author's user ID.
    *   `Author/Title eq 'John Doe'` (Incorrect) -> This kind of expanded query is not supported by the SharePoint connector. This is a major limitation compared to Dataverse.

### Creating and Updating SharePoint Items

*   The `Create item` and `Update item` actions present a form with fields corresponding to your list's columns.
*   **Dynamic Mapping:** The power comes from using dynamic content from a trigger or a previous action to populate these fields.
*   **Handling Complex Column Types (`Update item` example):**
    *   **Title (Single line of text):** `outputs('Get_email_details')?['Subject']`
    *   **AssignedTo (Person):** You must provide a **Claims token** in the person field. The easiest way to get this is from another person field or by using the `User().Email` from a Power Apps trigger. You can't just type an email address; you must select it from the people picker or provide the full claims value.
        *   **For the current user from a Power Apps trigger:** `triggerBody()?['User']?['Email']`
    *   **Status (Choice):** For a choice column, you must select "Enter custom value" and provide the exact string that matches one of the choices (e.g., "In Progress").
    *   **Customer (Lookup):** For a lookup column, you must provide the **numeric `ID`** of the item from the list you are looking up to. You often need a `Get items` action first to find the `ID` of the item you want to link to.

### Large Lists and Pagination

*   The `Get items` action will, by default, only return the first 100 items. If you need more, you must go into the action's **Settings (`...`)** and enable **Pagination**.
*   You can set a `Threshold` to retrieve up to a certain number of items (e.g., 5,000). The flow will then make multiple API calls in the background to fetch all the data, page by page.
*   > [!CAUTION]
    > While pagination lets you retrieve more than 100 items, it does not solve the client-side filtering problem. It's still vastly more efficient to filter at the source to get 50 items than it is to paginate and retrieve 5,000 items to then filter them in the flow. Only use pagination when you genuinely need to process a large set of records.

### Flashcard Q&A: SharePoint Data

*   **Q:** What is the most important parameter to use in the SharePoint `Get items` action to ensure your flow is efficient and scalable?
*   **A:** The **`Filter Query`** parameter, where you write a server-side OData query.

*   **Q:** You need to filter a SharePoint list for all items where the "Project Manager" (a Person column) is the current user. Can you do this directly in the `Filter Query`?
*   **A:** No, you cannot directly filter on the text value (e.g., name or email) of a complex column like Person in an OData query. You must filter by the person's SharePoint user `ID`.

*   **Q:** By default, how many items does the SharePoint `Get items` action return, and how do you retrieve more?
*   **A:** By default, it returns **100** items. You must enable **Pagination** in the action's settings and set a higher threshold to retrieve more.

---

## Part 3: Working with Dataverse Data

*   Dataverse is the premier data source, and its connector provides more power, flexibility, and better performance than the SharePoint connector.

### Querying with `List rows`: OData and FetchXML

*   The Dataverse `List rows` action is the equivalent of SharePoint's `Get items`. It also has powerful server-side filtering capabilities.
*   **1. OData `Filter rows`:**
    *   The syntax is very similar to SharePoint's OData, but it is more powerful and has better support for complex data types.
    *   **Filtering on Lookups (The Killer Feature):** You can directly filter based on values in a related table. The syntax uses the logical name of the lookup relationship.
        *   `_primarycontactid_value eq 'GUID_of_contact'` (Filter an Account by its Primary Contact's ID).
        *   To filter on a field of the related record, you must use a more complex `Fetch XML` query.
    *   **Filtering on Choice columns (`statuscode`):** `statuscode eq 755240001` (You filter by the numeric value of the choice, not its text label).
*   **2. `FetchXML Query`:**
    *   FetchXML is a proprietary XML-based query language for Dataverse. It is much more powerful than OData and is the go-to tool for complex queries.
    *   **Capabilities:**
        *   Perform complex joins across multiple related tables (`link-entity`).
        *   Filter on fields from related tables (e.g., "Get all Accounts where the related Primary Contact's city is 'Seattle'").
        *   Perform aggregations (group by, sum, average).
    *   **How to Build:** You don't write FetchXML by hand. You use the **Advanced Find** tool in a Model-Driven App or the **FetchXML Builder** tool in the XrmToolBox to build your query using a graphical interface, and then you copy and paste the generated XML into the `FetchXML Query` field in your flow action.
*   **`Select columns` and `Expand Query`:**
    *   **`Select columns`:** Similar to SQL `SELECT`, this allows you to specify a comma-separated list of the column logical names you want to retrieve. This is a critical performance optimization.
    *   **`Expand Query`:** Allows you to retrieve columns from a related `1:N` table in a single query.

### Creating and Updating Dataverse Records

*   The `Add a new row` and `Update a row` actions are used for write operations.
*   **Data Structure:** The actions expect data in a key-value pair format, which is naturally represented by JSON.
    ```json
    {
      "name": "A. Datum Corporation (Sample)",
      "creditlimit": 100000,
      "numberofemployees": 500,
      "description": "A. Datum provides data storage solutions."
    }
    ```
*   **Handling Special Data Types (`Update a row` example):**
    *   **`name` (Text):** Simple string value from dynamic content.
    *   **`revenue` (Currency):** Simple number value.
    *   **`statuscode` (Choice):** Provide the **numeric value** of the choice.
    *   **`primarycontactid` (Lookup):** This is the most complex. To link this account to a contact, you must provide the lookup in the format `/contacts(GUID_of_contact)`. The `GUID_of_contact` is the unique ID of the contact record. You typically get this from a preceding `List rows` action.
    *   **`ownerid` (Owner):** Similar to a lookup, `/systemusers(GUID_of_user)`.
*   The "Upsert" Advantage: The `Update a row` action can also perform an "upsert" (update or insert). If you provide a Row ID that exists, it updates it. If you provide an ID that does not exist, it can create a new record with that ID.

### Relational Data: The `Relate rows` Action

*   For **Many-to-Many (N:N)** relationships, you don't use `Update a row`. Dataverse has a dedicated, highly efficient action for this.
*   **Use Case:** You have an "Events" table and a "Contacts" table with an N:N relationship between them. You want to register a specific contact for a specific event.
1.  You have the `GUID` for the Event record and the `GUID` for the Contact record.
2.  Use the **`Relate rows`** action.
3.  **Item ID:** The `GUID` of the first record (e.g., the Event).
4.  **Relationship:** Select the name of the N:N relationship from the dropdown.
5.  **Relate With:** The full resource URL of the second record: `https://.../api/data/v9.2/contacts(GUID_of_contact)`.

### Flashcard Q&A: Dataverse Data

*   **Q:** What are the two query languages you can use to filter rows in the Dataverse `List rows` action?
*   **A:** **OData** (in the `Filter rows` field) and **FetchXML** (in the `FetchXML Query` field).

*   **Q:** You need to retrieve a list of "Accounts" and filter them based on the "City" of their "Primary Contact." Which query language must you use to perform this type of join-based filter?
*   **A:** You must use **FetchXML**, as OData filters in flows cannot query across relationships in this way.

*   **Q:** When updating a **Lookup** field in a Dataverse record, what value do you need to provide?
*   **A:** The resource path to the record you want to link to, in the format `/pluraltablename(GUID)`. For example, `/contacts(1234abcd-...)`.

---

## Part 4: Working with SQL Server and Excel Data

### SQL Server: Raw Power and Precision

*   The SQL connector translates flow actions into direct SQL queries, offering high performance.
*   **Querying with `Get rows`:**
    *   The action has an OData `Filter Query`, but it's often more intuitive for SQL developers to use the more direct actions.
*   **`Execute a SQL query` (Read-Only):**
    *   This is the power user's tool for reading data. You can write *any* valid `SELECT` statement.
    *   **Syntax:** `SELECT [Column1], [Column2] FROM [dbo].[TableName] WHERE [Country] = 'USA'`
    *   **Security:** This action is powerful but can be risky if you concatenate user input directly into the query string, as this can lead to **SQL injection vulnerabilities**. For example, `SELECT * FROM T WHERE C = '@{triggerBody()?['userInput']}'`.
    *   **Best Practice:** Always use **Stored Procedures** when user input is involved in a query.
    *   **Output:** Returns a result set (an array of objects) that can be used in an `Apply to each` loop.
*   **`Execute a stored procedure` (Read & Write):**
    *   This is the most secure, maintainable, and recommended way to interact with SQL, especially for write operations (`INSERT`, `UPDATE`, `DELETE`).
    *   **How it Works:** You select the stored procedure from a dropdown. Power Automate automatically creates input fields for each parameter the stored procedure expects. You populate these with dynamic content.
    *   **Benefits:**
        1.  **Security:** Prevents SQL injection.
        2.  **Performance:** The query plan is pre-compiled and optimized on the SQL server.
        3.  **Maintainability:** Business logic is kept in the database, where it belongs, not scattered across dozens of flows.

### Excel: Simplicity with Strict Limitations

*   Working with Excel as a database is only suitable for simple scenarios.
*   **The Prerequisite: Data must be in a formal Excel Table.** The connector cannot read raw cells.
*   **`List rows present in a table`:**
    *   The core action for reading data. It retrieves all rows from a specified table in an Excel workbook.
    *   It has `Filter Query` capabilities, but they are extremely limited and unreliable compared to other sources. It is best to assume no server-side filtering is possible.
*   **`Add a row into a table`:**
    *   Appends a new row to the end of the specified table.
    *   Power Automate automatically creates input fields for each column header in your table.
*   **`Update a row`:**
    *   This is the trickiest action. To update a row, you need a way to uniquely identify it.
    *   **`Key Column`:** You must tell the action which column contains the unique identifier for each row (e.g., an `ID` or `SKU` column).
    *   **`Key Value`:** You provide the unique ID of the row you want to update.
    *   **The action then finds the first row** where the `Key Column` matches the `Key Value` and updates the other fields.
*   > [!WARNING]
    > **Concurrency and Locking:** The biggest problem with using Excel as a database is file locking. When your flow is writing to a row, it locks the entire Excel file. If another user or another flow run tries to access the file at the same time, the second operation will fail. This makes Excel completely unsuitable for any system with more than one user or a high frequency of updates.

### Flashcard Q&A: SQL and Excel Data

*   **Q:** What is the most secure and performant way to execute a complex `INSERT` or `UPDATE` operation against a SQL Server database from a flow?
*   **A:** Use the **`Execute a stored procedure`** action.

*   **Q:** What is the critical prerequisite for your data in an Excel file before you can use it as a data source in Power Automate?
*   **A:** The data must be formatted as a **Table** (using the Insert > Table feature in Excel).

*   **Q:** What is the primary reason that makes Excel an unsuitable backend for multi-user or high-frequency applications?
*   **A:** **File locking**. Concurrent write operations will fail, leading to data loss and unreliable automations.

---

## Part 5: Advanced Data Transformation Patterns

### Pattern 1: Shaping and Filtering Arrays with `Select` and `Filter array`

*   Sometimes, your server-side query can't get the data in the *exact* shape or subset you need. The `Select` and `Filter array` data operations actions allow you to perform these transformations *in the flow*.

*   **The `Select` Action:**
    *   **Purpose:** Takes an array of objects and creates a *new* array, but with a different shape. You can rename, remove, or add new properties to each object.
    *   **`From` Input:** The source array (e.g., `outputs('Get_items')?['body/value']`).
    *   **`Map` Input:** A key-value pair mapping.
        *   **Key (Left side):** The name of the new property in your output array.
        *   **Value (Right side):** An expression defining the value for that property.
    *   **Example:** Take a complex SharePoint item and create a simple array for an HTML table.
        ```
        From: outputs('Get_items')?['body/value']
        Map:
          "TaskTitle"   ->   item()?['Title']
          "AssignedTo"  ->   item()?['Author']?['DisplayName']
          "DueDate"     ->   formatDateTime(item()?['DueDate'], 'd')
        ```

*   **The `Filter array` Action:**
    *   **Purpose:** Performs a **client-side** filter on an array that already exists in your flow.
    *   **`From` Input:** The source array.
    *   **Condition:** A condition builder, just like the main `Condition` action.
    *   **Example:** You have an array of project tasks. You want to send one email with the high-priority tasks and a separate email with the low-priority ones.
        1.  `Get items` to retrieve all tasks.
        2.  `Filter array` on the `value` output where `item()?['Priority']?['Value']` **is equal to** `"High"`.
        3.  `Filter array` on the `value` output where `item()?['Priority']?['Value']` **is equal to** `"Low"`.
        4.  You now have two separate, smaller arrays to work with.

### Pattern 2: Building HTML Tables and CSVs

*   **The Actions:** Power Automate provides `Create HTML table` and `Create CSV table` actions.
*   **How they work:**
    *   **`From` Input:** Takes an array of objects (typically from `Get items` or a `Select` action).
    *   **Columns:** You can choose `Automatic` (it uses all properties) or `Custom`.
    *   **Custom Columns:** For each column in your table, you provide a `Header` (the column title) and a `Value` (an expression that extracts the data for that column from the item).
*   **The Result:** These actions output a single **string** containing a fully formatted HTML `<table`>...`</table>` or a multi-line CSV. This string can then be placed directly into the body of a `Send an email` action or saved to a file using `Create file`.

### The End-to-End Pattern: Query -> Shape -> Format -> Deliver

1.  **Query:** Use `Get items`, `List rows`, or `Execute a SQL query`. **Apply a server-side `Filter Query`** to get the smallest possible dataset.
2.  **Shape (Optional):** Use a **`Select`** action to cherry-pick only the columns you need and rename them to be user-friendly. This simplifies the next step.
3.  **Format:** Use a **`Create HTML table`** action, taking the output of the `Select` action as its input.
4.  **Deliver:** Use a **`Send an email`** action. In the body, place the `Output` dynamic content from the `Create HTML table` action. Add some simple CSS styling in the email body for a professional look.
    ```html
    <style>
      table { border-collapse: collapse; width: 100%; }
      th, td { border: 1px solid black; padding: 8px; text-align: left; }
      th { background-color: #f2f2f2; }
    </style>
    @{outputs('Create_HTML_table')}
    ```
This four-step pattern is one of the most common and powerful in all of Power Automate for creating automated reports.
