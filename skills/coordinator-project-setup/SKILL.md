---
name: coordinator-project-setup
description: Use when setting up a project to run with multi-worker coordinator, structuring features.json, creating PRD docs, or preparing for parallel feature implementation
hooks:
  PreToolUse:
    - matcher: "Write"
      hooks:
        - type: command
          command: |
            input=$(cat)
            file_path=$(echo "$input" | jq -r '.tool_input.file_path // empty')
            content=$(echo "$input" | jq -r '.tool_input.content // empty')
            if [[ "$file_path" =~ \.claude/skills/test-.*/SKILL\.md$ ]]; then
              if echo "$content" | grep -qE '\{(devServerUrl|project-name|testingMode|config\.build\.testCommand)\}'; then
                echo "BLOCKED: Test skill contains placeholder values like {devServerUrl} or {project-name}." >&2
                echo "" >&2
                echo "You skipped the interview. Go back to Step 3 and use AskUserQuestion to ask:" >&2
                echo "  1. Testing mode (browser/unit/integration/manual)" >&2
                echo "  2. Dev server URL (if browser mode)" >&2
                echo "  3. Build command" >&2
                echo "" >&2
                echo "Then replace ALL placeholders with the user's actual answers." >&2
                exit 2
              fi
            fi
            exit 0
---

# Coordinator Project Setup

Sets up the project structure required for the coordinator agent to execute multi-phase plans.

## When to Use

Use this skill when:
- Starting a new feature that will be coordinated
- User has requirements but no prd.json yet
- Existing plan needs conversion to prd.json format
- Setting up progress.txt and folder structure

## Output Structure

This skill creates:

```
{plan_folder}/
├── prd.json          # Machine-readable task list
├── progress.txt      # Append-only learnings log
└── specs/            # Optional detailed spec files
    ├── BP-001-*.md
    ├── PAR-001-*.md
    └── ...

.claude/skills/
└── test-{project}/
    └── SKILL.md      # Project-specific test skill
```

## Workflow

### Step 1: Gather Requirements

If requirements are unclear, ask:
1. What is the feature/project name?
2. What are the main components/features to implement?
3. Are there any ordering dependencies?
4. What validation/testing is needed?

### Step 2: Create Plan Folder

```bash
mkdir -p {plan_folder}
mkdir -p {plan_folder}/specs
```

### Step 3: Interview User About Project Config

**⚠️ REQUIRED: DO NOT SKIP THIS STEP**

You MUST ask the user these questions before creating any test skill.
Do NOT proceed to Step 4 until you have received answers to ALL questions below.

---

**Question 1: Testing Mode** (REQUIRED)

Ask the user NOW using AskUserQuestion:
- question: "How should tasks be verified in this project?"
- header: "Testing"
- options:
  - "Browser testing" - Chrome extension tests web UI (games, dashboards)
  - "Unit tests" - Run test command (npm test, pytest, etc.)
  - "Integration tests" - Run commands, check API responses
  - "Manual" - I'll verify each task myself

**Store the answer as `testingMode`.**

---

**Question 2: Browser Config** (REQUIRED if testingMode = "Browser testing")

If the user selected "Browser testing", ask NOW using AskUserQuestion:
- question: "What URL does the dev server run on?"
- header: "Dev URL"
- options:
  - "localhost:8080" - Python http.server, common default
  - "localhost:3000" - React/Next.js default
  - "localhost:5173" - Vite default

**Store the answer as `devServerUrl`.**

---

**Question 3: Build Commands** (REQUIRED)

Ask the user NOW using AskUserQuestion:
- question: "How do you build/validate the project?"
- header: "Build"
- options:
  - "npm run build" - Node.js project
  - "No build step" - Vanilla JS, Python, etc.

**Store the answer as `buildCommand`.**

---

### Checkpoint: Verify Interview Complete

**⛔ STOP: Do not proceed until you have answers.**

Before proceeding to Step 4, confirm you have collected:
- [ ] `testingMode` from Question 1 (REQUIRED)
- [ ] `devServerUrl` from Question 2 (REQUIRED if browser mode)
- [ ] `buildCommand` from Question 3 (REQUIRED)

If any required answers are missing, GO BACK and ask the user now.

### Step 4: Create Custom Test Skill

**Prerequisite:** You must have completed the interview in Step 3 with these values:
- `testingMode` - determines which template to use below
- `devServerUrl` - used in Browser template (if applicable)
- `buildCommand` - stored in prd.json config

Create a project-specific test skill using the skill creator:

1. **Invoke the skill creator:**
   ```
   Skill({ skill: "superpowers:writing-skills" })
   ```

2. **Create the test skill** at `.claude/skills/test-{project-name}/SKILL.md`
   - Select the template below based on `testingMode` from Step 3
   - Replace placeholders with actual values from interview

3. **Set skill reference** in prd.json config:
   ```json
   "testing": {
     "skill": "/test-{project-name}"
   }
   ```

#### Test Skill Templates by Mode

Use `testingMode` from Step 3 to select the correct template:

**Browser Mode:**

```markdown
---
name: test-{project-name}
description: Browser testing for {project-name} via Chrome automation.
---

# Test Agent for {project-name}

You verify acceptance criteria using Chrome browser automation.

## Prerequisites
- Dev server running at {devServerUrl}
- Chrome with Claude extension active

## How to Test
1. Get browser context: `mcp__claude-in-chrome__tabs_context_mcp`
2. Navigate to {devServerUrl}
3. For each criterion:
   - Execute JavaScript verification
   - Take screenshots for visual checks
   - Capture console errors

## Output Format
PASS: X/Y criteria met (with evidence)
FAIL: List failures with details
```

**Unit Mode:**

```markdown
---
name: test-{project-name}
description: Unit test runner for {project-name}.
---

# Test Agent for {project-name}

You verify acceptance criteria by running the test suite.

## Test Command
{config.build.testCommand}

## How to Test
1. Run the test command
2. Parse output for failures
3. Check each criterion against results

## Output Format
PASS: All tests pass, criteria met
FAIL: List failures with test output
```

**Integration Mode:**

```markdown
---
name: test-{project-name}
description: Integration testing for {project-name}.
---

# Test Agent for {project-name}

You verify acceptance criteria via CLI commands and API calls.

## How to Test
1. Execute integration commands (curl, CLI, etc.)
2. Verify responses match expected values
3. Capture output as evidence

## Output Format
PASS: All criteria met (with evidence)
FAIL: List failures with command output
```

**Manual Mode:**

```markdown
---
name: test-{project-name}
description: Manual verification for {project-name}.
---

# Test Agent for {project-name}

You present acceptance criteria to the user for manual verification.

## How to Test
1. Present each criterion to the user
2. Use AskUserQuestion for each:
   - "Does {criterion} pass?"
   - Options: Pass, Fail
3. Collect responses

## Output Format
PASS: User confirmed all criteria
FAIL: List criteria user marked as failed
```

### Step 5: Generate prd.json

Create `prd.json` following this schema:

```json
{
  "project": "feature-name",
  "branchName": "feature/feature-name",
  "description": "High-level description of what this implements",
  "config": {
    "testing": {
      "mode": "browser | unit | integration | manual",
      "skill": "/test-game",
      "devServerUrl": "http://localhost:8080",
      "devServerCommand": "python3 -m http.server 8080"
    },
    "build": {
      "command": "npm run build",
      "lintCommand": "npm run lint",
      "testCommand": "npm test"
    }
  },
  "userStories": [
    {
      "id": "BP-001",
      "title": "Base plate: Foundation",
      "description": "Detailed description of this task",
      "acceptanceCriteria": [
        "Testable criterion 1",
        "Testable criterion 2"
      ],
      "priority": 1,
      "passes": false,
      "blockedBy": [],
      "isBasePlate": true
    }
  ]
}
```

### Step 6: Initialize progress.txt

```bash
cat > {plan_folder}/progress.txt << 'EOF'
# Progress Log
# Append-only learnings from coordination
# Project: {project-name}
# Started: {timestamp}
#
# Format: [{timestamp}] Task {id} {STATUS}: {message}
EOF
```

### Step 7: Create Spec Files (Optional)

For complex tasks, create individual spec files:

```markdown
# Task {id}: {title}

## Description
{detailed description}

## Implementation Details
{what needs to be implemented}

## Acceptance Criteria
{criteria from prd.json with more detail}

## Browser Test Instructions
{if applicable, step-by-step browser testing}

## JavaScript Verification
```javascript
// Commands that return true/false for pass/fail
```
```

## Validation Checklist

Before considering setup complete:

- [ ] `prd.json` exists and is valid JSON
- [ ] All task IDs are unique (BP-* or PAR-*)
- [ ] `blockedBy` arrays reference valid task IDs
- [ ] No circular dependencies
- [ ] All `acceptanceCriteria` are testable (not vague)
- [ ] `progress.txt` initialized
- [ ] Base plate tasks identified correctly
- [ ] Parallel tasks have correct `blockedBy`
- [ ] Test skill created at `.claude/skills/test-{project}/SKILL.md`
- [ ] `config.testing.skill` references the test skill

