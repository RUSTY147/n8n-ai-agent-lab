# Tools Documentation

This document describes the tools connected to the **n8n AI Agent Lab**.

The AI Agent uses these tools to perform actions and retrieve information instead of relying only on the language model.

---

# 1. Tool Architecture

The AI Agent is connected to several tools:

```text
                         ┌──────────────────┐
                         │     AI Agent     │
                         └────────┬─────────┘
                                  │
              ┌───────────────────┼────────────────────┐
              │                   │                    │
              ▼                   ▼                    ▼
       ┌─────────────┐     ┌──────────────┐     ┌──────────────┐
       │ Calculator  │     │   Employee   │     │   External   │
       │             │     │    Tools     │     │     APIs     │
       └─────────────┘     └──────┬───────┘     └──────┬───────┘
                                  │                    │
                         ┌────────┴────────┐      ┌────┴─────────┐
                         │                 │      │              │
                         ▼                 ▼      ▼              ▼
                  Find Employee    List Employees  Geocoding   Wikipedia
                                                    │
                                                    ▼
                                                  Weather
```

---

# 2. Calculator

## Purpose

The Calculator tool performs mathematical calculations.

It is useful when the user asks the AI Agent to perform arithmetic operations.

---

## Example

User:

```text
Calculate 347 * 829.
```

The AI Agent can delegate the calculation to the Calculator tool.

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

---

## Another Example

```text
Calculate 125 * 48.
```

Result:

```text
6000
```

---

## Why Use a Tool?

Mathematical calculations can be delegated to a deterministic calculation tool rather than asking the language model to perform the arithmetic itself.

This provides a clear separation between:

```text
Natural Language Understanding
          ↓
      AI Agent
          ↓
    Calculation
          ↓
      Calculator
```

---

# 3. Find Employee

## Purpose

The **Find Employee** tool searches the n8n Data Table for a specific employee.

The Data Table is:

```text
employees
```

---

## Available Fields

The example employee dataset contains:

| Field | Description |
|---|---|
| Name | Employee name |
| Role | Employee job role |
| Location | Employee location |

---

## Example Data

```text
Ajay
Data Engineer
Hyderabad
```

```text
Rahul
Developer
Bangalore
```

```text
Priya
Data Analyst
Chennai
```

```text
Arjun
Software Engineer
Mumbai
```

---

## Example Query

User:

```text
Where does Ajay work?
```

The AI Agent can call:

```text
Find Employee
```

The tool searches the Data Table:

```text
employees
```

and returns the matching employee record.

---

## Flow

```text
User
  ↓
AI Agent
  ↓
Find Employee
  ↓
employees Data Table
  ↓
Employee Record
  ↓
AI Agent
  ↓
Final Response
```

---

## Missing Employee

Example:

```text
Where does John work?
```

If John does not exist in the table, the tool should return no matching record.

The AI Agent should then communicate that the employee was not found.

It should not invent employee information.

---

# 4. List All Employees

## Purpose

The **List All Employees** tool retrieves all records from the employee Data Table.

---

## Example Query

```text
Who are all the employees?
```

The tool retrieves all records.

Example result:

```text
Ajay    → Data Engineer       → Hyderabad
Rahul   → Developer           → Bangalore
Priya   → Data Analyst        → Chennai
Arjun   → Software Engineer   → Mumbai
```

---

## Flow

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

# 5. Employee Data Source

The repository contains the example employee dataset:

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

The n8n workflow uses a Data Table named:

```text
employees
```

The JSON file serves as the example source data for reproducing the demonstration environment.

---

# 6. Geocoding

## Purpose

The Geocoding tool converts a city or location name into geographic coordinates.

The workflow uses the Open-Meteo Geocoding API.

Endpoint:

```text
https://geocoding-api.open-meteo.com/v1/search
```

---

## Input

The tool receives a location name.

Example:

```text
Mumbai
```

---

## Output

The API returns location information including:

```text
Latitude
Longitude
```

These coordinates can then be passed to the Weather tool.

---

## Flow

```text
Location Name
      ↓
Geocoding Tool
      ↓
Open-Meteo Geocoding API
      ↓
Latitude + Longitude
```

---

# 7. Weather

## Purpose

The Weather tool retrieves current weather information.

The workflow uses the Open-Meteo Forecast API.

Endpoint:

```text
https://api.open-meteo.com/v1/forecast
```

---

## Required Inputs

The weather request requires:

```text
Latitude
Longitude
```

The workflow requests current weather information including:

```text
temperature_2m
wind_speed_10m
```

---

## Flow

