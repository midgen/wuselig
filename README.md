# wuselig
Agentic AI for building video games

## Overview
Wuselig is a multi-agent system designed to collaboratively build video games. It uses LangChain to orchestrate specialized AI agents that handle different aspects of game development, from planning to implementation.

## Architecture

### Agent Types
- **Planning Agent**: Decomposes game concepts into actionable development tasks
- **Code Generation Agent**: Implements game features and mechanics
- **Review Agent**: Analyzes code quality and suggests improvements
- **Testing Agent**: Creates and executes test suites
- **Documentation Agent**: Maintains technical documentation and guides

### Workflow

#### Basic Flow
1. Create a GitHub issue describing what you want built
2. Invoke an agent by either:
   - Adding a label (e.g., `agent:planner`)
   - Mentioning the agent (e.g., `@wuselig-planner`)
   - Using the issue template dropdown
3. Agent reads the issue and begins work
4. Agent responds via comments and/or PRs
5. Human reviews and provides feedback
6. Process continues until task is complete

#### Example Interaction
```
User: Creates issue "Add jumping mechanic to platformer"
User: Adds label "agent:planner"
Planning Agent: Comments with breakdown:
  - Add jump input detection
  - Implement gravity system
  - Add jump physics
  - Create jump animation triggers
  - Add sound effects
User: Approves plan, adds "agent:coder" to subtasks
Code Agent: Creates PR for each subtask
Review Agent: Automatically reviews PRs
User: Merges approved PRs
```

## Tech Stack
- **Python 3.10+**: Core language
- **LangChain**: Agent orchestration and LLM integration
- **GitHub Actions**: Serverless compute for agents
- **GitHub API**: Issue tracking, PR management, and file operations
- **OpenAI/Anthropic APIs**: LLM providers for reasoning
- **Vercel Functions**: Webhook receiver (lightweight)

## Architecture Overview

Wuselig uses a hybrid serverless architecture:

```
GitHub Issue → Webhook → Vercel Function → GitHub Action → LangChain Agent
     ↓                                                           ↓
  User adds                                               Agent executes:
agent:planner                                            - Reads issue context
   label                                                 - Calls LLM for reasoning
                                                         - Uses RAG for game patterns
                                                         - Creates PRs/comments
                                                         - Updates codebase
```

### Key Benefits
- **Zero infrastructure costs** - Runs entirely on GitHub Actions free tier
- **Scalable** - Each agent runs in isolated GitHub Action
- **Secure** - Full access to private repos without exposing credentials
- **Auditable** - All agent actions logged in GitHub UI
- **Human-in-the-loop** - Natural PR review process

## Getting Started
1. Install Wuselig GitHub App on your game repository
2. Create issues describing features you want built  
3. Add agent labels (e.g., `agent:planner`) to trigger agents
4. Review and merge the PRs agents create
5. Iterate with feedback via issue comments

## License
MIT
