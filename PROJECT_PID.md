# Project Initiation Document (PID)
## Agentic Software Delivery System (ASDS)

Status: Draft v1  
Source: Simplified from the current [README.md](/Users/Projects/Smart-Dev-Builder/README.md)  
Document goal: convert the current concept into a modular, execution-ready PID

## 1. Executive Summary

ASDS is a software delivery system that converts approved requirements into code changes, tests, pull requests, and delivery reports with controlled use of AI agents.

This PID intentionally keeps the first version simple:

- one orchestrator
- independent agents with single responsibilities
- clear input/output contracts
- human approval at key gates
- modular rollout in phases

## 2. What This Project Is Solving

Teams spend time translating requirements into implementation plans, code changes, tests, reviews, and pull requests. ASDS aims to reduce that manual effort while still keeping control, quality, and auditability.

## 3. Key Corrections to the Current Draft

The current `README.md` is useful, but it is closer to a concept note or high-level PID than a true PRD. The following corrections are applied in this PID:

- reduce agent count for MVP instead of starting with many specialized agents
- make human approval explicit instead of assuming full autonomy
- define modules and agent boundaries separately
- keep RAG, vector DB, and advanced optimization optional for early phases
- replace broad goals with clearer scope and measurable outcomes
- require all agents to run independently through the orchestrator only

## 4. Project Scope

### In Scope for MVP

- intake a structured requirement from a defined template
- validate requirement quality before execution
- generate an implementation plan
- retrieve relevant repository context
- create code changes in a target repository
- generate and run tests
- perform basic code review and security checks
- create a branch, commit changes, and prepare a pull request
- generate a delivery report for each run

### Out of Scope for MVP

- production deployment automation
- cross-repository dependency orchestration
- autonomous release approvals
- support for every programming language
- advanced multi-model routing
- fully automatic architecture redesign

## 5. Target Users

- Product or delivery manager: submits approved requirements
- Engineering lead: reviews plan, approves execution, reviews PR
- Developer platform owner: manages integrations, policies, and system settings

## 6. Success Criteria

The MVP is successful when it can:

- process a valid requirement from intake to pull request
- create a reproducible execution log for each run
- pass configured tests before PR creation
- stop safely when requirements are unclear or quality checks fail
- reduce manual effort for small to medium feature work

Suggested measurable targets for MVP:

- 80 percent of valid stories complete the workflow without manual rework
- 90 percent of generated PRs include passing automated tests
- average execution time under 30 minutes for a small feature
- less than 3 orchestrator retries per workflow stage on average

## 7. Guiding Principles

- Keep the first release small and reliable
- Prefer explicit contracts over agent-to-agent discussion
- Keep agents independent and stateless where possible
- Use one orchestrator as the only control point
- Add complexity only after the basic workflow is stable

## 8. Modular Architecture

| Module | Purpose | Input | Output | Depends On |
|---|---|---|---|---|
| Requirement Intake | Accept and normalize requirement data | Story template, metadata | Structured requirement record | None |
| Requirement Quality Gate | Check clarity, scope, completeness | Structured requirement | Approved or rejected requirement | Requirement Intake |
| Planning | Break work into steps and file-level tasks | Approved requirement | Execution plan | Requirement Quality Gate |
| Context Retrieval | Fetch relevant code, docs, and history | Execution plan, repo info | Context package | Planning |
| Code Execution | Apply code changes | Plan, context package | Code diff | Context Retrieval |
| Quality and Test | Review code, generate tests, run checks | Code diff, acceptance criteria | Test results, findings, validation result | Code Execution |
| Git and PR | Create branch, commit, PR draft | Validated diff, metadata | PR link, commit info | Quality and Test |
| Reporting | Summarize result and artifacts | Workflow outputs | Final report | Git and PR |
| Orchestrator | Manage order, retries, state, audit trail | Workflow state | Routed tasks and final status | All modules |

## 9. Independent Agent Model

All agents must be independently executable. No agent should depend on direct conversation with another agent. The orchestrator passes only the required inputs and collects the outputs.

### Agent Independence Rules

- every agent has one responsibility
- every agent accepts a defined input contract
- every agent returns a defined output contract
- no direct agent-to-agent messaging
- retries happen at agent level, controlled by the orchestrator
- agents should be replaceable without changing the full system

### MVP Agent Set

| Agent | Responsibility | Input | Output |
|---|---|---|---|
| Requirement Parser Agent | Convert raw input into the standard schema | Raw story or form data | Structured requirement |
| Requirement Reviewer Agent | Score clarity and completeness | Structured requirement | Approval status, missing items |
| Planner Agent | Create task breakdown and execution order | Approved requirement | Execution plan |
| Context Agent | Gather relevant files, docs, and repo history | Execution plan, repo | Context package |
| Code Agent | Implement code changes | Plan, context package | Code diff |
| Quality Agent | Perform review, basic security checks, and standards checks | Code diff | Findings, pass or fail |
| Test Agent | Generate tests and run configured checks | Code diff, acceptance criteria | Test results |
| PR Agent | Create branch, commit, and PR draft | Validated changes | PR metadata |
| Report Agent | Produce human-readable summary | Full run artifacts | Delivery report |