```text
Latitude + Longitude
        ↓
Weather Tool
        ↓
Open-Meteo Forecast API
        ↓
Current Weather
```

---

# 8. Weather + Geocoding

The Weather tool normally requires coordinates.

Users, however, usually provide a city name.

For example:

```text
What is the weather in Mumbai?
```

The agent can therefore coordinate two tools:

```text
Mumbai
   ↓
Geocoding
   ↓
Latitude + Longitude
   ↓
Weather
   ↓
Current Weather
```

This is an example of **tool chaining**.

---

# 9. Wikipedia

## Purpose

The Wikipedia tool searches Wikipedia for general factual information.

It is implemented using an HTTP Request tool.

The tool can be used for topics such as:

- People
- Places
- Technologies
- Historical subjects
- General concepts

---

## Example

User:

```text
Who is Alan Turing?
```

The AI Agent can use the Wikipedia tool to search for relevant information.

---

## Flow

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

# 10. Wikipedia API Request

The workflow uses the Wikipedia API endpoint:

```text
https://en.wikipedia.org/w/api.php
```

The request performs a search against Wikipedia.

Important request parameters include:

```text
action=query
list=search
srsearch=<user topic>
format=json
srlimit=3
```

The `srsearch` value is supplied based on the user's requested topic.

---

# 11. Tool Descriptions

The AI Agent is provided with descriptions for each tool.

Tool descriptions help the model understand:

1. What the tool does.
2. When the tool should be used.
3. What parameters are required.
4. What information should be passed.
5. What the tool should not be used for.

For example, the employee tool is intended for questions about specific employees.

The weather tool requires geographic coordinates.

The geocoding tool is used to obtain those coordinates.

This allows the AI Agent to select tools based on the task.

---

# 12. Tool Selection

The AI Agent determines which tool is appropriate based on the user's request.

Examples:

| User Request | Tool |
|---|---|
| `Calculate 25 * 20` | Calculator |
| `Where does Ajay work?` | Find Employee |
| `Who are all the employees?` | List All Employees |
| `What is the weather in Mumbai?` | Geocoding + Weather |
| `Who is Alan Turing?` | Wikipedia |
| `What is the weather in Arjun's city?` | Find Employee + Geocoding + Weather |

---

# 13. Multi-Tool Execution

The most important tool demonstration is multi-tool execution.

Example:

```text
What is the weather in Arjun's city?
```

The AI Agent needs information from multiple tools.

---

## Step 1 — Find Employee

```text
Find Employee
      ↓
Arjun
      ↓
Mumbai
```

---

## Step 2 — Geocode Mumbai

```text
Mumbai
      ↓
Geocoding
      ↓
Latitude + Longitude
```

---

## Step 3 — Get Weather

```text
Latitude + Longitude
      ↓
Weather
      ↓
Current Weather
```

---

## Step 4 — Generate Response

```text
Tool Results
      ↓
AI Agent
      ↓
Natural-Language Response
```

---

## Complete Flow

```text
                         User
                           │
                           ▼
                    ┌────────────┐
                    │  AI Agent  │
                    └─────┬──────┘
                          │
                          ▼
                  ┌───────────────┐
                  │Find Employee  │
                  └───────┬───────┘
                          │
                          ▼
                    Arjun → Mumbai
                          │
                          ▼
                  ┌───────────────┐
                  │   Geocoding   │
                  └───────┬───────┘
                          │
                          ▼
                  Coordinates
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
                    AI Agent
                          │
                          ▼
                    User Response
```

---

# 14. Tool Dependencies

Some tools are independent.

For example:

```text
Calculator
```

does not require another tool.

However, some tools have dependencies.

Weather requires coordinates.

Therefore:

```text
Geocoding
     ↓
Coordinates
     ↓
Weather
```

An employee-based weather request introduces another dependency:

```text
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

---

# 15. Tool Usage Rules

The AI Agent follows several important rules.

### Employee Tools

Use employee tools when the user asks about an employee.

Do not invent employee information.

---

### Weather

Use geocoding before weather when only a location name is available.

Do not guess coordinates.

---

### Calculator

Use the calculator for arithmetic operations when appropriate.

---

### Wikipedia

Use Wikipedia for general factual searches that are suitable for the tool.

---

### Multiple Tools

When a request requires multiple tools, execute them in the required sequence.

---

# 16. Tool Error Handling

Tools can return no result or an error.

The AI Agent should handle these situations clearly.

For example:

```text
Where does John work?
```

If John does not exist:

```text
Employee John was not found.
```

The agent should not create fictional employee information.

---

## Invalid Weather Location

If a location cannot be resolved:

```text
User Location
      ↓
Geocoding
      ↓
