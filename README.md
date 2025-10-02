# Power-Automate-Interview-Prep
I swear there are no sources available online

### **1. Power Automate Basics**
**Types of Flows**:
| Type | Use Case | Trigger |
|------|----------|---------|
| **Cloud Flows** | Automate cloud-based tasks. | Manual, scheduled, or event-based. |
| **Desktop Flows** | Automate desktop apps (RPA). | UI interactions (e.g., clicks, keyboard). |
| **Business Process Flows** | Guide users through processes (e.g., approvals). | Model-driven apps. |

**Key Concepts**:
- **Trigger**: Starts the flow (e.g., "When a new email arrives").
- **Actions**: Steps performed (e.g., "Send an email," "Create a SharePoint item").
- **Connectors**: Pre-built integrations (e.g., Outlook, SharePoint, SQL).
- **Expressions**: Dynamic values (e.g., `triggerBody()?, outputs('Action_Name')`).

---

### **2. Cloud Flow Triggers**
**Common Triggers**:
| Trigger | Example Use Case |
|---------|------------------|
| **Manual (Button)** | Run on-demand (e.g., "Send Reminder"). |
| **Scheduled** | Run at specific times (e.g., daily reports). |
| **When an item is created/modified (SharePoint)** | Sync SharePoint lists. |
| **When a new email arrives (Outlook)** | Auto-sort emails. |
| **HTTP Request (API)** | Trigger via external systems. |
| **Power Apps (Button)** | Run from a Power App. |

**Recurrence Trigger Syntax**:
```json
{
  "frequency": "Day",
  "interval": 1,
  "hours": [8],
  "minutes": [0]
}
```

---

### **3. Actions**
**Common Actions**:
| Category | Actions | Example |
|----------|---------|---------|
| **Data Operations** | `Compose`, `Initialize Variable`, `Parse JSON` | `Compose: concat('Hello ', triggerBody()?['Name'])` |
| **Conditionals** | `Condition`, `Switch` | `Condition: @equals(triggerBody()?['Status'], 'Approved')` |
| **Loops** | `Apply to Each`, `Do Until` | `Apply to Each: outputs('Get_Items')?['value']` |
| **SharePoint** | `Get Items`, `Create Item`, `Update Item` | `Get Items: Site Address, List Name` |
| **Outlook** | `Send Email`, `Get Emails` | `Send Email: To, Subject, Body` |
| **Approvals** | `Start and Wait for Approval` | `Approvers: user@domain.com` |
| **HTTP** | `HTTP Request` (REST APIs) | `Method: GET, URI: 'https://api.example.com/data'` |
| **Dataverse** | `Create Record`, `Update Record` | `Table Name: Accounts, Item: {Name: 'New Account'}` |

**Expressions in Actions**:
- Reference dynamic content: `outputs('Previous_Action')?['body/property']`
- Use functions: `@concat('Name: ', triggerBody()?['Name'])`

---

### **4. Control Flow Actions**
**Conditionals**:
- **Condition**: `If yes/no` branches.
  ```plaintext
  Condition: @greater(triggerBody()?['Amount'], 1000)
  If yes: Send approval email.
  If no: Log to SharePoint.
  ```
- **Switch**: Multiple cases.
  ```plaintext
  Switch: @triggerBody()?['Status']
  Case "Approved": Update database.
  Case "Rejected": Notify manager.
  ```

**Loops**:
- **Apply to Each**: Iterate over arrays.
  ```plaintext
  Apply to Each: outputs('Get_Items')?['value']
  Action: Send email for each item.
  ```
- **Do Until**: Repeat until condition met.
  ```plaintext
  Do Until: @equals(variables('Counter'), 10)
  Action: Increment variable.
  ```

**Scope & Error Handling**:
- **Scope**: Group actions (e.g., for error handling).
- **Configure Run After**: Set action to run after success/failure.
- **Terminate**: Stop flow with status (`Succeeded`, `Failed`, `Cancelled`).

---

### **5. Expressions & Functions**
**Common Functions**:
| Category | Functions | Example |
|----------|-----------|---------|
| **String** | `concat()`, `substring()`, `replace()`, `split()` | `concat('Hello ', triggerBody()?['Name'])` |
| **Logical** | `equals()`, `greater()`, `and()`, `or()`, `not()` | `@and(greater(Amount, 100), equals(Status, 'Approved'))` |
| **Math** | `add()`, `sub()`, `mul()`, `div()` | `add(10, 5)` |
| **Date/Time** | `utcNow()`, `addDays()`, `formatDateTime()` | `formatDateTime(utcNow(), 'yyyy-MM-dd')` |
| **Array** | `length()`, `first()`, `last()`, `skip()`, `take()` | `first(outputs('Get_Items')?['value'])` |
| **JSON** | `json()`, `parseJSON()` | `parseJSON(triggerBody()?['Response'])` |
| **Variables** | `variables('varName')`, `setVariable()` | `setVariable('Counter', add(variables('Counter'), 1))` |

