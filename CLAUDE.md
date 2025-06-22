# Claude Context for Wuselig

## Project Overview
Wuselig is an agentic AI system for building video games using a serverless GitHub Actions architecture. This document provides technical context and guidelines for Claude when working with this codebase.

## Core Architecture
- **GitHub Issues**: Primary interface for human-agent interaction
- **GitHub Actions**: Serverless compute platform for agent execution
- **LangChain**: Agent framework with LLM integration and tool calling
- **GitHub API**: Full CRUD operations on repositories via agents
- **Webhook Triggers**: Lightweight functions to bridge GitHub events to Actions

## Architecture Design

### Core Components
1. **Agent Base Classes** (`agents/base.py`)
   - `BaseAgent`: Abstract class for all agents with LangChain integration
   - `GitHubAgent`: Base for agents that interact with GitHub API
   - Common interfaces for LLM calls, tool usage, and GitHub operations

2. **GitHub Integration** (`github/`)
   - Issue parsing and creation
   - PR management and automation  
   - File CRUD operations
   - Repository analysis tools

3. **Agent Implementations** (`agents/`)
   - Each agent is a standalone Python module
   - Invoked directly by GitHub Actions
   - Uses LangChain tools for LLM reasoning and GitHub operations
   - Stateless execution with context from GitHub issue

4. **Webhook Handler** (`webhook/`)
   - Lightweight Vercel function
   - Receives GitHub webhooks
   - Triggers appropriate GitHub Action workflows
   - Passes issue context to agents

5. **GitHub Actions Workflows** (`.github/workflows/`)
   - Agent execution environment
   - Python setup and dependency installation
   - Secure environment variable management
   - Logging and error handling

### Development Guidelines

#### Code Style
- Type hints for all functions
- Docstrings for public methods
- Follow PEP 8
- Use `black` for formatting
- Use `mypy` for type checking

#### Agent Development
- Each agent should be stateless
- Use dependency injection for external services
- Log all decisions and actions
- Implement proper error handling and retries

#### LangChain Patterns
- Use structured outputs for agent responses
- Implement memory for context retention
- Use tools/functions for external interactions
- Chain agents using LangGraph for complex workflows

### Testing
- `pytest` for unit tests
- Mock external APIs (GitHub, LLMs)
- Test agent decisions with fixed prompts
- Integration tests for full workflows

### Common Commands
```bash
# Setup
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install -e ".[dev]"

# Development
pytest                    # Run tests
black .                   # Format code
mypy .                   # Type check
python -m wuselig.api    # Start API server

# Agent Testing
python -m wuselig.agents.planner --dry-run
python -m wuselig.orchestrator --issue-id=123
```

## Project Structure
```
wuselig/
├── agents/                    # Agent implementations
│   ├── __init__.py
│   ├── base.py               # Base classes with LangChain integration
│   ├── planner.py            # Planning agent - breaks down issues
│   ├── coder.py              # Code generation agent - implements features
│   ├── reviewer.py           # Code review agent - analyzes PRs
│   ├── tester.py             # Test generation agent - creates tests
│   └── documenter.py         # Documentation agent - updates docs
├── github/                   # GitHub API integration
│   ├── __init__.py
│   ├── client.py             # GitHub API wrapper with retry logic
│   ├── models.py             # Pydantic models for GitHub data
│   └── tools.py              # LangChain tools for GitHub operations
├── webhook/                  # Webhook handler (deployed to Vercel)
│   ├── handler.py            # Main webhook receiver
│   └── vercel.json           # Vercel deployment config
├── prompts/                  # LangChain prompt templates
│   ├── planner_prompts.py
│   ├── coder_prompts.py
│   └── reviewer_prompts.py
├── .github/
│   └── workflows/            # GitHub Actions workflows
│       ├── agent-runner.yml  # Main agent execution workflow
│       └── setup.yml         # Environment setup
├── tests/                    # Test suite
│   ├── test_agents.py
│   ├── test_github.py
│   └── fixtures/
└── docs/                     # Documentation
    └── game-patterns/        # RAG knowledge base for agents
```

## Key Concepts

### Agent Communication
- Agents communicate through structured messages
- Each message has: type, sender, recipient, payload
- Orchestrator routes messages and maintains history

### Workflow State
- Stored in orchestrator
- Tracks: current phase, active agents, completed tasks
- Persisted to allow resume on failure

### GitHub Integration
- Issues are the primary work unit
- Each issue spawns a workflow
- PRs are created automatically
- Human approval required for merge

### Agent Invocation via GitHub

#### Setup Requirements
1. **GitHub App (Recommended)**
   - Single app can act as multiple agents
   - No email/account needed
   - Better rate limits than personal tokens
   - Can be installed on multiple repos
   - Permissions:
     - Issues (read/write)
     - Pull requests (write)
     - Repository contents (write)
     - Webhooks (admin)
   
   **How it works:**
   - App posts as "YourApp[bot]"
   - Use different comment signatures/headers to indicate agent type
   - Example: "🤖 Planning Agent Analysis:" vs "🔧 Code Agent Implementation:"

2. **Alternative: Single Bot Account**
   - Create ONE bot user (e.g., @wuselig-bot)
   - Use it for all agents
   - Differentiate agents via:
     - Comment formatting
     - Status checks (for PR reviews)
     - Labels they apply

