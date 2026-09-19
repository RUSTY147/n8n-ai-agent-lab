# Setup Guide

This guide explains how to install, configure, and run the **n8n AI Agent Lab** locally on Windows.

The project uses:

- n8n
- Ollama
- Qwen 3 4B
- n8n AI Agent
- n8n Simple Memory
- n8n Data Tables
- External HTTP APIs

---

## Prerequisites

Before starting, install:

- Node.js
- npm
- n8n
- Ollama
- Git

The project is currently designed and tested for a local Windows environment.

---

# 1. Install Node.js

Download and install Node.js.

After installation, open PowerShell and verify:

```powershell
node --version
npm --version
```

Example:

```text
v24.x.x
11.x.x
```

---

# 2. Install Git

Install Git for Windows.

Verify the installation:

```powershell
git --version
```

Example:

```text
git version 2.x.x
```

---

# 3. Install n8n

Install n8n globally using npm:

```powershell
npm install -g n8n
```

Verify the installation:

```powershell
n8n --version
```

Start n8n:

```powershell
n8n
```

Open the n8n editor in your browser:

```text
http://localhost:5678
```

Keep the n8n process running while using the workflow.

---

# 4. Install Ollama

Install Ollama for Windows.

Verify the installation:

```powershell
ollama --version
```

---

# 5. Download the Qwen Model

This project uses:

```text
qwen3:4b
```

Download the model:

```powershell
ollama pull qwen3:4b
```

Verify that the model is installed:

```powershell
ollama list
```

You should see something similar to:

```text
NAME
qwen3:4b
```

---

# 6. Verify the Ollama API

Ollama provides a local HTTP API.

The workflow uses:

```text
http://127.0.0.1:11434
```

Verify that the API is responding:

```powershell
Invoke-RestMethod http://127.0.0.1:11434/api/tags
```

The response should contain information about the installed models.

For example:

```text
models
------
{@{name=qwen3:4b; ...}}
```

---

# 7. Clone the Repository

Clone the GitHub repository:

```powershell
git clone <YOUR_GITHUB_REPOSITORY_URL>
```

Move into the project:

```powershell
cd n8n-ai-agent-lab
```

The project structure should look like:

```text
n8n-ai-agent-lab/
│
├── .gitignore
├── README.md
│
├── data/
│   └── employees.json
│
├── docs/
│   ├── architecture.md
│   ├── setup.md
│   └── tools.md
│
└── workflows/
    └── local-ai-agent.json
```

---

# 8. Start n8n

From PowerShell:

```powershell
n8n
```

Open:

```text
http://localhost:5678
```

---

# 9. Import the Workflow

Open the n8n editor.

Import the workflow from:

```text
workflows/local-ai-agent.json
```

The workflow contains the AI Agent and its connected tools.

After importing, review the workflow before executing it.

---

# 10. Configure Ollama

Open the **Ollama Chat Model** node.

Configure the Ollama connection.

### Base URL

Use:

```text
http://127.0.0.1:11434
```

### Model

Use:

```text
qwen3:4b
```

The model must already be installed locally.

Verify with:

```powershell
ollama list
```

---

# 11. Configure the AI Agent

The AI Agent should use the local Ollama model.

The main components are:

```text
Chat Trigger
      ↓
AI Agent
      ↓
Ollama Chat Model
```

The AI Agent also connects to memory and tools.

---

# 12. Configure Simple Memory

The workflow uses **Simple Memory** to maintain conversation context.

The memory uses the chat session ID.

The configuration uses:

```text
Session ID:
{{ $json.sessionId }}
```

This allows the same chat session to remember previous messages.

Example:

```text
User:
My name is Ajay.

User:
What is my name?

Agent:
Your name is Ajay.
```

Memory is session-based and should not be treated as permanent user storage.

---

# 13. Configure the Employee Data Table

The workflow uses an n8n Data Table named:

```text
employees
```

The table contains:

| Name | Role | Location |
|---|---|---|
| Ajay | Data Engineer | Hyderabad |
| Rahul | Developer | Bangalore |
| Priya | Data Analyst | Chennai |
| Arjun | Software Engineer | Mumbai |

The repository also contains the example source data:

```text
data/employees.json
```

The JSON file contains:

```json
[
  {
    "name": "Ajay",
    "role": "Data Engineer",
    "location": "Hyderabad"
  },
  {
    "name": "Rahul",
    "role": "Developer",
    "location": "Bangalore"
  },
  {
    "name": "Priya",
    "role": "Data Analyst",
    "location": "Chennai"
  },
  {
    "name": "Arjun",
    "role": "Software Engineer",
    "location": "Mumbai"
  }
]
```

If the Data Table does not exist after importing the workflow, create an n8n Data Table named:

```text
employees
```

and add the example records.

---

# 14. Configure Employee Search

The workflow contains a tool named:

```text
Find Employee
```

It searches the `employees` Data Table using the employee name.

Example:

```text
Where does Ajay work?
```

The agent should use the employee tool and return information similar to:

```text
Ajay is a Data Engineer based in Hyderabad.
```

If an employee does not exist, the agent should report that the employee was not found rather than inventing information.

---

# 15. Configure Employee List

The workflow also contains:

```text
List All Employees
```

This tool returns all records from the employee Data Table.

Example:

```text
Who are all the employees?
```

Expected information:

```text
Ajay - Data Engineer - Hyderabad
Rahul - Developer - Bangalore
Priya - Data Analyst - Chennai
Arjun - Software Engineer - Mumbai
```

---

# 16. Calculator Tool

The AI Agent can use the Calculator tool for mathematical calculations.

Example:

```text
Calculate 347 * 829.
```

Expected result:

```text
287663
```

Another example:

```text
Calculate 125 * 48.
```

Expected result:

```text
6000
```

---

# 17. Geocoding Tool

The workflow uses the Open-Meteo geocoding API.

The endpoint is:

```text
https://geocoding-api.open-meteo.com/v1/search
```

The tool converts a city or location name into geographic coordinates.

Example:

```text
Mumbai
```

The tool returns latitude and longitude.

The geocoding tool should be used before the weather tool when coordinates are required.

---

# 18. Weather Tool

The workflow uses the Open-Meteo weather API.

The endpoint is:

```text
https://api.open-meteo.com/v1/forecast
```

The weather tool receives:

```text
latitude
longitude
```

and retrieves current weather information.

The workflow requests:

```text
temperature_2m
wind_speed_10m
```

The intended flow is:

```text
City
  ↓
Geocoding
  ↓
Latitude + Longitude
  ↓
Weather API
  ↓
Current Weather
```

---

# 19. Wikipedia Tool

The workflow uses the Wikipedia API to search for general factual information.

Example:

```text
Who is Alan Turing?
```

The agent can use the Wikipedia tool to search for relevant information.

The tool is intended for general topics such as:

- People
- Places
- Technologies
- Historical subjects
- General concepts

---

# 20. Test the Workflow

After configuring the workflow, test each capability individually.

---

## Test 1 — Memory

Send:

```text
My name is Ajay.
```

Then:

```text
What is my name?
```

The second question should use the conversation memory.

---

## Test 2 — Calculator

Send:

```text
Calculate 347 * 829.
```

The agent should use the Calculator tool.

---

## Test 3 — Employee Search

Send:

```text
Where does Ajay work?
```

The agent should use:

```text
Find Employee
```

and retrieve Ajay's information from the Data Table.

---

## Test 4 — Employee List

Send:

```text
Who are all the employees?
```

The agent should use:

```text
List All Employees
```

---

## Test 5 — Weather

Send:

```text
What is the weather in Mumbai?
```

The agent should:

```text
Identify Mumbai
      ↓
Geocode Mumbai
      ↓
Get coordinates
      ↓
Call Weather API
      ↓
Return current weather
```

---

## Test 6 — Multi-Tool Agent

Send:

```text
What is the weather in Arjun's city?
```

This is an important test because the agent needs multiple tools.

Expected flow:

```text
User Question
      ↓
AI Agent
      ↓
Find Employee
      ↓
Find Arjun
      ↓
Get Mumbai
      ↓
Geocoding
      ↓
Get Mumbai Coordinates
      ↓
Weather API
      ↓
Current Weather
      ↓
AI Agent
      ↓
Final Response
```

This demonstrates tool chaining.

---

## Test 7 — Wikipedia

Send:

```text
Who is Alan Turing?
```

The agent should use the Wikipedia tool to retrieve relevant information.

---

# 21. Troubleshooting

## Ollama Connection Error

Check whether Ollama is running:

```powershell
ollama list
```

Then test the API:

```powershell
Invoke-RestMethod http://127.0.0.1:11434/api/tags
```

If the API responds successfully, verify that the Ollama Chat Model node is configured with:

```text
http://127.0.0.1:11434
```

---

## `localhost` Connection Problem

If:

```text
http://localhost:11434
```

does not work correctly in the n8n environment, use:

```text
http://127.0.0.1:11434
```

instead.

---

## Qwen Model Not Found

Check:

```powershell
ollama list
```

If `qwen3:4b` is missing:

```powershell
ollama pull qwen3:4b
```

---

## n8n Does Not Start

Check the installed version:

```powershell
n8n --version
```

If n8n is not installed:

```powershell
npm install -g n8n
```

Then:

```powershell
n8n
```

---