**Work with JSON**:
- Parse JSON: `parseJSON(triggerBody()?['Response'])`
- Stringify: `json(variables('MyObject'))`

---

### **6. Connectors**
**Popular Connectors**:
| Connector | Use Case |
|-----------|----------|
| **SharePoint** | Manage lists/libraries. |
| **Outlook/Office 365** | Send/read emails, manage calendars. |
| **SQL Server** | Query/update databases. |
| **Excel Online** | Read/write Excel files. |
| **Teams** | Post messages, manage channels. |
| **HTTP** | Call REST APIs. |
| **Dataverse** | CRUD operations on tables. |
| **Power Apps** | Trigger flows from apps. |
| **Approvals** | Create approval workflows. |
| **AI Builder** | Use pre-built AI models (e.g., text recognition). |

**Custom Connectors**:
- Create for unsupported APIs (use OpenAPI/Swagger).

---

### **7. Working with Data**
**SharePoint**:
- **Get Items**: Filter with OData.
  ```plaintext
  Filter Query: Status eq 'Approved' and Amount gt 1000
  ```
- **Create/Update Item**: Map fields dynamically.

**Dataverse**:
- **Create Record**:
  ```json
  {
    "name": "New Account",
    "revenue": 100000
  }
  ```
- **List Records**: Use `FetchXML` or OData filters.

**SQL Server**:
- **Execute Query**:
  ```sql
  SELECT * FROM Customers WHERE Country = 'USA'
  ```

**Excel**:
- **List Rows**: Get table data.
- **Add Row**: Append data to a table.

---

### **8. Approvals**
**Start an Approval**:
- **Action**: `Start and wait for an approval`.
- **Fields**:
  - **Approval Type**: "Approve/Reject" or "Custom Responses".
  - **Assigned To**: Email or user ID.
  - **Details**: Title, description, item link.

**Handle Responses**:
- Use `Condition` to check `Response` (e.g., `@equals(outputs('Approval')?['Response'], 'Approve')`).

---

### **9. Error Handling**
**Best Practices**:
- **Scope Actions**: Group related actions.
- **Configure Run After**: Set actions to run after failure.
- **Use "Terminate"**: Stop flow on critical errors.
- **Retry Policies**: For HTTP actions (e.g., retry 3 times).

**Example**:
```plaintext
Scope: "Process Order"
  - Action 1: Get Order (SharePoint)
  - Action 2: Update Inventory (SQL)
    Configure Run After: Action 1 succeeds.
  - Action 3: Send Confirmation (Outlook)
    Configure Run After: Action 2 succeeds.
  - Action 4: Log Error (If any action fails)
    Configure Run After: Action 1, 2, or 3 fails.
```

---

### **10. Testing & Debugging**
**Tools**:
- **Test Panel**: Run flow manually with test data.
- **Run History**: Check past runs (success/failure details).
- **Peek Code**: View raw inputs/outputs.

**Debugging Tips**:
- Use `Compose` actions to inspect variables.
- Log errors to SharePoint/email:
  ```plaintext
  Compose: concat('Error: ', outputs('HTTP_Request')?['body/error/message'])
  ```
- **Retry Failed Runs**: Fix issues and resubmit.

---

### **11. Performance Optimization**
- **Avoid "Apply to Each" for Large Arrays**: Use `Select` or batch operations.
- **Limit Data**: Use `Top Count` or `Filter Query` in triggers/actions.
- **Parallel Branches**: Run independent actions concurrently.
- **Asynchronous Flows**: Use `Response` actions for long-running tasks.
- **Disable Tracking**: For high-volume flows (Admin Center).

---

### **12. Integration with Other Tools**
**Power Apps**:
- Trigger flows from apps using `PowerAutomate.Run()`.
- Pass parameters:
  ```powerfx
  PowerAutomate.Run("FlowName", {param1: "value1", param2: "value2"})
  ```

**Teams**:
- **Post Adaptive Cards**: Rich interactive messages.
- **Bot Flows**: Respond to Teams messages.

**Power BI**:
- Use `Power Automate` visual to trigger flows from reports.

**Azure Logic Apps**:
- Migrate complex flows to Logic Apps for enterprise scale.

---

