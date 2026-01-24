---
name: create-implementation-plan
description: Use when creating implementation plans for multi-phase projects. Ensures plans have proper base plate (sequential) vs parallel phase structure for the coordinator.
---

# Implementation Plan Creator

Creates structured implementation plans with explicit phase dependencies for the coordinator agent.

## Output Format: prd.json

**CRITICAL:** All plans MUST output a machine-readable `prd.json` file. This replaces markdown tables.

### prd.json Schema

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
      "lintCommand": "npm run lint",
      "testCommand": "npm test"
    }
  },
  "userStories": [
    {
      "id": "BP-001",
      "title": "Base plate: Core setup",
      "description": "Foundational task that others depend on",
      "acceptanceCriteria": [
        "Files created correctly",
        "Build passes"
      ],
      "priority": 1,
      "passes": false,
      "blockedBy": [],
      "isBasePlate": true
    },
    {
      "id": "PAR-001",
      "title": "Parallel: Feature A",
      "description": "Independent feature",
      "acceptanceCriteria": [
        "Feature works as specified",
        "Tests pass"
      ],
      "priority": 2,
      "passes": false,
      "blockedBy": ["BP-001"],
      "isBasePlate": false
    }
  ]
}
```

### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique task ID. Prefix: `BP-` for base plate, `PAR-` for parallel |
| `title` | string | Short descriptive title |
| `description` | string | Detailed description of what to implement |
| `acceptanceCriteria` | string[] | Testable conditions (must be verifiable by test agent) |
| `priority` | number | Execution order (1 = highest priority) |
| `passes` | boolean | Whether task has passed all tests (starts `false`) |
| `blockedBy` | string[] | Array of task IDs that must complete first |
| `isBasePlate` | boolean | `true` = sequential, `false` = parallelizable |

## CRITICAL: Base Plate Structure

Every implementation plan MUST include:

1. **Base Plate Tasks** - Foundational tasks that must complete sequentially (`isBasePlate: true`)
2. **Parallel Tasks** - Tasks that can run simultaneously after base plate (`isBasePlate: false`)

### Identifying Base Plate vs Parallel

#### A task is BASE PLATE if:
- It establishes core architecture (data models, base classes, config)
- Multiple other tasks import/extend its code
- It must exist before other tasks can compile/run
- Failure here breaks everything downstream

#### A task is PARALLEL if:
- It only depends on base plate (not other parallel tasks)
- It adds independent features/functionality
- It can be developed without knowing details of other parallel tasks
- Merge order doesn't matter (no cross-dependencies)

## CRITICAL: Every Task Must Be Testable

**No task is complete without verification.** Every `acceptanceCriteria` item MUST be:

1. **Specific** - Not vague like "works correctly"
2. **Testable** - Can be verified programmatically
3. **Observable** - Can be checked via file existence, build output, or browser test

### Criteria Types (by testing mode)

| Type | Example | Mode | How Tested |
|------|---------|------|------------|
| File existence | "src/config.ts exists" | All | Glob/Read tools |
| Build passes | "No TypeScript errors" | All | Run build command |
| Unit test | "Tests in tests/*.test.ts pass" | unit | Run test command |
| Browser state | "Modal visible after click" | browser | Chrome JS inspection |
| Visual check | "No visual glitches" | browser | Chrome screenshot |
| API response | "GET /users returns 200" | integration | curl/fetch |
| CLI output | "help command shows usage" | integration | Bash |

### Verification Commands by Mode

**For unit testing mode:**
```json
{
  "acceptanceCriteria": [
    "src/models/User.ts exists",
    "npm run build passes with no errors",
    "npm test passes with no failures"
  ]
}
```

**For browser testing mode:**
Include inline JavaScript verification:
```json
{
  "acceptanceCriteria": [
    "Component exists: document.querySelector('.modal') !== null",
    "State correct: window.appState.isLoggedIn === true",
    "No console errors"
  ]
}
```

**For integration testing mode:**
```json
{
  "acceptanceCriteria": [
    "curl -s http://localhost:3000/health returns 200",
    "./cli --help shows usage text",
    "docker ps shows container running"
  ]
}
```

**For manual testing mode:**
```json
{
  "acceptanceCriteria": [
    "User can navigate to settings page",
    "Form validation shows error on invalid email",
    "Data persists after page refresh"
  ]
}
```

## Plan Structure

When creating a plan, output:

### 1. Main prd.json file

```
{plan_folder}/prd.json
```

### 2. Individual task spec files (optional but recommended for complex tasks)

```
{plan_folder}/specs/BP-001-core-setup.md
{plan_folder}/specs/PAR-001-feature-a.md
```

### 3. Progress file (initialized empty)

```
{plan_folder}/progress.txt
```

## Workflow

1. Analyze the project requirements
2. Identify foundational work → these become base plate tasks
3. Identify independent features → these become parallel tasks
4. Assign IDs: `BP-001`, `BP-002`, ... for base plate; `PAR-001`, `PAR-002`, ... for parallel
5. Set `blockedBy` arrays (parallel tasks should block on final base plate task)
6. Write acceptance criteria that are testable
7. Output `prd.json` and optional spec files

## Validation Checklist

Before finalizing a plan, verify:

**Structure:**
- [ ] `prd.json` follows schema exactly
- [ ] All base plate tasks have `isBasePlate: true`
- [ ] All parallel tasks have `isBasePlate: false`
- [ ] `blockedBy` arrays reference valid task IDs
- [ ] No circular dependencies exist
- [ ] Task IDs are unique

**Testing (EVERY task):**
- [ ] Every `acceptanceCriteria` item is testable (not vague)
- [ ] Browser tests have JavaScript verification commands
- [ ] Build/lint criteria specify the exact command
- [ ] File existence criteria specify exact paths

**Coordinator-Ready:**
- [ ] A coordinator can execute this plan without asking questions
- [ ] Test agent can verify completion using provided criteria
- [ ] `progress.txt` is initialized

## Example: Authentication System (Unit Testing)

**prd.json:**
```json
{
  "project": "auth-system",
  "branchName": "feature/auth-system",
  "description": "Implements user authentication with JWT",
  "config": {
    "testing": { "mode": "unit" },
    "build": { "command": "npm run build", "testCommand": "npm test" }
  },
  "userStories": [
    {
      "id": "BP-001",
      "title": "Auth config and types",
      "description": "Set up auth configuration and TypeScript types",
      "acceptanceCriteria": [
        "src/config/auth.ts exists",
        "Exports JWT_SECRET, TOKEN_EXPIRY constants",
        "npm run build passes"
      ],
      "priority": 1,
      "passes": false,
      "blockedBy": [],
      "isBasePlate": true
    },
    {
      "id": "BP-002",
      "title": "Auth service",
      "description": "Create authentication service with login/logout",
      "acceptanceCriteria": [
        "src/services/AuthService.ts exists",
        "Has login(), logout(), validateToken() methods",
        "Unit tests in tests/auth.test.ts pass"
      ],
      "priority": 2,
      "passes": false,
      "blockedBy": ["BP-001"],
      "isBasePlate": true
    },
    {
      "id": "PAR-001",
      "title": "Login endpoint",
      "description": "Implement POST /auth/login",
      "acceptanceCriteria": [
        "src/routes/auth.ts has login handler",
        "Returns JWT on valid credentials",
        "Returns 401 on invalid credentials"
      ],
      "priority": 3,
      "passes": false,
      "blockedBy": ["BP-002"],
      "isBasePlate": false
    },
    {
      "id": "PAR-002",
      "title": "Auth middleware",
      "description": "JWT validation middleware",
      "acceptanceCriteria": [
        "src/middleware/auth.ts exists",
        "Validates JWT from Authorization header",
        "Sets req.user on valid token"
      ],
      "priority": 3,
      "passes": false,
      "blockedBy": ["BP-002"],
      "isBasePlate": false
    },
    {
      "id": "PAR-003",
      "title": "Password hashing",
      "description": "Secure password storage with bcrypt",
      "acceptanceCriteria": [
        "src/utils/password.ts exists",
        "hashPassword() returns bcrypt hash",
        "verifyPassword() compares correctly"
      ],
      "priority": 3,
      "passes": false,
      "blockedBy": ["BP-002"],
      "isBasePlate": false
    }
  ]
}
```

**Execution order:**
1. BP-001 (sequential - base plate)
2. BP-002 (sequential - base plate, blocked by BP-001)
3. PAR-001, PAR-002, PAR-003 (parallel - all blocked by BP-002)
