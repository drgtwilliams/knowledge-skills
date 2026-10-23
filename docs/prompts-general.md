# General Engineering Prompts — Architecture, Testing, DevOps & More

> Cross-language prompts for architecture design, debugging, testing, DevOps, code review, and documentation. All built on the [CRTSE framework](best-practices.md).

---

## Architecture & System Design

### New service architecture review
```
Act as a principal software architect with 20 years of experience in distributed systems.

I'm designing a new notification service for our platform. Here's the context:

- 10M+ users, peak 50K notifications/second
- Channels: push (FCM/APNs), email (SendGrid), SMS (Twilio), in-app (WebSocket)
- Requirements: at-least-once delivery, per-user channel preferences, rate limiting (max 5/day per channel), 99.95% delivery SLA
- The rest of our stack: Kubernetes (EKS), Kafka for event streaming, PostgreSQL for transactional data, Redis for caching

Task: Review this proposed architecture and identify:

1. Single points of failure
2. Scalability bottlenecks at 10x current load
3. Data consistency risks (duplicate sends, lost notifications)
4. Missing observability (what we should monitor/alert on)
5. Cost optimization opportunities

Proposed architecture:
- REST API accepts notification requests, writes to PostgreSQL `notifications` table
- A worker pool polls PostgreSQL every 5 seconds for pending notifications
- Worker calls provider APIs directly (FCM, SendGrid, Twilio, WebSocket)
- Status updated back to PostgreSQL

Output: a structured review with specific recommendations for each of the 5 dimensions, ranked by severity.
```

### Database schema design
```
Act as a senior database architect with expertise in PostgreSQL.

I'm building a multi-tenant SaaS project management tool.

Task: Design the database schema for workspaces, projects, tasks, and users.

Requirements:
- Multi-tenant: each workspace is isolated (no cross-tenant data leaks — enforced at the schema/RLS level)
- Projects belong to a workspace; tasks belong to a project
- Tasks have: title, description, status (enum: backlog, todo, in_progress, review, done), priority (p0-p4), assignee, due date, estimated hours, labels (many-to-many)
- Users belong to workspaces with roles (owner, admin, member, viewer)
- Full audit trail: every task status change logged with who changed it, when, and the old/new values
- Soft deletes: tasks can be archived (not permanently deleted) by workspace admins
- Indexes for the top 5 query patterns: (1) tasks by status in a project, (2) tasks assigned to a user across all projects, (3) overdue tasks, (4) tasks by label, (5) workspace audit log by date range

Do NOT:
- Use a single `settings` JSONB column for all workspace config (normalize)
- Use polymorphic associations
- Skip foreign key constraints

Output: the full DDL with table definitions, constraints, indexes, and RLS policies.
```

---

## Debugging & Root Cause Analysis

### Structured debug prompt (the most important prompt in the library)
```
Act as a senior software engineer debugging a production issue. Work through this systematically. Do not guess — explain your reasoning at each step.

I'm debugging [problem description] in my [tech stack] application.

Error/behavior:
[paste the exact error message, stack trace, or behavior description]

What I've tried:
[list what you've already eliminated — prevents the LLM from suggesting solved branches]

Relevant code:
[paste the minimal reproducing code — not the whole file, just the relevant function(s) and types]

Environment:
- Language/runtime version: [e.g., Python 3.12.4, Node 22.5, Java 21.0.3]
- Framework/library versions: [e.g., FastAPI 0.115.2, React 19.0, Spring Boot 3.4]
- OS: [e.g., Linux x86_64, macOS 15, Windows 11]
- Recent changes: [what changed that might correlate — deployment, dependency bump, traffic spike]

I need you to:
1. State the most likely root cause and explain WHY it's the most likely (not the only possibility)
2. List 2-3 alternative causes with their probability
3. For the top cause, suggest a specific fix with the exact code change
4. Suggest a test that would have caught this before production
5. Suggest a monitoring alert that would detect this if it happens again

Do NOT: suggest generic debugging steps (restart, check logs, update dependencies). Be specific to the error and code shown.
```

---

## Testing

### Integration test for a microservice boundary
```
Act as a senior QA engineer.

I have two microservices: OrderService (places orders) and InventoryService (reserves stock). They communicate via REST. I need integration tests that verify the happy path and failure modes at the boundary.

Task: Write integration tests for the order placement flow:
- Happy path: item in stock → order placed → inventory decremented
- Out of stock: InventoryService returns 409 → OrderService returns appropriate error
- Timeout: InventoryService is slow (>3s) → OrderService returns 503 with retry-after header
- Partial failure: multi-item order where one item is out of stock → entire order rejected (no partial decrement)
- Idempotency: same order ID submitted twice → inventory only decremented once

Requirements:
- Language/framework: [Python + pytest, Java + JUnit 5 + TestContainers, C# + xUnit + WireMock]
- Both services run in Docker via TestContainers (or docker-compose for the test)
- InventoryService is mocked at HTTP level (WireMock / responses / MockServer)
- Each test is independent (order IDs are unique per test)
- Tests verify BOTH the HTTP response and the side effect (inventory was/wasn't decremented)

Do NOT:
- Test through the real InventoryService (unit test with mocks at boundary)
- Share mutable state between tests

Output: the complete test class/file.
```