Note: For MVP, `Architecture Agent`, `Refactor Agent`, `Security Agent`, and `Commit Message Agent` should not be separate services. Their responsibilities can be absorbed into `Planner Agent`, `Code Agent`, `Quality Agent`, and `PR Agent` to keep the system simpler.

## 10. Requirement Input Standard

Every incoming feature request should follow this structure:

```yaml
feature_title:
business_goal:
user_story:
functional_requirements:
non_functional_requirements:
acceptance_criteria:
api_or_data_changes:
edge_cases:
dependencies:
out_of_scope:
definition_of_done:
```

Minimum rules:

- acceptance criteria must be testable
- out of scope must be explicit
- dependencies must be named
- edge cases must be listed
- definition of done must include validation expectations

## 11. Workflow

1. Intake requirement
2. Parse into standard schema
3. Review completeness and block low-quality input
4. Request human approval
5. Build implementation plan
6. Retrieve relevant repository context
7. Generate code changes
8. Review code and run tests
9. Validate final output
10. Create branch, commit, and PR draft
11. Publish delivery report

## 12. Orchestration Design

The orchestrator is the only component allowed to:

- start agents
- pass data between agents
- persist workflow state
- trigger retries
- stop a failed workflow
- escalate to a human reviewer

Recommended workflow state:

```json
{
  "run_id": "uuid",
  "current_stage": "quality_and_test",
  "status": "in_progress",
  "retry_count": 1,
  "artifacts": [],
  "errors": []
}
```

Retry policy:

- maximum 3 retries per stage
- retry only on recoverable failures
- escalate to human review after retry limit
- do not continue to PR if validation fails

## 13. Recommended MVP Technology Choices

Keep the first implementation simple and opinionated.

- Backend API: FastAPI
- Orchestrator: Python service with explicit workflow state machine
- Queue: Redis-backed task queue
- State store: PostgreSQL
- Repository integration: GitHub App or scoped GitHub token
- Execution environment: isolated workspace or container sandbox
- LLM provider: choose one provider for MVP
- Retrieval: start with file search and git history; add vector DB later only if needed

Recommendation: do not begin with multiple agent frameworks and multiple LLM providers. That will increase failure modes early. Start with one orchestration pattern and one model provider.

## 14. Delivery Phases

### Phase 0: Definition and Contracts

- finalize input schema
- define agent input/output contracts
- define approval gates
- choose the supported repository type for MVP

### Phase 1: Core Execution Path

- build intake, parser, reviewer, planner, and code agent flow
- support one repository type
- produce code diff and execution logs

### Phase 2: Quality Path

- add quality agent and test agent
- add retry handling
- block PR creation on failed checks

### Phase 3: Git and Reporting

- add branch creation, commit creation, PR draft generation
- add final delivery report

### Phase 4: Optimization

- add improved retrieval
- add cost controls
- add metrics dashboard

## 15. Gaps and Missing Information

The current project definition still needs the following decisions before build starts:

### Product and Scope Gaps

- What is the exact first use case: bug fix, CRUD feature, API endpoint, UI task, or full-stack story?
- Which repository type is supported first: backend only, frontend only, or full-stack?
- What programming languages and frameworks are in scope for MVP?
- Is the goal internal platform tooling or a product for external customers?

### Requirement and Approval Gaps

- Who is allowed to submit requirements?
- Who approves the requirement before execution starts?
- Who approves the PR before merge?
- What happens when a requirement is incomplete: block, ask questions, or create draft plan only?

### Technical Gaps

- How will the system access the target repository securely?
- What level of sandboxing is required for code execution?
- What tools are allowed during execution: terminal, package manager, test runner, internet access?
- What test frameworks must be supported first?
- How should secrets be injected for repository and CI integration?

### Quality and Risk Gaps

- What are the minimum quality gates before PR creation?
- What security checks are mandatory in MVP?
- What should happen when tests are flaky?
- What is the rollback or cleanup policy for failed branches and partial runs?

### Metrics and Business Gaps

- What is the acceptable cost per execution?
- What is the target execution time?
- What percentage of tasks should complete without human fixes?
- How will success be measured against current manual delivery work?

## 16. Immediate Next Decisions

Before implementation, the team should lock these items:

1. choose one repository type for MVP
2. choose one language and framework set for MVP
3. choose one LLM provider for MVP
4. define the approval flow
5. define the minimum quality gates
6. define sandbox and repository access policy

## 17. Final Recommendation

This project is viable, but the current document is too broad for direct execution. The best path is to treat the system as a modular workflow with independent agents, narrow the MVP, and delay advanced capabilities until the basic requirement-to-PR flow is stable.
