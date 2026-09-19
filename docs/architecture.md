# Architecture

## Overview

The **n8n AI Agent Lab** is a local AI agent project built using **n8n** and **Ollama**.

The system allows a user to interact with an AI agent using natural language. The agent can understand the request, decide which tool is required, execute the tool, process the result, and return a response.

The project demonstrates:

- n8n AI Agents
- Local LLM execution
- Ollama
- Qwen 3 4B
- Tool calling
- Multi-tool execution
- Conversation memory
- n8n Data Tables
- HTTP APIs
- API chaining
- Natural-language interaction
- Local AI automation

---

# 1. High-Level Architecture

The overall architecture is:

```text
                         ┌──────────────────────┐
                         │        USER          │
                         │                      │
                         │ Natural Language     │
                         │      Request         │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │     n8n Chat         │
                         │       Trigger       │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │      AI Agent        │
                         │                      │
                         │  Request Analysis    │
                         │  Tool Selection      │
                         │  Tool Execution      │
                         │  Response Generation │
                         └──────────┬───────────┘
                                    │
                  ┌─────────────────┼──────────────────┐
                  │                 │                  │
                  ▼                 ▼                  ▼
          ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
          │ Ollama       │  │   Memory     │  │    Tools     │
          │              │  │              │  │              │
          │ Qwen 3 4B    │  │ Session      │  │ Calculator   │
          │ Local LLM    │  │ Context      │  │ Employees    │
          └──────────────┘  └──────────────┘  │ Weather      │
                                              │ Geocoding    │
                                              │ Wikipedia    │
                                              └──────┬───────┘
                                                     │
                                  ┌──────────────────┼──────────────────┐
                                  │                  │                  │
                                  ▼                  ▼                  ▼
                           ┌─────────────┐   ┌──────────────┐   ┌────────────┐
                           │ n8n Data    │   │ Open-Meteo   │   │ Wikipedia  │
                           │ Table       │   │ APIs         │   │ API        │
                           │             │   │              │   │            │
                           │ employees   │   │ Geocoding    │   │ Search     │
                           │             │   │ Weather      │   │            │
                           └─────────────┘   └──────────────┘   └────────────┘
```

---

# 2. Core Components

## 2.1 User

The user communicates with the system using natural language.

Examples:

```text
Where does Ajay work?
```

```text
Calculate 347 * 829.
```

```text
What is the weather in Mumbai?
```

```text
What is the weather in Arjun's city?
```

The user does not need to know which tool is required.

The AI Agent determines the required operation.

---

# 3. Chat Trigger

The **Chat Trigger** is the entry point of the workflow.

The basic flow is:

```text
User
  ↓
Chat Trigger
  ↓
AI Agent
```

The Chat Trigger receives the user's message and passes the request into the AI Agent.

The workflow also receives a session identifier that can be used by the memory component.

Example input:

```text
What is the weather in Arjun's city?
```

The request is passed to the AI Agent for processing.

---

# 4. AI Agent

The **AI Agent** is the central decision-making component of the workflow.

Its responsibilities include:

1. Understand the user's request.
2. Determine whether a tool is required.
3. Select the appropriate tool.
4. Determine the required tool parameters.
5. Execute the tool.
6. Process the returned information.
7. Generate the final response.

The AI Agent is connected to the local Ollama model.

High-level flow:

```text
User Request
     ↓
AI Agent
     ↓
Understand Request
     ↓
Select Tool
     ↓
Execute Tool
     ↓
Process Result
     ↓
Generate Response
```

---

# 5. Local LLM — Ollama

Ollama provides the local language model used by the AI Agent.

Current model:

```text
qwen3:4b
```

The model runs locally instead of requiring a hosted LLM API.

The n8n workflow connects to:

```text
http://127.0.0.1:11434
```

The local architecture is:

```text
n8n
 │
 ▼
Ollama
 │
 ▼
Qwen 3 4B
```

This allows the project to experiment with AI agents while keeping the primary language-model execution on the local machine.

---

# 6. Conversation Memory

The workflow uses **Simple Memory** to maintain context during a conversation.

The memory uses the chat session ID.

Conceptually:

```text
Chat Session
     │
     ├── Message 1
     ├── Message 2
     ├── Message 3
     └── Message 4
             │
             ▼
        Simple Memory
```

Example:

```text
User:
My name is Ajay.
```

Later:

```text
User:
What is my name?
```

The AI Agent can use the conversation memory to answer:

```text
Your name is Ajay.
```

Memory is associated with the current conversation session.

---

# 7. Tool Architecture

