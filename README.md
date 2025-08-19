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
| Qdrant         | (via Docker)                 |

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
- **Qdrant** and **PostgreSQL** are started automatically via Docker Compose.
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

The three prompts below together will best showcase the range and power of the text-to-SQL solution.

Summary Table:

| Prompt #	| Complexity	| SQL Features Demonstrated |	Why Show It? |
|---|---------|----------------------------------------|-------------------------------------------------|
| 1	| Easy | GROUP BY, HAVING	| Basic aggregation/data quality |
| 2	| Medium	| JOIN, WHERE, multi-column SELECT	| Multi-table, real reporting |
| 3	| Hard	| Multi-join, GROUP BY, aggregation	| Advanced analytics, BI scenario |

### Easy Prompt Example

**Prompt:**
Are there any duplicate entries in the 'tx_id' column of the 'dbo_deposit_extra' table?

**Why:**
- Demonstrates simple aggregation and grouping.
- Easy to verify correctness.
- Shows the model can answer basic data quality questions.

**Answer:**
Yes, there are duplicate entries in the 'tx_id' column of the 'dbo_deposit_extra' table. Here are a few examples of 'tx_id' values with their respective duplicate counts:

- M00000048 appears 4 times.
- M00003024 appears 15 times.
- M00003327 appears 7 times.
- There are more duplicate entries in the data, but these are just a few examples.

Results: Considering the limit of the context window, we retrieve up to 500 rows of results to generate the answer.

**Table of Results:**

| tx_id      | duplicate_count |
|------------|----------------|
| M00000048  | 4              |
| M00003024  | 15             |
| M00003327  | 7              |
| M00012325  | 10             |
| M00035765  | 6              |
| M00112325  | 3              |
| M00992325  | 4              |
| M07597931  | 2              |
| test00001  | 19             |

**SQL Query:**
```sql
SELECT
  dbo_deposit_extra.tx_id,
  COUNT_BIG(1) AS duplicate_count
FROM
  (
    SELECT
      dbo_deposit_extra.tx_id AS tx_id
    FROM
      (
        SELECT
          __source.tx_id AS tx_id,
          __source.tx_id_part_srl_num AS tx_id_part_srl_num
        FROM
          cbstrnext_dev.dbo.deposit_extra AS __source
      ) AS dbo_deposit_extra
  ) AS dbo_deposit_extra
GROUP BY
  dbo_deposit_extra.tx_id
HAVING
  COUNT_BIG(1) > 1
```

**Chart:**

![alt text](images\prompt_1.png)

The query is Correct. The answer and SQL match the intent, and the chart visualizes the duplicate counts.

---

### Medium Prompt Example

**Prompt:**
Generate a report of deposit extra batch events linked to specific event types and sources, including related account info and timestamps.

**Why:**
- Involves joining tables, filtering, and selecting multiple columns.
- Demonstrates the model’s ability to handle multi-table queries and more complex reporting.

**Answer:**
Here is the report of deposit extra batch events linked to specific event types and sources, including related account information and timestamps:

Event Type: core.NotifyTranPost  
Event Source: core.ofti_postman  
Account ID: 9300236488  
Account Name: Not available  
Account Balance: Not available  
Event Timestamp: 2025-07-04 10:27:50

Event Type: core.NotifyTranPost  
Event Source: core.ofti_postman  
Account ID: 1440153303  
Account Name: Not available  
Account Balance: Not available  
Event Timestamp: 2025-06-11 10:58:54

Event Type: core.NotifyTranPost  
Event Source: core.ofti_postman  
Account ID: 1455154303  
Account Name: Not available  
Account Balance: Not available  
Event Timestamp: 2025-06-11 10:58:54

Event Type: core.NotifyTranPost  
Event Source: core.ofti_postman  
Account ID: 8855154303  
Account Name: Not available  
Account Balance: Not available  
Event Timestamp: 2025-06-11 10:58:54

Event Type: core.NotifyTranPost  
Event Source: core.ofti_postman  
Account ID: 8899900009  
Account Name: Not available  
Account Balance: Not available  
Event Timestamp: 2025-06-10 10:30:06

Note: There are more results, but only the top few are shown here.

Results: Considering the limit of the context window, we retrieve up to 500 rows of results to generate the answer.

**Table of Results:**

