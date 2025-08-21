# Running WrenAI Locally: Full Setup & Troubleshooting Guide

## Overview
This document details the process of running WrenAI locally using Docker, connecting to Azure OpenAI, configuring models, and troubleshooting common issues. It includes step-by-step instructions, configuration examples, and real-world debugging notes.

---

## 1. Prerequisites
| Tool           | Version/Notes                |
|----------------|-----------------------------|
| Docker Desktop | Latest (Linux containers)    |
| Python         | 3.12.x (for local dev)       |
| Node.js        | For UI (if running manually) |
| Azure Account  | With OpenAI resource access  |
| PostgreSQL     | (via Docker, if needed)      |
| Qdrant         | (via Docker, auto)                 |

---

## 2. Cloning and Preparing the Project
- Clone the WrenAI repository from GitHub. *Requires Git.
```powershell
git clone https://github.com/Canner/WrenAI.git
```
- Folder structure should include `docker/`, `wren-ai-service/`, `wren-ui/`, etc.

---

## 3. Environment Configuration

### .env File (in `docker/`)
Create `.env` from the example:
```powershell
copy docker\env.example docker\.env
```

Edit `.env` to include your Azure OpenAI credentials:
```properties
OPENAI_API_KEY=<your-azure-openai-key>
OPENAI_API_BASE=https://<your-resource-name>.openai.azure.com/
OPENAI_API_TYPE=azure
OPENAI_API_VERSION=2024-12-01-preview
```

Set service ports and other variables as needed.

---