The AI Agent can access multiple tools.

Current tools include:

```text
                    AI Agent
                        │
       ┌────────────────┼─────────────────┐
       │                │                 │
       ▼                ▼                 ▼
 Calculator       Employee Tools     External APIs
                       │                 │
              ┌────────┴────────┐   ┌────┴─────────┐
              │                 │   │              │
              ▼                 ▼   ▼              ▼
        Find Employee    List All   Geocoding    Wikipedia
                         Employees     │
                                      ▼
                                    Weather
```

The agent chooses the required tool based on the user's request.

---

# 8. Calculator Tool

The Calculator tool handles mathematical operations.

Example:

```text
Calculate 347 * 829.
```

The workflow can delegate the calculation to the Calculator tool.

Example:

```text
User Request
     ↓
AI Agent
     ↓
Calculator
     ↓
287663
     ↓
AI Agent
     ↓
Final Response
```

Another example:

```text
Calculate 125 * 48.
```

Result:

```text
6000
```

---

# 9. Employee Data Architecture

The project contains an example employee dataset.

The source data is stored in:

```text
data/employees.json
```

Example:

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

The n8n workflow uses an n8n Data Table named:

```text
employees
```

The conceptual architecture is:

```text
employees.json
      │
      ▼
n8n Data Table
      │
      ▼
employees
      │
      ├───────────────┐
      ▼               ▼
Find Employee    List All Employees
```

---

# 10. Find Employee Tool

The **Find Employee** tool searches the employee Data Table using an employee name.

Example:

```text
Where does Ajay work?
```

The flow is:

```text
User
  ↓
AI Agent
  ↓
Find Employee
  ↓
Search employees table
  ↓
Ajay
  ↓
Data Engineer
  ↓
Hyderabad
  ↓
AI Agent
  ↓
Final Response
```

The tool is designed to retrieve actual records from the Data Table rather than allowing the AI Agent to invent employee information.

If an employee is not found, the agent should communicate that no matching employee was found.

---

# 11. List All Employees Tool

The **List All Employees** tool retrieves all employee records.

Example:

```text
Who are all the employees?
```

The flow is:

```text
User
  ↓
AI Agent
  ↓
List All Employees
  ↓
employees Data Table
  ↓
All Employee Records
  ↓
AI Agent
  ↓
Final Response
```

---

# 12. Geocoding Tool

The Geocoding tool converts a city or location name into geographic coordinates.

The workflow uses the Open-Meteo Geocoding API.

Endpoint:

```text
https://geocoding-api.open-meteo.com/v1/search
```

Example:

```text
Mumbai
```

The geocoding service returns information including:

```text
Latitude
Longitude
```

The architecture is:

```text
City Name
    ↓
Geocoding API
    ↓
Latitude + Longitude
```

The coordinates can then be passed to the weather tool.

---

# 13. Weather Tool

The Weather tool uses the Open-Meteo Forecast API.

Endpoint:

```text
https://api.open-meteo.com/v1/forecast
```

The weather tool requires:

```text
Latitude
Longitude
```

The workflow requests current weather information such as:

```text
temperature_2m
wind_speed_10m
```

The basic flow is:

```text
Location
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

# 14. Wikipedia Tool

The workflow also contains a Wikipedia HTTP tool.

The tool is intended for general factual searches.

Example:

```text
Who is Alan Turing?
```

The architecture is:

```text
User Question
      ↓
AI Agent
      ↓
Wikipedia Tool
      ↓
Wikipedia API
      ↓
Search Results
      ↓
AI Agent
      ↓
Final Response
```

---

# 15. Multi-Tool Execution

One of the main capabilities demonstrated by this project is the ability to use multiple tools to answer a single question.

Example:

```text
What is the weather in Arjun's city?
```

The question requires more than one operation.

The agent needs to:

```text
1. Find Arjun.
2. Determine Arjun's city.
3. Convert the city into coordinates.
4. Retrieve the weather.
5. Generate the final response.
```

The execution flow is:

```text
                        User
                          │
                          ▼
                  "Weather in
                   Arjun's city?"
                          │
                          ▼
                    ┌───────────┐
                    │ AI Agent  │
                    └─────┬─────┘
                          │
                          ▼
                  ┌───────────────┐
                  │ Find Employee │
                  └───────┬───────┘
                          │
                          ▼
                       Arjun
                          │
                          ▼
                       Mumbai
                          │
                          ▼
                  ┌───────────────┐
                  │   Geocoding   │
                  └───────┬───────┘
                          │
                          ▼
                Latitude + Longitude
                          │
                          ▼
                  ┌───────────────┐
                  │    Weather    │
                  └───────┬───────┘
                          │
                          ▼
                    Weather Data
                          │
                          ▼
                    ┌───────────┐
                    │ AI Agent  │
                    └─────┬─────┘
                          │
                          ▼
                    Final Answer
