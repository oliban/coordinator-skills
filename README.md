# Coordinator Skills

Reusable Claude Code skills for multi-phase project coordination with parallel task execution.

## Skills

| Skill | Description |
|-------|-------------|
| `/coordinator-project-setup` | Interactive setup wizard that creates prd.json and a custom test skill |
| `/create-implementation-plan` | Create structured implementation plans with base plate + parallel phases |
| `/coordinate` | Execute plans by orchestrating coding, review, and test agents |

## Installation

Add to your project's `.claude/settings.json`:

```json
{
  "skills": {
    "imports": [
      {
        "source": "git@github.com:oliban/coordinator-skills.git",
        "path": "skills"
      }
    ]
  }
}
```

Or copy the `skills/` directory to your project's `.claude/skills/`.

## Usage

### 1. Set Up a New Feature

```
/coordinator-project-setup
```

This will:
1. Ask about your testing approach (browser, unit, integration, manual)
2. Gather build commands and dev server URL
3. Create a **custom test skill** at `.claude/skills/test-{project}/SKILL.md`
4. Create `prd.json` with tasks and config

### 2. Execute the Plan

```
/coordinate path/to/plan-folder
```

The coordinator will:
- Run base plate tasks sequentially
- Run parallel tasks concurrently
- Spawn coding, review, and test agents
- Test agents invoke your project's custom test skill
- Track progress in `progress.txt`

## Flow Diagram

```
/coordinator-project-setup
        │
        ▼
┌───────────────────────────────┐
│ 1. Interview user             │
│    - Testing mode             │
│    - Dev server URL           │
│    - Build commands           │
└───────────────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ 2. Create test skill          │
│    .claude/skills/test-{proj} │
│    (mode-specific template)   │
└───────────────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ 3. Create prd.json            │
│    - Tasks with criteria      │
│    - Config with skill ref    │
└───────────────────────────────┘
        │
        ▼
/coordinate path/to/plan
        │
        ▼
┌───────────────────────────────┐
│ For each task:                │
│   Coder → Reviewer → Tester   │
│                     ↓         │
│            Invokes /test-{proj}│
└───────────────────────────────┘
```

## Output Structure

After setup, your project will have:

```
your-project/
├── .claude/
│   └── skills/
│       └── test-{project}/
│           └── SKILL.md      # Custom test skill
└── {plan-folder}/
    ├── prd.json              # Task definitions
    ├── progress.txt          # Execution log
    └── specs/                # Optional task specs
```

## prd.json Schema

```json
{
  "project": "feature-name",
  "branchName": "feature/feature-name",
  "description": "What this implements",
  "config": {
    "testing": {
      "mode": "browser | unit | integration | manual",
      "skill": "/test-feature-name",
      "devServerUrl": "http://localhost:8080",
      "devServerCommand": "python3 -m http.server 8080"
    },
    "build": {
      "command": "npm run build",
      "testCommand": "npm test"
    }
  },
  "userStories": [
    {
      "id": "BP-001",
      "title": "Base plate: Core setup",
      "description": "Foundational task",
      "acceptanceCriteria": ["Files exist", "Build passes"],
      "priority": 1,
      "passes": false,
      "blockedBy": [],
      "isBasePlate": true
    }
  ]
}
```

## Testing Modes

| Mode | Test Skill Behavior | Pre-flight Check |
|------|---------------------|------------------|
| `browser` | Chrome automation, JS verification, screenshots | Chrome extension + dev server |
| `unit` | Run test command, parse output | None |
| `integration` | CLI commands, API calls, curl | Optional server check |
| `manual` | Present criteria to user via AskUserQuestion | None |

## Task Types

- **Base Plate** (`BP-*`): Sequential tasks that form the foundation
- **Parallel** (`PAR-*`): Independent tasks that run concurrently after base plate

## License

MIT