### Property-based test for a parser/serializer
```
Act as a senior QA engineer experienced in property-based testing.

I have a `serialize_user(user: User) -> bytes` and `deserialize_user(data: bytes) -> User` pair for a binary protocol.

Task: Write property-based tests that verify:
- Round-trip: `deserialize(serialize(user)) == user` for all valid Users
- Serialized output is deterministic (same user → same bytes)
- Deserialize rejects invalid data cleanly (no panics/exceptions leaking — returns a Result/error type)
- Edge cases: empty strings, max-length strings (255 chars for username), zero values, extreme dates (year 1, year 9999)

Requirements:
- Language: [Python + Hypothesis, Java + jqwik, C# + FsCheck]
- The User generator must cover the full valid input space
- Shrinking: if a test fails, the framework should produce the minimal failing example automatically
- Run at least 10,000 random test cases

Do NOT:
- Write example-based tests for this — the point is property-based coverage of the input space
- Use fixed seeds after initial development

Output: the test file with generators and property tests.
```

---

## DevOps & Infrastructure

### Dockerfile for a production service
```
Act as a senior DevOps engineer.

I need a production Dockerfile for our Python FastAPI service.

Task: Write a multi-stage Dockerfile that:
- Build stage: installs dependencies from requirements.txt (pinned with hashes)
- Run stage: slim base image (python:3.12-slim), copies only the built venv and application code
- Runs as non-root user (uid 1000)
- Uses Gunicorn with Uvicorn workers (4 workers, preload app, graceful timeout 30s)
- HEALTHCHECK that hits the /health endpoint
- Labels for OCI metadata (org, repo, version, commit SHA)

Requirements:
- .dockerignore: exclude __pycache__, .git, .env, tests, local configs
- Layer caching: COPY requirements.txt first → pip install → COPY app code last
- No secrets in the image (API keys come from environment/K8s secrets at runtime)
- Image size target: <200MB

Do NOT:
- Use `pip install` without `--no-cache-dir`
- Run as root in the final stage
- Clone the repo inside the Dockerfile — use the build context

Output: the Dockerfile and .dockerignore.
```

### Terraform module for a serverless function
```
Act as a senior cloud infrastructure engineer (AWS, Terraform).

I need a reusable Terraform module for deploying a Lambda function behind API Gateway.

Task: Write a Terraform module (AWS provider 5.x) that:
- Creates a Lambda function (configurable: runtime, memory, timeout, env vars)
- Creates an IAM role with least-privilege: Lambda execution + any S3/DynamoDB/SQS permissions passed as inputs
- Creates an API Gateway HTTP API (not REST API) with a single $default route
- Enables CloudWatch Logs with a 30-day retention policy
- Tags all resources with the provided tag map

Requirements:
- Input variables: function_name, runtime, handler, source_dir, memory_size (default 128), timeout (default 30), environment_variables (map, optional), additional_policy_arns (list, optional), api_stage_name (default "prod"), tags (map)
- Output values: function_arn, function_name, api_endpoint, api_id
- The Lambda zip is built from source_dir (pip install -r requirements.txt -t ./package, zip)
- Validators: runtime must be one of the supported Python runtimes; timeout must be 1-900; memory must be 128-10240

Do NOT:
- Use a single monolithic main.tf — use standard module structure (main, variables, outputs)
- Hardcode the region
- Create API Gateway REST API (v1) — use HTTP API (v2)

Output: the complete module (main.tf, variables.tf, outputs.tf, and the packaging script if needed).
```

---

## Code Review

### PR review prompt
```
Review this code change like a senior engineer who will be on-call for this service next week.

The change: [describe what the PR does at a high level]

The diff:
[paste the diff]

Review checklist — flag issues in these categories:
1. Correctness: will this do what it intends under all edge cases?
2. Security: are there injection vectors, exposed secrets, missing authz?
3. Performance: N+1 queries, unnecessary allocations, blocking the event loop, memory leaks?
4. Error handling: are failures handled gracefully or swallowed silently?
5. Testing: does the test coverage match the risk? Are edge cases tested?
6. Observability: are the right things logged at the right levels? Would you find the root cause at 2 AM?
7. Maintainability: will the next developer understand this in 6 months? Are names clear? Is the abstraction right-sized?

For each issue found:
- Quote the specific line(s)
- Explain the risk in plain language
- Suggest a concrete fix

Then: give an overall verdict: APPROVE (no blockers), COMMENT (minor suggestions), or REQUEST CHANGES (blockers found).

Do NOT: nitpick formatting the linter already catches.
```

---

## Documentation

### README generation from codebase
```
Act as a senior technical writer.

Based on this project, generate a comprehensive README.md:

[paste the project structure — file tree + key file summaries, or provide the repo URL for the AI to read]

The README must include:
- Title and one-sentence description
- Badges (license, build status, version, language)
- What problem it solves (2-3 sentences)
- Quick start (install + run in 3 commands max)
- Configuration (environment variables or config file with examples)
- API overview (if applicable — one request/response example)
- Architecture (high-level — a text diagram or component list)
- Development setup (how to contribute)
- License

Do NOT:
- Copy-paste full API documentation
- Include placeholder sections ("Coming soon")
- Use more than 500 words total (brevity over completeness)

Output: the complete README.md.
```