### **13. Desktop Flows (RPA)**
**Key Actions**:
| Action | Example |
|--------|---------|
| **Launch App** | Open Excel/SAP. |
| **UI Elements** | Click button, enter text. |
| **Keyboard Shortcuts** | `Send Keys: Ctrl+C`. |
| **If/Else** | Check if window exists. |
| **Loops** | Process each row in Excel. |
| **Variables** | Store/reuse values. |
| **Error Handling** | Retry on failure. |

**Best Practices**:
- Use **UI Selectors** (not coordinates) for reliability.
- **Wait for Elements**: Avoid timing issues.
- **Modularize**: Break into subflows for reusability.

---

### **14. Business Process Flows**
**Components**:
- **Stages**: Steps in the process (e.g., "Qualify" → "Develop" → "Propose").
- **Data Steps**: Fields to collect (e.g., "Budget," "Timeline").
- **Workflows**: Automate actions between stages.

**Example**:
```plaintext
Business Process Flow: "Opportunity Management"
  Stage 1: Qualify (Fields: Customer Name, Budget)
  Stage 2: Develop (Fields: Proposal Sent)
  Stage 3: Propose (Fields: Contract Signed)
```

---

### **15. Admin & Governance**
**Environment Strategies**:
- **Default**: Development/testing.
- **Production**: User-facing, locked down.
- **Sandbox**: Pre-prod testing.

**Data Loss Prevention (DLP)**:
- **Policies**: Restrict connector combinations (e.g., block SharePoint + personal email).
- **Admin Center**: Monitor compliance.

**Licensing**:
- **Per User**: Assign to individuals.
- **Per Flow**: For unattended flows (e.g., scheduled).

---
### **16. Power Automate Expressions Cheat Sheet**
| Task | Expression |
|------|------------|
| Get current date/time | `utcNow()` |
| Format date | `formatDateTime(utcNow(), 'yyyy-MM-dd')` |
| Add days to date | `addDays(utcNow(), 7)` |
| Check if empty | `@empty(triggerBody()?['Name'])` |
| Concatenate strings | `concat('Hello ', triggerBody()?['Name'])` |
| Convert to uppercase | `toUpper('hello')` |
| Get first item in array | `first(outputs('Get_Items')?['value'])` |
| Filter array | `@filter(outputs('Get_Items')?['value'], item()?['Status'] == 'Approved')` |
| Parse JSON | `parseJSON(triggerBody()?['Response'])` |
| String contains | `@contains(triggerBody()?['Email'], '@domain.com')` |

---

### **17. Common Use Cases**
| Scenario | Flow Type | Key Actions |
|----------|-----------|-------------|
| **Email Automation** | Cloud Flow | Trigger: "New email"; Actions: "Send email," "Create SharePoint item." |
| **Document Approval** | Cloud Flow | Trigger: "File uploaded to SharePoint"; Actions: "Start approval," "Move file." |
| **Data Sync** | Cloud Flow | Trigger: "Recurrence"; Actions: "Get SQL data," "Update Dataverse." |
| **Notifications** | Cloud Flow | Trigger: "SharePoint item modified"; Actions: "Post to Teams." |
| **RPA (Invoice Processing)** | Desktop Flow | Actions: "Open Excel," "Extract data," "Enter into SAP." |
| **Chatbot Automation** | Cloud Flow | Trigger: "Teams message"; Actions: "Call AI Builder," "Reply with adaptive card." |

---

### **18. Limits & Quotas**
| Resource | Limit (Per Flow) |
|----------|------------------|
| **Runtime** | 30 days (default timeout). |
| **Actions** | 250 actions per run. |
| **Loops** | 100,000 iterations. |
| **HTTP Requests** | 100 MB response size. |
| **Concurrency** | 100 parallel runs (can be increased). |

**Avoid Throttling**:
- Add delays (`Delay` action) between API calls.
- Use `Do Until` with retries for rate-limited APIs.

---
### **19. Power Automate CLI (Command Line)**
| Command | Purpose |
|---------|---------|
| `pac flow init` | Initialize a flow project. |
| `pac flow export` | Export flow to file. |
| `pac flow import` | Import flow to environment. |
| `pac auth create` | Authenticate to Power Platform. |

---
### **20. Resources**
- **Docs**: [Microsoft Power Automate Docs](https://learn.microsoft.com/en-us/power-automate/)
- **Templates**: [Power Automate Templates](https://flow.microsoft.com/en-us/templates/)
- **Community**: [Power Users Forum](https://powerusers.microsoft.com/)
- **Tools**:
  - [Flow Checker](https://flow.microsoft.com/en-us/flow-checker/) (performance tips).
  - [Postman](https://www.postman.com/) (test APIs).

---
**Pro Tip**: Use `Ctrl+Space` in the expression editor for autocomplete!
Need a deeper dive on any topic (e.g., OData queries, custom connectors)?