## 4. Database Setup
- **Qdrant** is started automatically via Docker Compose.
- No manual DB setup is needed unless you want to connect to your own database, which we did ([see Connecting to Your Own Database](#connecting-to-your-own-database)).

---

## 5. Model Deployment (Azure OpenAI)
| Model Type   | Azure Deployment Name | Used For         | Embedding Dim |
|--------------|----------------------|------------------|--------------|
| LLM          | gpt-4o               | Chat/Completion  | N/A          |
| Embedder     | text-embedding-3-large| Embeddings/RAG   | 3072         |

- Deploy these models in your Azure OpenAI resource.
- Use the **deployment name** (not the generic model name) in your config.

---

## 6. config.yaml Example
```yaml
type: llm
provider: litellm_llm
models:
  - alias: default
    model: gpt-4o  # Azure deployment name
    context_window_size: 1000000
    kwargs:
      max_tokens: 4096
      n: 1
      seed: 0
      temperature: 0
---
type: embedder
provider: litellm_embedder
models:
  - model: text-embedding-3-large  # Azure deployment name
    alias: default
    timeout: 120
---
type: document_store
provider: qdrant
location: http://qdrant:6333
embedding_model_dim: 3072
---
# ...other sections as needed
```

---

## 7. Running WrenAI

### Using Docker Compose
```powershell
docker compose -f docker\docker-compose.yaml up
```
- Access UI at [http://localhost:3000](http://localhost:3000)

### Manual (Dev) Run
- See `wren-ai-service/README.md` for Poetry/Justfile instructions.

---

## 8. Testing the API/Model

### Example Python Test
```python
from openai import AzureOpenAI
client = AzureOpenAI(
    api_version="2024-12-01-preview",
    azure_endpoint="https://<your-resource-name>.openai.azure.com/",
    api_key="<your-azure-openai-key>",
)
response = client.chat.completions.create(
    messages=[{"role": "user", "content": "I am going to Paris, what should I see?"}],
    model="gpt-4o"
)
print(response.choices[0].message.content)
```

#### Result Example
| Prompt                                      | Model   | Result (truncated)                |
|---------------------------------------------|---------|-----------------------------------|
| I am going to Paris, what should I see?     | gpt-4o  | "You should see the Eiffel Tower..." |

---

## 9. Troubleshooting & Debugging

### Common Issues & Fixes
| Issue/Log Message                                         | Cause & Solution                                                                 |
|----------------------------------------------------------|---------------------------------------------------------------------------------|
| `IsADirectoryError: .../config.yaml`                      | You created a folder, not a file. Delete folder, create file.                   |
| `NotFoundError: OpenAIException - Resource not found`     | Model/embedding not deployed or wrong name in config. Use Azure deployment name. |
| `You didn't provide an API key`                           | API key missing in `.env`. Add your Azure OpenAI key.                           |
| No `[WREN DEBUG]` output in logs                          | Docker not using your local code. Check build context/volume mounts.             |
| `version` is obsolete warning in Docker Compose           | Remove `version:` from `docker-compose.yaml` (optional, not an error).           |

### Debugging Tips
- Add print statements in `src/config.py` to verify environment/config loading.
- Use `docker exec -it <container_id> /bin/bash` to inspect running containers.
- Use `find / -name config.py` inside the container to locate your code.

---

## 10. Documentation & References
- [WrenAI Github](https://github.com/Canner/WrenAI)
- [WrenAI Docs](https://docs.getwren.ai/oss/concept/wren_ai_service)
- [Azure OpenAI](https://portal.azure.com)
- [Qdrant](https://qdrant.tech/)
- [Docker Compose](https://docs.docker.com/compose/)

---

## 11. OpenAI gpt-4o Example Prompts & Results Table
| Prompt                                      | Model   | Result (truncated)                |
|---------------------------------------------|---------|-----------------------------------|
| I am going to Paris, what should I see?     | gpt-4o  | "You should see the Eiffel Tower..." |
| What is the population of Asia in 2020?     | gpt-4o  | "The population of Asia in 2020..."   |

---

## 12. Notes
- Always use your Azure deployment names in config files.
- For full RAG/search, deploy an embedding model and set its dimension in config.
- If you change code, ensure Docker uses your local files (rebuild or use volume mounts).
- For advanced debugging, check logs and inspect containers directly.

---

## 13. Real Example Prompts and Results

### Dataset Explanation

**Overview**

This dataset simulates a realistic banking environment, including customers, accounts, employees, branches, transactions, loans, and summary views. It is designed to support a wide range of analytics, reporting, and natural language querying scenarios.

You can find the database documentation at: https://github.com/Phicks-debug/SQL-Generate-Data

**Tables and Views**

1. customers
   - Description: Stores customer personal information.
   - Key Columns: customer_id (PK), first_name, last_name, date_of_birth, email, phone_number, address, credit_score, is_active

2. employees
   - Description: Bank staff, including relationship managers and branch managers.
   - Key Columns: employee_id (PK), first_name, last_name, branch_id, role, hire_date

3. branches
   - Description: Physical bank branches.
   - Key Columns: branch_id (PK), branch_name, address, phone_number, manager_id (FK to employees), opening_date, assets_value, is_active
   - Relationships: manager_id → employees.employee_id

4. account_types
   - Description: Types of accounts (e.g., checking, savings).
   - Key Columns: account_type_id (PK), type_name, description

5. accounts
   - Description: Bank accounts held by customers.
   - Key Columns: account_id (PK), customer_id (FK), account_type_id (FK), branch_id (FK), relationship_manager_id (FK), balance, open_date, is_active
   - Relationships:
     - customer_id → customers.customer_id
     - account_type_id → account_types.account_type_id
     - branch_id → branches.branch_id
     - relationship_manager_id → employees.employee_id

6. customer_accounts
   - Description: Join table for many-to-many relationship between customers and accounts (if supported).
   - Key Columns: customer_id (FK), account_id (FK)
   - Relationships:
      - customer_id → customers.customer_id
      - account_id → accounts.account_id

7. loans
   - Description: Loan records for customers.
   - Key Columns: loan_id (PK), customer_id (FK), approving_employee_id (FK), amount, issue_date, due_date, status
   - Relationships:
      - customer_id → customers.customer_id
      - approving_employee_id → employees.employee_id

8. transactions
   - Description: Records of money transfers and payments.
   - Key Columns: transaction_id (PK), source_account_id (FK), destination_account_id (FK), amount, transaction_date, processing_employee_id (FK), description
   - Relationships:
      - source_account_id → accounts.account_id
      - destination_account_id → accounts.account_id
      - processing_employee_id → employees.employee_id

9. customer_account_summary (View)
   - Description: Summarizes account balances and activity per customer.
   - Note: This is a view, not a base table.

10. transaction_history (View)
     - Description: Provides a historical log of transactions, possibly with additional computed fields.
     - Note: This is a view, not a base table.

**Relationships Diagram (Textual)**

- customers ← (customer_id) → accounts
- accounts ← (account_type_id) → account_types
- accounts ← (branch_id) → branches
- accounts ← (relationship_manager_id) → employees
- branches ← (manager_id) → employees
- customer_accounts: customer_id → customers, account_id → accounts
- loans: customer_id → customers, approving_employee_id → employees
- transactions: source_account_id/destination_account_id → accounts, processing_employee_id → employees
- Views: customer_account_summary and transaction_history are based on the above tables

**Usage**

- This schema supports a wide range of banking queries, including customer analytics, account summaries, transaction histories, loan reporting, and more.
- It is suitable for text-to-SQL, BI, and LLM-driven analytics demos.

**Notes**

- All foreign key relationships are enforced for data integrity.
- Views are provided for convenience and reporting.
- The dataset is synthetic and safe for public demos.

---

*For questions or schema diagrams, see the SQL files or contact the repository maintainer.*

---

<br>
The three prompts below together will best showcase the range and power of the text-to-SQL solution.

Summary Table:

| Prompt #	| Complexity	| SQL Features Demonstrated |	Why Show It? |
|---|---------|----------------------------------------|-------------------------------------------------|
| 1	| Easy | GROUP BY, COUNT	| Basic aggregation/data quality |
| 2	| Medium	| JOIN, WHERE, ORDER BY, LIMIT	| Multi-table, real reporting |
| 3	| Hard	| Multi-join, GROUP BY, aggregation	| Advanced analytics, BI scenario |

### Easy Prompt Example

**Prompt:**
List all the account types and the number of accounts they have. Order them in descending order by the number of accounts.

**Why:**
- Demonstrates simple aggregation and grouping.
- Easy to verify correctness.
- Shows the model can answer basic data quality questions.

**Answer:**
Here is the list of account types along with the number of accounts they have, ordered from the highest to the lowest:

1. Non-Profit Organization Account: 28 accounts
2. Health Savings Account (HSA): 27 accounts
3. Individual Retirement Account (IRA) Savings: 25 accounts
4. Foreign Currency Account: 24 accounts
5. Certificates of Deposit: 24 accounts

...and more results omitted. Let me know if you'd like to see the full list!

Considering the limit of the context window, we retrieve up to 500 rows of results to generate the answer.

**Table of Results:**

| account_type_name      | account_count |
|------------|----------------|
| Non-Profit Organization Account  | 28              |
| Health Savings Account (HSA)  | 27             |
| Individual Retirement Account (IRA) Savings  | 25              |
| Foreign Currency Account  | 24             |
| Certificates of Deposit  | 24              |
| High-Yield Online Savings  | 22              |
| Senior Advantage  | 22              |
| Money Market Account  | 22              |
| Basic Checking  | 20             |
| Student Checking  | 19             |
| Premium Savings  | 19             |
| Platinum Checking  | 18             |
| Gold Priority Banking  | 12             |
| Small Business Checking  | 10             |
| Business Advantage  | 8             |

**SQL Query:**
```sql
SELECT
  "public_account_types"."type_name" AS "account_type_name",
  COUNT("public_accounts"."account_id") AS "account_count"
FROM
  "public_account_types"
  LEFT JOIN "public_accounts" ON "public_account_types"."account_type_id" = "public_accounts"."account_type_id"
GROUP BY
  "public_account_types"."type_name"
ORDER BY
  COUNT("public_accounts"."account_id") DESC
```

You’re seeing Wren SQL. If you want to see this query on our own database, click “Show original SQL” to get the exact syntax.
<details>
<summary>Show original SQL</summary>

```sql
SELECT
  account_type_name,
  account_count
FROM
  (
    SELECT
      public_account_types.type_name AS account_type_name,
      COUNT(public_accounts.account_id) AS account_count,
      COUNT(public_accounts.account_id)
    FROM
      (
        SELECT
          public_account_types.account_type_id,
          public_account_types.type_name
        FROM
          (
            SELECT
              __source.account_type_id AS account_type_id,
              __source.type_name AS type_name
            FROM
              banking_db."public".account_types AS __source
          ) AS public_account_types
      ) AS public_account_types
      LEFT JOIN (
        SELECT
          public_accounts.account_id,
          public_accounts.account_type_id
        FROM
          (
            SELECT
              __source.account_id AS account_id,
              __source.account_type_id AS account_type_id
            FROM
              banking_db."public".accounts AS __source
          ) AS public_accounts
      ) AS public_accounts ON public_account_types.account_type_id = public_accounts.account_type_id
    GROUP BY
      public_account_types.type_name
    ORDER BY
      COUNT(public_accounts.account_id) DESC NULLS LAST
  )
```
</details>

<br>

**Chart:**

<img width="700" height="320" alt="visualization" src="https://github.com/user-attachments/assets/59d1a044-d964-41dc-a157-a49ebcc4b791" />


The query is Correct. The answer and SQL match the intent, and the chart visualizes the duplicate counts.

---

### Medium Prompt Example

**Prompt:**
Show the last 10 transactions for customer Mark Lewis, including the transaction amount, date, and source/destination account numbers.

**Why:**
- Involves joining tables, filtering, and selecting multiple columns.
- Demonstrates the model’s ability to handle multi-table queries and more complex reporting.

**Answer:**
Here are the last 10 transactions for customer Mark Lewis, including the transaction amount, date, and source/destination account numbers:

1. Date: 2025-08-12 00:19:48.296814
   - Amount: $597.27
   - Source Account: ACCT-15004839
   - Destination Account: None

2. Date: 2025-07-20 15:41:54.322154
   - Amount: $38.76
   - Source Account: None
   - Destination Account: ACCT-90471014

3. Date: 2025-07-18 12:03:15.286367
   - Amount: $9,133.70
   - Source Account: ACCT-90471014
   - Destination Account: ACCT-72904922

4. Date: 2025-07-16 05:54:17.282851
   - Amount: $2,032.67
   - Source Account: None
   - Destination Account: ACCT-15004839

5. Date: 2025-07-06 04:34:02.334791
   - Amount: $9,852.85
   - Source Account: ACCT-15004839
   - Destination Account: ACCT-32643038

6. Date: 2025-06-17 21:05:24.293256
   - Amount: $779.52
   - Source Account: ACCT-90471014
   - Destination Account: None

7. Date: 2025-06-10 06:00:19.339913
   - Amount: $6,857.06
   - Source Account: ACCT-90471014
   - Destination Account: ACCT-61089251

8. Date: 2025-06-07 15:42:11.320272
   - Amount: $445.47
   - Source Account: None
   - Destination Account: ACCT-90471014

This list includes all the last 10 transactions for Mark Lewis. Let me know if you need further details!

Considering the limit of the context window, we retrieve up to 500 rows of results to generate the answer.

**Table of Results:**

| transaction_date	         | transaction_amount      | source_account_number	 | destination_account_number |
|--------------------|------------------|------------|--------------|
| 2025-08-12 00:19:48.296814 | 597.27  | ACCT-15004839 | null         |
| 2025-07-20 15:41:54.322154 | 38.76 | null | ACCT-90471014         |
| 2025-07-18 12:03:15.286367 | 9133.7 | ACCT-90471014 | ACCT-72904922         |
| 2025-07-16 05:54:17.282851 | 2032.67 | null | ACCT-15004839         |
| 2025-07-06 04:34:02.334791 | 9852.85 | ACCT-15004839 | ACCT-32643038         |
| 2025-06-17 21:05:24.293256 | 779.52 | ACCT-90471014 | null         |
| 2025-06-10 06:00:19.339913 | 6857.06 | ACCT-90471014 | ACCT-61089251         |
| 2025-06-07 15:42:11.320272 | 445.47 | null | ACCT-90471014         |

**SQL Query:**
```sql
SELECT
  "transaction_date" AS "transaction_date",
  "amount" AS "transaction_amount",
  "source_account" AS "source_account_number",
  "destination_account" AS "destination_account_number"
FROM
  "public_transaction_history"
WHERE
  LOWER("customer_name") = LOWER('Mark Lewis')
ORDER BY
  "transaction_date" DESC
LIMIT
  10
```

You’re seeing Wren SQL. If you want to see this query on our own database, click “Show original SQL” to get the exact syntax.
<details>
<summary>Show original SQL</summary>

```sql
SELECT
  public_transaction_history.transaction_date,
  public_transaction_history.amount AS transaction_amount,
  public_transaction_history.source_account AS source_account_number,
  public_transaction_history.destination_account AS destination_account_number
FROM
  (
    SELECT
      public_transaction_history.amount,
      public_transaction_history.customer_name,
      public_transaction_history.destination_account,
      public_transaction_history.source_account,
      public_transaction_history.transaction_date
    FROM
      (
        SELECT
          __source.amount AS amount,
          __source.customer_name AS customer_name,
          __source.destination_account AS destination_account,
          __source.source_account AS source_account,
          __source.transaction_date AS transaction_date
        FROM
          banking_db."public".transaction_history AS __source
      ) AS public_transaction_history
  ) AS public_transaction_history
WHERE
  LOWER(public_transaction_history.customer_name) = LOWER('Mark Lewis')
ORDER BY
  public_transaction_history.transaction_date DESC NULLS LAST
LIMIT
  10
```
</details>

<br>

**Chart:**

<img width="700" height="320" alt="visualization (1)" src="https://github.com/user-attachments/assets/a37e5bba-89d0-4381-b4f9-4907f8d706c2" />


On point result with a great visualization.

*Depending on the model's whims, it may sometimes not display a chart at first, but if you run the same prompt again, it may display.

---

### Hard Prompt Example

**Prompt:**
For each month in the past year, show the number of new accounts opened, the number of loans issued, and the total value of deposits made.

**Why:**
- Requires multiple joins, grouping, and aggregation functions (COUNT DISTINCT, MIN, MAX, AVG).
- Showcases the model’s ability to generate advanced analytics queries.
- Represents a real-world business intelligence/reporting scenario.

**Answer:**
Here is the summary of the requested data for each month in the past year:

- August 2024: 7 new accounts, 1 loan issued, $0 in deposits.
- September 2024: 1 new account, 0 loans issued, $0 in deposits.
- October 2024: 8 new accounts, 3 loans issued, $0 in deposits.
- November 2024: 4 new accounts, 0 loans issued, $0 in deposits.
- December 2024: 3 new accounts, 0 loans issued, $0 in deposits.
- January 2025: 4 new accounts, 1 loan issued, $0 in deposits.
- February 2025: 4 new accounts, 1 loan issued, $0 in deposits.
- March 2025: 2 new accounts, 0 loans issued, $0 in deposits.
- April 2025: 2 new accounts, 0 loans issued, $0 in deposits.
- May 2025: 6 new accounts, 0 loans issued, $27,922.34 in deposits.
- June 2025: 8 new accounts, 2 loans issued, $79,048.12 in deposits.
- July 2025: 4 new accounts, 0 loans issued, $108,513.98 in deposits.

This provides a clear view of the activity for each month over the past year. Let me know if you need further details!

**Table of Results:**

| month   | new_accounts_count | loans_issued_count | total_deposits |
|-------------|-------------------|------------------------|----------------------|
| 2024-08-01 00:00:00.000000  | 7 | 1 | 0 |
| 2024-09-01 00:00:00.000000  | 1 | 0 | 0 |
| 2024-10-01 00:00:00.000000  | 8 | 3 | 0 |
| 2024-11-01 00:00:00.000000  | 4 | 0 | 0 |
| 2024-12-01 00:00:00.000000  | 3 | 0 | 0 |
| 2025-01-01 00:00:00.000000  | 4 | 1 | 0 |
| 2025-02-01 00:00:00.000000  | 4 | 1 | 0 |
| 2025-03-01 00:00:00.000000  | 2 | 0 | 0 |
| 2025-04-01 00:00:00.000000  | 2 | 0 | 0 |
| 2025-05-01 00:00:00.000000  | 6 | 0 | 27922.34 |
| 2025-06-01 00:00:00.000000  | 8 | 2 | 79048.12 |
| 2025-07-01 00:00:00.000000  | 4 | 0 | 108513.98 |

**SQL Query:**
```sql
WITH
  "new_accounts" AS (
    SELECT
      DATE_TRUNC ('MONTH', "open_date") AS "month",
      COUNT(*) AS "new_accounts_count"
    FROM
      "public_accounts"
    WHERE
      "open_date" >= CAST('2024-08-01' AS TIMESTAMP)
      AND "open_date" < CAST('2025-08-01' AS TIMESTAMP)
    GROUP BY
      DATE_TRUNC ('MONTH', "open_date")
  ),
  "loans_issued" AS (
    SELECT
      DATE_TRUNC ('MONTH', "issue_date") AS "month",
      COUNT(*) AS "loans_issued_count"
    FROM
      "public_loans"
    WHERE
      "issue_date" >= CAST('2024-08-01' AS TIMESTAMP)
      AND "issue_date" < CAST('2025-08-01' AS TIMESTAMP)
    GROUP BY
      DATE_TRUNC ('MONTH', "issue_date")
  ),
  "deposits_made" AS (
    SELECT
      DATE_TRUNC ('MONTH', "transaction_date") AS "month",
      SUM("amount") AS "total_deposits"
    FROM
      "public_transactions"
    WHERE
      "transaction_type" = 'deposit'
      AND "transaction_date" >= CAST('2024-08-01' AS TIMESTAMP)
      AND "transaction_date" < CAST('2025-08-01' AS TIMESTAMP)
    GROUP BY
      DATE_TRUNC ('MONTH', "transaction_date")
  )
SELECT
  COALESCE("na"."month", "li"."month", "dm"."month") AS "month",
  COALESCE("na"."new_accounts_count", 0) AS "new_accounts_count",
  COALESCE("li"."loans_issued_count", 0) AS "loans_issued_count",
  COALESCE("dm"."total_deposits", 0) AS "total_deposits"
FROM
  "new_accounts" AS "na"
  FULL OUTER JOIN "loans_issued" AS "li" ON "na"."month" = "li"."month"
  FULL OUTER JOIN "deposits_made" AS "dm" ON COALESCE("na"."month", "li"."month") = "dm"."month"
ORDER BY
  "month"
```

You’re seeing Wren SQL. If you want to see this query on our own database, click “Show original SQL” to get the exact syntax.
<details>
<summary>Show original SQL</summary>

```sql
SELECT
  COALESCE(na."month", li."month", dm."month") AS "month",
  COALESCE(na.new_accounts_count, 0) AS new_accounts_count,
  COALESCE(li.loans_issued_count, 0) AS loans_issued_count,
  COALESCE(dm.total_deposits, CAST(0 AS DECIMAL(38, 10))) AS total_deposits
FROM
  (
    SELECT
      DATE_TRUNC ('MONTH', public_accounts.open_date) AS "month",
      COUNT(1) AS new_accounts_count
    FROM
      (
        SELECT
          public_accounts.open_date
        FROM
          (
            SELECT
              __source.open_date AS open_date
            FROM
              banking_db."public".accounts AS __source
          ) AS public_accounts
      ) AS public_accounts
    WHERE
      public_accounts.open_date >= CAST('2024-08-01' AS TIMESTAMP)
      AND public_accounts.open_date < CAST('2025-08-01' AS TIMESTAMP)
    GROUP BY
      DATE_TRUNC ('MONTH', public_accounts.open_date)
  ) AS na
  FULL JOIN (
    SELECT
      DATE_TRUNC (
        'MONTH',
        CAST(public_loans.issue_date AS TIMESTAMP)
      ) AS "month",
      COUNT(1) AS loans_issued_count
    FROM
      (
        SELECT
          public_loans.issue_date
        FROM
          (
            SELECT
              __source.issue_date AS issue_date
            FROM
              banking_db."public".loans AS __source
          ) AS public_loans
      ) AS public_loans
    WHERE
      CAST(public_loans.issue_date AS TIMESTAMP) >= CAST('2024-08-01' AS TIMESTAMP)
      AND CAST(public_loans.issue_date AS TIMESTAMP) < CAST('2025-08-01' AS TIMESTAMP)
    GROUP BY
      DATE_TRUNC (
        'MONTH',
        CAST(public_loans.issue_date AS TIMESTAMP)
      )
  ) AS li ON na."month" = li."month"
  FULL JOIN (
    SELECT
      DATE_TRUNC ('MONTH', public_transactions.transaction_date) AS "month",
      SUM(public_transactions.amount) AS total_deposits
    FROM
      (
        SELECT
          public_transactions.amount,
          public_transactions.transaction_date,
          public_transactions.transaction_type
        FROM
          (
            SELECT
              __source.amount AS amount,
              __source.transaction_date AS transaction_date,
              __source.transaction_type AS transaction_type
            FROM
              banking_db."public".transactions AS __source
          ) AS public_transactions
      ) AS public_transactions
    WHERE
      public_transactions.transaction_type = 'deposit'
      AND public_transactions.transaction_date >= CAST('2024-08-01' AS TIMESTAMP)
      AND public_transactions.transaction_date < CAST('2025-08-01' AS TIMESTAMP)
    GROUP BY
      DATE_TRUNC ('MONTH', public_transactions.transaction_date)
  ) AS dm ON COALESCE(na."month", li."month") = dm."month"
ORDER BY
  "month" ASC
```
</details>

<br>

**Chart:**


<img width="700" height="320" alt="visualization (2)" src="https://github.com/user-attachments/assets/9a4a1b85-5321-40bd-8e68-c8530ab1f660" />


Correct. The answer, SQL, and result table match the intent.

---

Indicative execution times for each query:

<span style="font-size:11px">*These times are indicative. Meaning they are only for the time it took to execute THESE SPECIFIC three queries. From the moment you ask till the time it takes to show you the results table.</span>


| Prompt #	| Complexity	| Question | Answer preperation time |
|---|---------|----------------------------------------|-----------|
| 1	| Easy | "List all account types and..."  |  ~16s	|
| 2	| Medium	| "Show the last 10 transactions..."	| ~20s |
| 3	| Hard	| "For each month in the past year..." | ~29s |

---

*This guide is based on real setup, troubleshooting, and successful runs of WrenAI with Azure OpenAI and Docker.*

---

## Known Limitations & Gotchas

- **Context Window Limit:** Only up to 500 rows are shown in results due to the model's context window. For larger datasets, results are truncated.
- **Null Values:** Some fields at the results may appear as `null` or not provided if the data is missing or not joined correctly in the SQL. This is easily fixed by asking in the prompt to remove every row that contains a null value.
- **Model/Embedding Deployment:** You must deploy the exact model and embedding (with correct deployment names) in Azure OpenAI. Otherwise, you will get 'resource not found' errors.
- **Manual Database Connection:** If you want to connect to your own SQL Server, follow the official guide (see next section).
- **Feature Gaps:** Some advanced features (e.g., RAG, semantic search) require an embedding model and may not work if not configured.
- **Prompt Sensitivity:** The quality of results depends on prompt clarity. Vague prompts may yield incomplete or unexpected answers.
- **Upgrade Caveats:** When updating WrenAI, check for breaking changes in config or Docker images.

---

## Connecting to Your Own Database

To connect WrenAI to your own Data Source Type (PostgreSQL, MySQL, Microsoft SQL Server, etc...), follow the official guide:
- [WrenAI Data Source Connection Guide](https://docs.getwren.ai/oss/guide/connect/overview)

**Summary:**
- Configure your database connection in the WrenAI UI or in the config files as described in the guide.
- Make sure your database is accessible from the Docker network, if needed (e.g., use host.docker.internal for local connection).
- Test the connection in the UI before running prompts.

---



