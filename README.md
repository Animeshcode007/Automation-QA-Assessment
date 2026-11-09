# GitHub Morning Brief – n8n Automation

**Developer:** Animesh Khare  
**Project:** Task 2 – Workflow Automation using n8n  
**Technology:** n8n, JavaScript, GitHub REST API, Telegram Bot API

## 1. Project Overview

GitHub Morning Brief is an automated workflow built using n8n that fetches popular React repositories from GitHub, identifies the top 5 repositories based on star count, retrieves detailed information about the most popular repository, and sends a formatted digest to Telegram.

The workflow also includes conditional routing based on repository popularity, hourly scheduling, and a separate error-handling workflow to ensure failures are reported through Telegram.

## 2. Features

- **Automated Scheduling:** Executes automatically every hour using the Schedule Trigger.
- **Manual Execution:** Supports manual workflow execution for testing.
- **GitHub API Integration:** Fetches the 20 most-starred repositories matching the `topic:react` query.
- **Data Transformation:** Uses a JavaScript Code node to sort repositories by stars and select the top 5.
- **Repository Enrichment:** Makes a second GitHub API request to retrieve detailed information about the highest-ranked repository.
- **Conditional Routing:** Checks whether the featured repository has more than 50,000 stars.
- **Telegram Notifications:** Sends different messages depending on the popularity condition.
- **Error Handling:** Uses a separate Error Trigger workflow to send failure notifications to Telegram.

## 3. Workflow Architecture

The main workflow follows this sequence:

```text
Schedule Trigger / Manual Trigger
              |
              v
       HTTP Request
       (GitHub Search API)
              |
              v
         Select Top 5
         (JavaScript)
              |
              v
      Get Top Repo Details
       (GitHub API)
              |
              v
              IF
       (Stars > 50,000?)
          /       \
        Yes        No
         |          |
         v          v
    High Popularity  Normal Popularity
       Report          Report
         |               |
         v               v
       Telegram        Telegram
```

**Separate Error Handling Workflow:**

```text
       Error Trigger
             |
             v
     Telegram Notification
```

## 4. APIs Used

### First HTTP Request – GitHub Repository Search

**Endpoint:**

`https://api.github.com/search/repositories`

**Query Parameters:**

| Parameter | Value |
|---|---|
| q | topic:react |
| sort | stars |
| order | desc |
| per_page | 20 |

This request retrieves up to 20 React-related repositories sorted by their star count in descending order.

### Second HTTP Request – Repository Details

**Endpoint:**

`https://api.github.com/repos/{owner}/{repo}`

The workflow dynamically constructs this URL using the full name of the highest-ranked repository.

It retrieves additional information, including:
- Repository name
- Total stars
- Fork count
- Open issues
- Primary programming language
- Repository URL

## 5. Data Transformation

The JavaScript Code node performs the following operations:

1. Extracts repository data from the GitHub API response.
2. Sorts repositories by their star count in descending order.
3. Selects the top 5 repositories.
4. Reshapes the response into a simplified JSON structure.
5. Stores the featured repository's full name for the second API request.

The output includes:
- `fetchedCount`
- `selectedCount`
- `top5`
- `topRepoFullName`
- `popularThreshold`

## 6. Conditional Logic

The IF node checks the star count of the featured repository.

**Condition:**

`stargazers_count > 50000`

| Condition | Action |
|---|---|
| True | Sends a High Popularity Alert with the top 5 repositories and detailed featured repository information. |
| False | Sends a Normal Popularity Report to Telegram. |

## 7. Telegram Notifications

The workflow sends the final digest through a Telegram bot.

### High Popularity Alert

Includes:
- Top 5 React repositories
- Star counts
- Programming languages
- Repository URLs
- Detailed information about the featured repository

### Normal Popularity Report

Includes:
- Top 5 React repositories
- Star counts
- Programming languages
- Repository URLs
- Featured repository information

## 8. Error Handling

A separate workflow named **GitHub Brief Error Handler** is configured.

It consists of:

1. **Error Trigger:** Automatically receives error information from the main workflow.
2. **Telegram Notification:** Sends an error report containing:
   - Workflow name
   - Failed node
   - Error message
   - Execution ID

This ensures that workflow failures are not silently ignored.

## 9. Credentials and Security

- Telegram credentials are configured using n8n's Credentials store.
- Bot tokens are not manually hard-coded into workflow nodes.
- The Telegram credential is referenced by the Telegram nodes.
- The GitHub public API is accessed without authentication.

**Note:** The Telegram Chat ID is configured in the notification nodes. It is an identifier, not a bot authentication secret.

## 10. Setup and Execution

### Prerequisites

- An n8n instance (cloud or self-hosted).
- A Telegram bot created using BotFather.
- A Telegram account and chat ID.
- A configured Telegram credential in n8n.

### Installation Steps

1. Open your n8n instance.
2. Import `Task2_Workflow_Animesh_Khare.json`.
3. Import `GitHub_Brief_Error_Handler.json` as a separate workflow.
4. Configure or verify the Telegram credentials.
5. Verify the Telegram Chat ID in the notification nodes.
6. Ensure the main workflow references the correct Error Handler workflow.
7. Save and publish the main workflow.
8. Execute manually to test the complete workflow.
9. Activate the workflow to enable hourly scheduling.

## 11. Testing and Validation

The following functionality has been tested:

- Successful GitHub API requests.
- Successful repository sorting and top 5 selection.
- Successful retrieval of featured repository details.
- Successful execution of the IF node.
- Successful Telegram message delivery.
- Successful execution of the high-popularity branch.
- Successful execution of the normal-popularity branch.
- Hourly schedule configuration.
- Error Handler configuration and Telegram error notification.

## 12. Project Files

| File | Description |
|---|---|
| `Task2_Workflow_Animesh_Khare.json` | Main n8n workflow |
| `GitHub_Brief_Error_Handler.json` | Separate error-handling workflow |
| `README.md` | Project documentation |

## 13. Conclusion

GitHub Morning Brief demonstrates how n8n can be used to build an automated API-driven workflow involving data retrieval, JavaScript transformation, API enrichment, conditional branching, scheduled execution, and Telegram notifications.

The project provides a practical example of workflow automation with error reporting and credential management using n8n.

---

**Developed by Animesh Khare**  
*Task 2 – n8n Workflow Automation*