```

This demonstrates **tool chaining**.

---

# 16. Tool Chaining

Some tools depend on the result of another tool.

The weather example is the clearest case.

The weather API needs coordinates.

The user only provides a city.

Therefore:

```text
City
 ↓
Geocoding
 ↓
Coordinates
 ↓
Weather
```

For an employee-based weather request:

```text
Employee Name
 ↓
Find Employee
 ↓
Employee Location
 ↓
Geocoding
 ↓
Coordinates
 ↓
Weather
```

The AI Agent coordinates these operations.

---

# 17. End-to-End Data Flow

The complete request lifecycle is:

```text
┌───────────────────────┐
│        USER           │
│ Natural Language      │
│ Request               │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│     Chat Trigger      │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│       AI Agent        │
└───────────┬───────────┘
            │
            ├──────────────► Ollama / Qwen 3 4B
            │
            ├──────────────► Simple Memory
            │
            └──────────────► Tools
                               │
                ┌──────────────┼───────────────┐
                │              │               │
                ▼              ▼               ▼
           Calculator     Employee       External APIs
                              Tools              │
                                │                │
                                ▼                ▼
                          Data Table      Open-Meteo /
                                         Wikipedia
                                │
                                └───────┬────────┘
                                        │
                                        ▼
                                  Tool Results
                                        │
                                        ▼
                                   AI Agent
                                        │
                                        ▼
                                  Final Answer
```

---

# 18. Separation of Responsibilities

The architecture separates different responsibilities.

| Component | Responsibility |
|---|---|
| Chat Trigger | Receives the user request |
| AI Agent | Understands the request and coordinates actions |
| Ollama | Provides the local language model |
| Qwen 3 4B | Processes natural-language instructions |
| Simple Memory | Maintains conversation context |
| Calculator | Performs mathematical calculations |
| Find Employee | Searches for a specific employee |
| List All Employees | Retrieves all employees |
| Geocoding | Converts locations into coordinates |
| Weather | Retrieves current weather |
| Wikipedia | Searches general factual information |
| Data Table | Stores example employee records |

---

# 19. Local System Architecture

The project is designed around a local development environment.

```text
┌───────────────────────────────────────────────────┐
│                   Local Machine                   │
│                                                   │
│  ┌─────────────────┐      ┌───────────────────┐  │
│  │      n8n        │      │      Ollama       │  │
│  │                 │◄────►│                   │  │
│  │   AI Agent      │      │    Qwen 3 4B      │  │
│  │   Tools         │      │                   │  │
│  │   Memory        │      │  Port 11434       │  │
│  └────────┬────────┘      └───────────────────┘  │
│           │                                       │
│           │                                       │
│           ▼                                       │
│    ┌───────────────┐                              │
│    │ n8n Data      │                              │
│    │ Table         │                              │
│    │ employees     │                              │
│    └───────────────┘                              │
│                                                   │
└───────────────────────┬───────────────────────────┘
                        │
                        │ HTTP
                        ▼
             ┌──────────────────────┐
             │   External APIs      │
             │                      │
             │ Open-Meteo           │
             │ Wikipedia            │
             └──────────────────────┘
```

---

# 20. Local AI Design

The language model component is local:

```text
n8n
 ↓
Ollama
 ↓
Qwen 3 4B
```

External APIs are only used when the selected tool requires external information.

For example:

```text
Calculator
```

can perform a calculation directly.

Whereas:

```text
Weather
```

requires an external weather API.

---

# 21. Example Interaction Flows

## Example 1 — Memory

```text
User:
My name is Ajay.

       ↓

Chat Trigger

       ↓

AI Agent

       ↓

Simple Memory

       ↓

Response
```

Later:

```text
User:
What is my name?

       ↓

AI Agent

       ↓

Simple Memory

       ↓

Ajay
```

---

## Example 2 — Employee Search

```text
User:
Where does Priya work?

       ↓

AI Agent

       ↓

Find Employee

       ↓

employees Data Table

       ↓

Priya
Data Analyst
Chennai

       ↓

AI Agent

       ↓

Final Response
```

---

## Example 3 — Calculator

```text
User:
Calculate 125 * 48.

       ↓

AI Agent

       ↓

Calculator

       ↓

6000

       ↓

AI Agent

       ↓

