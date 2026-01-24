# Coordinator Skills

Reusable Claude Code skills for multi-phase project coordination with parallel task execution.

## Skills

| Skill | Description |
|-------|-------------|
| `/coordinator-project-setup` | Interactive setup wizard that creates prd.json with project config |
| `/create-implementation-plan` | Create structured implementation plans with base plate + parallel phases |
| `/coordinate` | Execute plans by orchestrating coding, review, and test agents |

## Installation

Add to your project's `.claude/settings.json`:

```json
{
  "skills": {
    "source": "git@github.com:oliban/coordinator-skills.git",
    "path": "skills"
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
- Ask about your testing approach (browser, unit, integration, manual)
- Gather build commands
- Create `prd.json` with tasks and config

### 2. Execute the Plan

```
/coordinate path/to/plan-folder
```

The coordinator will:
- Run base plate tasks sequentially
- Run parallel tasks concurrently
- Spawn coding, review, and test agents
- Track progress in `progress.txt`

## prd.json Schema

```json
{
  "project": "feature-name",
  "branchName": "feature/feature-name",
  "description": "What this implements",
  "config": {
    "testing": {
      "mode": "browser | unit | integration | manual",
      "skill": "/test-game",
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

| Mode | When to Use | Pre-flight |
|------|-------------|------------|
| `browser` | Web UI testing with Chrome extension | Chrome extension + dev server |
| `unit` | Projects with test suites | None |
| `integration` | API/CLI testing | Optional server check |
| `manual` | User verifies each task | None |

## Task Types

- **Base Plate** (`BP-*`): Sequential tasks that form the foundation
- **Parallel** (`PAR-*`): Independent tasks that run concurrently after base plate

## License

MIT