2. **Webhook Configuration**
   - Listen for `issues` events (opened, edited, labeled)
   - Listen for `issue_comment` events
   - Webhook endpoint: `https://your-api.com/github/webhooks`

3. **Agent Assignment Methods**
   ```
   Method 1: Labels (Cleanest)
   - Apply label like "agent:planner" or "agent:coder"
   - Webhook triggers on label addition
   - Single GitHub App handles all agent types
   
   Method 2: Commands in Comments
   - "/wuselig plan" or "/wuselig code"
   - Similar to how GitHub Copilot commands work
   - Parse commands from issue comments
   
   Method 3: Issue Template Fields
   - Dropdown to select agent type
   - Metadata in issue body (parsed by webhook)
   ```

#### Example GitHub App Manifest
```json
{
  "name": "Wuselig",
  "description": "AI agents for game development",
  "url": "https://github.com/yourusername/wuselig",
  "hook_attributes": {
    "url": "https://your-api.com/github/webhooks"
  },
  "redirect_url": "https://your-api.com/github/callback",
  "public": false,
  "default_permissions": {
    "issues": "write",
    "pull_requests": "write",
    "contents": "write",
    "metadata": "read"
  },
  "default_events": [
    "issues",
    "issue_comment",
    "pull_request",
    "pull_request_review"
  ]
}
```

#### Issue Templates
```yaml
# .github/ISSUE_TEMPLATE/agent-task.yml
name: Agent Task
description: Create a task for Wuselig agents
body:
  - type: dropdown
    id: agent
    attributes:
      label: Agent Type
      options:
        - Planning Agent
        - Code Generation Agent
        - Review Agent
        - Testing Agent
        - Documentation Agent
  - type: textarea
    id: description
    attributes:
      label: Task Description
      description: What should the agent do?
  - type: textarea
    id: context
    attributes:
      label: Additional Context
      description: Any relevant files, dependencies, or constraints
```

#### Workflow Example
1. User creates issue: "Build a player movement system"
2. User adds label: `agent:planner`
3. Webhook fires to Wuselig API
4. Planning Agent:
   - Reads issue content
   - Creates subtask issues
   - Comments with plan
5. User reviews plan, adds `agent:coder` to subtasks
6. Code Agent creates PRs for each subtask
7. Review Agent auto-comments on PRs

### Error Handling
- Agents should never crash the system
- Failed tasks are retried with backoff
- Human intervention requested after max retries

## Environment Variables
```bash
GITHUB_TOKEN=           # GitHub API access
OPENAI_API_KEY=        # For GPT models
ANTHROPIC_API_KEY=     # For Claude models
WUSELIG_LOG_LEVEL=     # DEBUG, INFO, WARNING, ERROR
WUSELIG_WEBHOOK_SECRET= # GitHub webhook validation
```

## Deployment Options

### 1. Serverless (Recommended for starting)
- **GitHub Actions**: Run agents directly in workflows
  - Free 2000 minutes/month
  - No hosting needed
  - Limited to 6 hour runs
- **Vercel/Netlify Functions**: For webhook receiver
- **AWS Lambda**: For webhook + agent execution

### 2. Container Hosting
- **Fly.io**: Free tier with 3 shared VMs
- **Railway**: $5 credit/month free
- **Google Cloud Run**: Generous free tier
- **Render**: Free tier available

### 3. Hybrid Approach (Recommended)
```
GitHub Issue (with label) → GitHub Webhook → Vercel Function → Trigger GitHub Action
                                                                    ↓
                                                            Python/LangChain Agent
                                                                    ↓
                                                    ┌───────────────┴───────────────┐
                                                    ↓                               ↓
                                            LangChain Tools:                 GitHub Toolkit:
                                            - LLM (GPT/Claude)             - Read issue details
                                            - RAG (game docs)              - Comment on issues  
                                            - Code analysis               - Create/edit files
                                            - Memory/context              - Create PRs
                                                                         - Apply labels
```

#### GitHub Actions Workflow Example
```yaml
# .github/workflows/agent-runner.yml
name: Run Wuselig Agent
on:
  repository_dispatch:
    types: [agent-trigger]

jobs:
  run-agent:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install langchain openai anthropic pygithub
          pip install -e .
      
      - name: Run Agent
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          ISSUE_NUMBER: ${{ github.event.client_payload.issue_number }}
          AGENT_TYPE: ${{ github.event.client_payload.agent_type }}
        run: |
          python -m wuselig.agents.$AGENT_TYPE \
            --issue=$ISSUE_NUMBER \
            --repo=${{ github.repository }}
```

#### Agent Implementation Pattern
```python
# wuselig/agents/planner.py
from langchain import LLMChain, PromptTemplate
from langchain.tools import Tool
from github import Github
import os

class PlannerAgent:
    def __init__(self, issue_number: int, repo_name: str):
        self.github = Github(os.environ['GITHUB_TOKEN'])
        self.repo = self.github.get_repo(repo_name)
        self.issue = self.repo.get_issue(issue_number)
        
        # LangChain tools
        self.tools = [
            Tool(name="read_issue", func=self.read_issue),
            Tool(name="comment_on_issue", func=self.comment),
            Tool(name="create_subtask", func=self.create_subtask),
            Tool(name="search_codebase", func=self.search_code),
        ]
        
    def run(self):
        # Read issue, analyze with LLM, create plan, etc.
        pass
```

### 4. Local Development
- **ngrok**: Expose local server for webhooks
- **smee.io**: GitHub's webhook proxy for testing