Final Response
```

---

## Example 4 — Weather

```text
User:
What is the weather in Mumbai?

       ↓

AI Agent

       ↓

Geocoding

       ↓

Mumbai Coordinates

       ↓

Weather API

       ↓

Current Weather

       ↓

AI Agent

       ↓

Final Response
```

---

## Example 5 — Multi-Tool

```text
User:
What is the weather in Arjun's city?

       ↓

AI Agent

       ↓

Find Employee
       ↓
Arjun
       ↓
Mumbai

       ↓

Geocoding
       ↓
Mumbai Coordinates

       ↓

Weather API
       ↓
Current Weather

       ↓

AI Agent

       ↓

Final Response
```

---

# 22. Error Handling Principles

The AI Agent should not invent information when a tool does not provide the requested data.

For example, if the user asks:

```text
Where does John work?
```

and John does not exist in the employee Data Table, the agent should communicate that the employee was not found.

Similarly, the weather workflow should not invent coordinates.

The intended flow is:

```text
Location
   ↓
Geocoding
   ↓
Coordinates
   ↓
Weather
```

If geocoding does not return a valid location, the weather request should not be based on guessed coordinates.

---

# 23. Security Considerations

The repository is intended to contain configuration and example data, not secrets.

The following should not be committed to GitHub:

```text
API keys
Passwords
Access tokens
Private credentials
.env files
Production credentials
Private employee information
Database passwords
```

The `.gitignore` file is used to prevent common local configuration and secret files from being committed.

Before publishing an exported n8n workflow, review the JSON for credential information and other sensitive configuration.

---

# 24. Repository Architecture

The GitHub repository is organized as:

```text
n8n-ai-agent-lab/
│
├── README.md
│
├── .gitignore
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

### README.md

Provides the project overview and quick introduction.

### workflows/

Contains exported n8n workflows.

### data/

Contains example datasets used by the workflow.

### docs/

Contains detailed technical documentation.

---

# 25. Architecture Summary

The project can be summarized as:

```text
                  ┌─────────────┐
                  │    User     │
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │ Chat Trigger│
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │  AI Agent   │
                  └──────┬──────┘
                         │
       ┌─────────────────┼──────────────────┐
       │                 │                  │
       ▼                 ▼                  ▼
   ┌────────┐      ┌───────────┐      ┌──────────┐
   │ Ollama │      │  Memory   │      │  Tools   │
   │ Qwen   │      │           │      │          │
   │ 3:4b   │      │ Session   │      │ Calculator│
   └────────┘      └───────────┘      │ Employees│
                                      │ Weather  │
                                      │ Geocoding│
                                      │ Wikipedia│
                                      └────┬─────┘
                                           │
                         ┌─────────────────┼─────────────────┐
                         │                 │                 │
                         ▼                 ▼                 ▼
                    Data Table        Open-Meteo        Wikipedia
                    employees            APIs               API
                         │                 │
                         └─────────────────┴─────────────────┐
                                                           │
                                                           ▼
                                                     Tool Results
                                                           │
                                                           ▼
                                                      AI Agent
                                                           │
                                                           ▼
                                                      User Response
```

---

# 26. Key Concepts Demonstrated

This project demonstrates:

- Local LLM integration
- n8n AI Agents
- Ollama integration
- Qwen model integration
- AI Agent tool calling
- Tool selection
- Multi-tool execution
- Tool chaining
- Conversation memory
- Structured data access
- n8n Data Tables
- HTTP API integration
- External API orchestration
- Natural-language interfaces
- Workflow debugging
- Local AI development
- GitHub-based workflow versioning

---

# 27. Future Architecture Improvements

Potential future improvements include:

```text
Current
   │
   ▼
AI Agent + Local LLM
   │
   ├── Memory
   ├── Employee Tools
   ├── Calculator
   ├── Weather
   └── Wikipedia
```

Possible future expansion:

```text
AI Agent
   │
   ├── Database Tools
   ├── REST API Tools
   ├── File Processing
   ├── Email Automation
   ├── GitHub Automation
   ├── Monitoring
   ├── Logging
   └── Error Recovery
```

The project can therefore serve as a foundation for experimenting with increasingly capable AI-powered automation workflows.

---

# Conclusion

The **n8n AI Agent Lab** demonstrates how a local language model can be combined with n8n's workflow automation and tool-calling capabilities.

The system accepts natural-language requests, determines which capabilities are required, executes the appropriate tools, combines the results, and returns a user-friendly response.

The architecture intentionally keeps the language model local through Ollama while allowing the agent to interact with structured data and external APIs through n8n tools.