No Result
      ↓
Do Not Guess Coordinates
```

The agent should communicate that the location could not be resolved.

---

# 17. Tool Calling vs Direct LLM Response

Without tools, the LLM can only generate an answer from its model context.

With tools:

```text
User
 ↓
AI Agent
 ↓
Tool
 ↓
Real / Structured Data
 ↓
AI Agent
 ↓
Response
```

This allows the agent to interact with information and operations outside the language model itself.

---

# 18. Tool Categories

The tools in this project can be grouped into three categories.

## Computation

```text
Calculator
```

Used for deterministic mathematical operations.

---

## Structured Data

```text
Find Employee
List All Employees
```

Used to retrieve information from the n8n Data Table.

---

## External APIs

```text
Geocoding
Weather
Wikipedia
```

Used to retrieve information from external services.

---

# 19. Current Tool Inventory

| Tool | Category | Input | Output |
|---|---|---|---|
| Calculator | Computation | Mathematical expression | Calculation result |
| Find Employee | Structured Data | Employee name | Employee record |
| List All Employees | Structured Data | None | Employee records |
| Geocoding | External API | Location name | Coordinates |
| Weather | External API | Latitude + longitude | Current weather |
| Wikipedia | External API | Search topic | Search results |

---

# 20. Example Tool Selection Matrix

```text
User Request
      │
      ▼
┌───────────────────────────────┐
│       AI Agent Analysis       │
└───────────────┬───────────────┘
                │
        ┌───────┼────────┐
        │       │        │
        ▼       ▼        ▼
     Math?   Employee?  General
        │       │       │
        ▼       ▼       ▼
 Calculator  Employee  Wikipedia
             Tools
                │
                ▼
          Location Query?
                │
                ▼
             Geocoding
                │
                ▼
             Weather
```

---

# 21. Testing the Tools

Each tool can be tested independently before testing multi-tool execution.

### Calculator

```text
Calculate 125 * 48.
```

### Find Employee

```text
Where does Ajay work?
```

### List Employees

```text
Who are all the employees?
```

### Geocoding

```text
What are the coordinates of Mumbai?
```

### Weather

```text
What is the weather in Mumbai?
```

### Wikipedia

```text
Who is Alan Turing?
```

### Multi-Tool

```text
What is the weather in Arjun's city?
```

---

# 22. Tool Execution Philosophy

The purpose of using tools is to separate different responsibilities.

The AI Agent handles:

```text
Understanding
Decision Making
Tool Selection
Response Generation
```

The tools handle:

```text
Calculations
Data Retrieval
API Requests
External Information
```

This creates a modular architecture.

```text
                 AI Agent
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
    Memory        Tools        Model
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
  Structured    Computation   External
     Data                       APIs
```

---

# 23. Adding New Tools

The architecture is designed so that additional tools can be connected to the AI Agent.

Potential future tools include:

```text
Database Tool
     ↓
SQL Query Tool
     ↓
File Search Tool
     ↓
GitHub Tool
     ↓
Email Tool
     ↓
REST API Tool
```

A new tool should have:

1. A clear description.
2. Clearly defined inputs.
3. Appropriate parameter descriptions.
4. Clear usage instructions.
5. Error handling.
6. Appropriate security considerations.

---

# 24. Security Considerations

Tools that interact with external systems can potentially access sensitive information.

Never commit:

```text
API keys
Passwords
Access tokens
Database credentials
Private tokens
Production secrets
```

Use n8n credentials or environment-based configuration where appropriate.

The repository should contain only the configuration and example data required to reproduce the demonstration.

---

# 25. Tool Architecture Summary

The current AI Agent tool architecture is:

```text
                         AI AGENT
                            │
       ┌────────────────────┼────────────────────┐
       │                    │                    │
       ▼                    ▼                    ▼
   Calculator          Employee Tools       External APIs
                            │                    │
                     ┌──────┴──────┐       ┌─────┴─────┐
                     │             │       │           │
                     ▼             ▼       ▼           ▼
               Find Employee  List All  Geocoding   Wikipedia
                              Employees      │
                                            ▼
                                          Weather
```

The architecture allows the AI Agent to combine multiple capabilities in a single conversation.

---

# Conclusion

The tools are the main mechanism that allows the AI Agent to move beyond simple text generation.

The project demonstrates three important concepts:

```text
AI Agent
   +
Tools
   +
Tool Chaining
   =
Action-Oriented AI Workflow
```

The current implementation combines local AI through Ollama with n8n's tool-calling capabilities, structured employee data, deterministic calculations, and external APIs.

This provides a foundation for extending the project into more advanced AI-powered automation workflows.