## Employee Not Found

Verify that the Data Table:

```text
employees
```

exists in n8n.

Check that it contains the expected records.

---

## Weather Tool Fails

Check that:

1. The geocoding tool is working.
2. Latitude is being returned.
3. Longitude is being returned.
4. The weather API request receives valid coordinates.
5. The weather API node is connected to the AI Agent.

---

## Workflow Does Not Respond

Check the following:

1. n8n is running.
2. Ollama is running.
3. `qwen3:4b` is installed.
4. The Ollama Base URL is correct.
5. The workflow was imported successfully.
6. Required credentials are configured.
7. The employee Data Table exists.
8. The Chat Trigger is configured correctly.

---

# 22. Local Architecture

The complete local environment looks like:

```text
                         USER
                           │
                           ▼
                  ┌─────────────────┐
                  │   n8n Chat      │
                  │    Trigger      │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │    AI Agent     │
                  └────────┬────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
         ┌────────┐  ┌──────────┐  ┌───────────┐
         │ Ollama │  │  Memory  │  │   Tools   │
         │ Qwen   │  │          │  │           │
         │ 3:4b   │  │ Session  │  │ Calculator│
         └────────┘  └──────────┘  │ Employees │
                                    │ Weather   │
                                    │ Wikipedia │
                                    └─────┬─────┘
                                          │
                             ┌────────────┼────────────┐
                             │            │            │
                             ▼            ▼            ▼
                         Data Table   Open-Meteo   Wikipedia
                         employees       APIs         API
```

---

# 23. Project Files

Important project files:

```text
README.md
```

Main project documentation.

```text
workflows/local-ai-agent.json
```

Exported n8n workflow.

```text
data/employees.json
```

Example employee dataset.

```text
docs/architecture.md
```

Technical architecture documentation.

```text
docs/setup.md
```

Installation and setup instructions.

```text
docs/tools.md
```

Documentation for the AI Agent tools.

```text
.gitignore
```

Prevents local files, secrets, logs, and environment files from being committed.

---

# 24. Security Notes

Do not commit:

- API keys
- Passwords
- Access tokens
- Private credentials
- `.env` files
- Personal data
- Production database credentials

The exported workflow should be reviewed before publishing it to GitHub.

Credential references may appear in an exported n8n workflow, but actual secrets should never be committed.

---

# 25. Running the Project

The normal startup sequence is:

### Terminal 1

Start Ollama if required by your installation:

```powershell
ollama serve
```

### Terminal 2

Start n8n:

```powershell
n8n
```

Then open:

```text
http://localhost:5678
```

Open the imported workflow and use the chat interface.

---

# 26. Quick Start

For an already configured machine:

```powershell
ollama list
```

Confirm:

```text
qwen3:4b
```

Then:

```powershell
n8n
```

Open:

```text
http://localhost:5678
```

Import:

```text
workflows/local-ai-agent.json
```

Configure the Ollama connection if required.

Then test:

```text
What is the weather in Arjun's city?
```

---

# 27. Example End-to-End Interaction

User:

```text
What is the weather in Arjun's city?
```

The AI Agent determines that it needs employee information first.

```text
AI Agent
   ↓
Find Employee
   ↓
Arjun
   ↓
Mumbai
```

The agent then resolves the location.

```text
Mumbai
   ↓
Geocoding API
   ↓
Latitude + Longitude
```

The coordinates are passed to the weather API.

```text
Latitude + Longitude
   ↓
Weather API
   ↓
Current Weather
```

Finally, the AI Agent converts the tool results into a natural-language response.

```text
Tool Results
     ↓
AI Agent
     ↓
User-Friendly Response
```

---

# 28. Learning Objectives

This project demonstrates the following n8n concepts:

- Building an AI Agent
- Connecting a local LLM
- Using Ollama with n8n
- Configuring AI Agent tools
- Tool calling
- Multi-tool execution
- Tool chaining
- Conversation memory
- Data Tables
- HTTP Request tools
- External APIs
- Natural-language interfaces
- Local AI development
- Workflow debugging
- n8n workflow export and version control

---

# 29. Future Improvements

Possible future improvements include:

- Add more employee operations
- Add database integration
- Add REST API tools
- Add authentication
- Add structured logging
- Add automated testing
- Add error-handling workflows
- Add monitoring
- Add additional local LLMs
- Compare different local models
- Add a deterministic workflow alongside the AI Agent
- Add a web-based frontend
- Containerize the project with Docker

---

## Conclusion

The **n8n AI Agent Lab** provides a local environment for experimenting with AI agents, tool calling, memory, APIs, and workflow automation.

The project combines a local Ollama model with n8n's workflow automation capabilities, allowing natural-language requests to be translated into tool-based actions.