## prd.json Quick Reference

### Task ID Prefixes

| Prefix | Meaning | Execution |
|--------|---------|-----------|
| `BP-*` | Base Plate | Sequential, in priority order |
| `PAR-*` | Parallel | Concurrent after all blockers pass |

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier |
| `title` | string | Short descriptive title |
| `description` | string | Detailed implementation notes |
| `acceptanceCriteria` | string[] | Testable conditions |
| `priority` | number | 1 = highest priority |
| `passes` | boolean | Always start as `false` |
| `blockedBy` | string[] | Task IDs that must complete first |
| `isBasePlate` | boolean | `true` for sequential, `false` for parallel |

### Acceptance Criteria Best Practices

**Good (testable):**
- "File js/config.js exists with SPEED constant"
- "Build passes: npm run build exits with code 0"
- "Player visible: game.scene.scenes[0].player !== undefined"

**Bad (vague):**
- "Works correctly"
- "Good performance"
- "User-friendly interface"

## Example: API Service

```json
{
  "project": "user-api",
  "branchName": "feature/user-api",
  "description": "Implements user management API endpoints",
  "config": {
    "testing": { "mode": "unit", "skill": "/test-user-api" },
    "build": { "command": "npm run build", "testCommand": "npm test" }
  },
  "userStories": [
    {
      "id": "BP-001",
      "title": "User model and schema",
      "description": "Create User model with validation",
      "acceptanceCriteria": [
        "src/models/User.ts exists",
        "User has id, email, name fields",
        "npm run build passes"
      ],
      "priority": 1,
      "passes": false,
      "blockedBy": [],
      "isBasePlate": true
    },
    {
      "id": "PAR-001",
      "title": "GET /users endpoint",
      "description": "List all users with pagination",
      "acceptanceCriteria": [
        "GET /users returns 200",
        "Response includes users array",
        "Unit tests pass"
      ],
      "priority": 2,
      "passes": false,
      "blockedBy": ["BP-001"],
      "isBasePlate": false
    }
  ]
}
```

## Example: Web Application

```json
{
  "project": "dashboard",
  "branchName": "feature/dashboard",
  "description": "Implements admin dashboard with charts",
  "config": {
    "testing": {
      "mode": "browser",
      "skill": "/test-dashboard",
      "devServerUrl": "http://localhost:3000",
      "devServerCommand": "npm run dev"
    }
  },
  "userStories": [
    {
      "id": "BP-001",
      "title": "Dashboard layout",
      "description": "Create base dashboard layout with sidebar",
      "acceptanceCriteria": [
        "src/components/Dashboard.tsx exists",
        "Sidebar visible: document.querySelector('.sidebar') !== null",
        "No console errors"
      ],
      "priority": 1,
      "passes": false,
      "blockedBy": [],
      "isBasePlate": true
    }
  ]
}
```

## Example: CLI Tool

```json
{
  "project": "cli-tool",
  "branchName": "feature/cli-tool",
  "description": "Implements command-line interface",
  "config": {
    "testing": { "mode": "integration", "skill": "/test-cli-tool" },
    "build": { "command": "go build" }
  },
  "userStories": [
    {
      "id": "BP-001",
      "title": "CLI framework setup",
      "description": "Set up cobra CLI with root command",
      "acceptanceCriteria": [
        "cmd/root.go exists",
        "./cli --help shows usage",
        "go build succeeds"
      ],
      "priority": 1,
      "passes": false,
      "blockedBy": [],
      "isBasePlate": true
    }
  ]
}
```

## Handoff to Coordinator

After setup is complete, tell the user:

```
Project setup complete!

Created:
- {plan_folder}/prd.json (X tasks: Y base plate, Z parallel)
- {plan_folder}/progress.txt
- {plan_folder}/specs/ (if applicable)
- .claude/skills/test-{project}/SKILL.md

Testing mode: {config.testing.mode}
Test skill: /test-{project}

To execute this plan, run:
/coordinate {plan_folder}
```

**Add mode-specific prerequisites:**

| Mode | Prerequisites |
|------|---------------|
| `browser` | Chrome with Claude extension active, dev server running at {devServerUrl} |
| `unit` | Test framework installed |
| `integration` | Services running (if applicable) |
| `manual` | None (user will verify manually) |