| event_type         | event_source      | account_id | account_name | account_balance | event_timestamp         |
|--------------------|------------------|------------|--------------|-----------------|------------------------|
| core.NotifyTranPost| core.ofti_postman| 9300236488 | null         | null            | 2025-07-04 10:27:50.000000 |
| core.NotifyTranPost| core.ofti_postman| 1440153303 | null         | null            | 2025-06-11 10:58:54.000000 |
| core.NotifyTranPost| core.ofti_postman| 1455154303 | null         | null            | 2025-06-11 10:58:54.000000 |
| core.NotifyTranPost| core.ofti_postman| 8855154303 | null         | null            | 2025-06-11 10:58:54.000000 |
| core.NotifyTranPost| core.ofti_postman| 8899900009 | null         | null            | 2025-06-10 10:30:06.000000 |
| core.NotifyTranPost| core.ofti_postman| 9999900009 | null         | null            | 2025-06-10 10:30:06.000000 |
| core.NotifyTranPost| core.ofti_postman| 9999900015 | null         | null            | 2025-06-10 10:30:06.000000 |
| core.NotifyTranPost| core.ofti_postman| 9999900025 | null         | null            | 2025-06-10 10:30:06.000000 |

**SQL Query:**
```sql
SELECT
  src.event_type,
  src.event_source,
  src.acnt_id_othr_id AS account_id,
  leg.acnt_nm AS account_name,
  leg.acnt_balance AS account_balance,
  src.event_ts AS event_timestamp
FROM
  (
    SELECT
      dbo_deposit_extra_batch_src.acnt_id_othr_id AS acnt_id_othr_id,
      dbo_deposit_extra_batch_src.event_source AS event_source,
      dbo_deposit_extra_batch_src.event_ts AS event_ts,
      dbo_deposit_extra_batch_src.event_type AS event_type
    FROM
      (
        SELECT
          __source.acnt_id_othr_id AS acnt_id_othr_id,
          __source.event_source AS event_source,
          __source.event_ts AS event_ts,
          __source.event_type AS event_type
        FROM
          cbstrnext_dev.dbo.deposit_extra_batch_src AS __source
      ) AS dbo_deposit_extra_batch_src
  ) AS src
  LEFT JOIN (
    SELECT
      dbo_deposit_extra_leg.acnt_balance AS acnt_balance,
      dbo_deposit_extra_leg.acnt_id_othr_id AS acnt_id_othr_id,
      dbo_deposit_extra_leg.acnt_nm AS acnt_nm
    FROM
      (
        SELECT
          __source.acnt_balance AS acnt_balance,
          __source.acnt_id_othr_id AS acnt_id_othr_id,
          __source.acnt_nm AS acnt_nm,
          __source.tx_id_part_srl_num AS tx_id_part_srl_num
        FROM
          cbstrnext_dev.dbo.deposit_extra_leg AS __source
      ) AS dbo_deposit_extra_leg
  ) AS leg ON src.acnt_id_othr_id = leg.acnt_id_othr_id
WHERE
  NOT src.event_type IS NULL
  AND NOT src.event_source IS NULL
ORDER BY
  src.event_ts DESC
```

**Chart:**
_Not available for this example._ The provided data contains event types, sources, and timestamps, but lacks meaningful numerical or categorical diversity for visualization. Thus, no chart is generated.

Partial. The answer and SQL are correct for the available data, but account name and balance are missing (null) for these events. One way would be to ask specifically in the prompt to remove the null values.

---

### Hard Prompt Example

**Prompt:**
For each event type, show the number of unique accounts involved, the earliest and latest event timestamp, and the average account balance at the time of the event.

**Why:**
- Requires multiple joins, grouping, and aggregation functions (COUNT DISTINCT, MIN, MAX, AVG).
- Showcases the model’s ability to generate advanced analytics queries.
- Represents a real-world business intelligence/reporting scenario.

**Answer:**
Here is the summary for each event type:

Unknown Event Type:

- Unique Accounts: 24
- Earliest Event: December 1, 2015, 12:40:56
- Latest Event: April 21, 2025, 02:02:45

core.accountlienadd:

- Unique Accounts: 25
- Earliest Event: April 15, 2025, 11:10:16
- Latest Event: July 15, 2025, 13:04:40

core.accountlienlift:

- Unique Accounts: 14
- Earliest Event: April 15, 2025, 11:13:16
- Latest Event: July 11, 2025, 18:08:20

core.DepositMQ:

- Unique Accounts: 6
- Earliest Event: July 21, 2025, 09:46:17
- Latest Event: July 21, 2025, 11:29:37

core.NotifyTranPost:

- Unique Accounts: 55
- Earliest Event: March 21, 2025, 02:02:45
- Latest Event: August 6, 2025, 16:47:32

The average account balance at the time of the events is not included in the data provided, so it cannot be calculated. Let me know if you need further assistance!

**Table of Results:**

| event_type   | unique_accounts | earliest_event_ts | latest_event_ts |
|-------------|-------------------|------------------------|----------------------|
| null  | 24 | 2015-12-01 12:40:56.789789 | 2025-04-21 02:02:45.000000 |
| core.accountlienadd  | 25 | 2025-04-15 11:10:16.000000 | 2025-07-15 13:04:40.000000 |
| core.accountlienlift  | 14 | 2025-04-15 11:13:16.000000 | 2025-07-11 18:08:20.000000 |
| core.DepositMQ  | 6 | 2025-07-21 09:46:17.523375 | 2025-07-21 11:29:37.045065 |
| core.NotifyTranPost  | 55 | 2025-03-21 02:02:45.000456 | 2025-08-06 16:47:32.000000 |


**SQL Query:**
```sql
SELECT
  [CombinedEvents].event_type,
  COUNT_BIG(DISTINCT [CombinedEvents].acnt_id_othr_id) AS unique_accounts,
  MIN([CombinedEvents].event_ts) AS earliest_event_ts,
  MAX([CombinedEvents].event_ts) AS latest_event_ts
FROM
  (
    SELECT
      dbo_deposit_extra.event_type,
      dbo_deposit_extra.event_ts,
      dbo_deposit_extra.acnt_id_othr_id
    FROM
      (
        SELECT
          dbo_deposit_extra.acnt_id_othr_id AS acnt_id_othr_id,
          dbo_deposit_extra.event_ts AS event_ts,
          dbo_deposit_extra.event_type AS event_type
        FROM
          (
            SELECT
              __source.acnt_id_othr_id AS acnt_id_othr_id,
              __source.event_ts AS event_ts,
              __source.event_type AS event_type,
              __source.tx_id_part_srl_num AS tx_id_part_srl_num
            FROM
              cbstrnext_dev.dbo.deposit_extra AS __source
          ) AS dbo_deposit_extra
      ) AS dbo_deposit_extra
    UNION ALL
    SELECT
      dbo_lien.event_type,
      dbo_lien.event_ts,
      dbo_lien.acnt_id_othr_id
    FROM
      (
        SELECT
          dbo_lien.acnt_id_othr_id AS acnt_id_othr_id,
          dbo_lien.event_ts AS event_ts,
          dbo_lien.event_type AS event_type
        FROM
          (
            SELECT
              __source.acnt_id_othr_id AS acnt_id_othr_id,
              __source.event_ts AS event_ts,
              __source.event_type AS event_type,
              __source.ts_count AS ts_count
            FROM
              cbstrnext_dev.dbo.lien AS __source
          ) AS dbo_lien
      ) AS dbo_lien
    UNION ALL
    SELECT
      dbo_deposit_all_v.event_type,
      dbo_deposit_all_v.event_ts,
      dbo_deposit_all_v.acnt_id_othr_id
    FROM
      (
        SELECT
          dbo_deposit_all_v.acnt_id_othr_id AS acnt_id_othr_id,
          dbo_deposit_all_v.event_ts AS event_ts,
          dbo_deposit_all_v.event_type AS event_type
        FROM
          (
            SELECT
              __source.acnt_id_othr_id AS acnt_id_othr_id,
              __source.event_ts AS event_ts,
              __source.event_type AS event_type
            FROM
              cbstrnext_dev.dbo.deposit_all_v AS __source
          ) AS dbo_deposit_all_v
      ) AS dbo_deposit_all_v
  ) AS [CombinedEvents]
GROUP BY
  [CombinedEvents].event_type
```

**Chart:**

![alt text](images\prompt_3.png)

Correct. The answer, SQL, and result table match the intent. Expiry date is not available for these transactions.

---

*This guide is based on real setup, troubleshooting, and successful runs of WrenAI with Azure OpenAI and Docker.*

---

## Known Limitations & Gotchas

- **Context Window Limit:** Only up to 500 rows are shown in results due to the model's context window. For larger datasets, results are truncated.
- **Null Values:** Some fields (e.g., account name, expiry date) may appear as `null` or not provided if the data is missing or not joined correctly in the SQL. This is easily fixed by asking in the prompt to remove every row that contains a null value.
- **Model/Embedding Deployment:** You must deploy the exact model and embedding (with correct deployment names) in Azure OpenAI. Otherwise, you will get 'resource not found' errors.
- **Manual Database Connection:** If you want to connect to your own SQL Server, follow the official guide (see next section).
- **Feature Gaps:** Some advanced features (e.g., RAG, semantic search) require an embedding model and may not work if not configured.
- **Prompt Sensitivity:** The quality of results depends on prompt clarity. Vague prompts may yield incomplete or unexpected answers.
- **Upgrade Caveats:** When updating WrenAI, check for breaking changes in config or Docker images.

---

## Connecting to Your Own Database

To connect WrenAI to your own SQL Server database, follow the official guide:
- [WrenAI SQL Server Connection Guide](https://docs.getwren.ai/oss/guide/connect/sqlserver)

**Summary:**
- Configure your database connection in the WrenAI UI or in the config files as described in the guide.
- Make sure your database is accessible from the Docker network, if needed (e.g., use host.docker.internal for local SQL Server on Windows).
- Test the connection in the UI before running prompts.

---

