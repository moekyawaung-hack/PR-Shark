Great — here are **2, 3, and 4**:

- **2. Real GitHub Projects setup for a software team**
- **3. Working GitHub GraphQL/API example**
- **4. GitHub Actions automation example for Projects**

---

# 2. Real GitHub Projects setup for a software team

Let’s design a practical GitHub Project for a typical software team.

## Scenario
Team:
- 5 developers
- 1 QA
- 1 product manager

Repositories
- `frontend-app`
- `backend-api`
- `infra`
- `mobile-app`

Goals:
- Manage sprint work
- Track bugs/features
- Maintain roadmap visibility
- Keep planning close to cod
---

## Recommended project structure

### Project name
**Engineering Delivery**

### Core fields

#### 1. Status
Type: **Single select**

Options:
- Backlog
- Ready
- In Progress
- In Review
- Blocked
- Done

#### 2. Priority
Type: **Single select**

Options:
- P0
- P1
- P2
- P3

Meaning:
- **P0** = urgent production issue
- **P1** = high-impact
- **P2** = normal
- **P3** = low priority

#### 3. Iteration
Type: **Iteration**

Examples:
- Sprint 12
- Sprint 13
- Sprint 14

#### 4. Estimate
Type: **Number** or single select

Examples:
- 1
- 2
- 3
- 5
- 8

#### 5. Team
Type: **Single select**

Options:
- Frontend
- Backend
- Mobile
- Platform
- QA

#### 6. Target date
Type: **Date**

#### 7. Effort / Risk Notes
Type: **Text**

Useful for small comments like:
- waiting on vendor
- API design incomplete
- blocked by migration

---

## Suggested views

### A. Sprint Board
Layout: **Board**
Group by: **Status**
Filter:
- `Iteration = current sprint`

Purpose:
- Daily standups
- Track progress in sprint

---

### B. Backlog
Layout: **Table**
Filter:
- `Status = Backlog` or `Status = Ready`

Sort by:
1. Priority
2. Target date

Purpose:
- PM and leads refine work
- Prepare next sprint

---

### C. Roadmap
Layout: **Roadmap**
Use:
- Target date
- Maybe group by Team

Purpose:
- Leadership/product visibility
- Release planning

---

### D. Bugs Only
Layout: **Table** or **Board**
Filter:
- label = `bug`

Sort by:
- Priority descending

Purpose:
- triage and production support

---

### E. My Work
Layout: **Table**
Filter:
- `assignee = @me`
- `Status != Done`

Purpose:
- personal work queue

---

### F. Blocked Items
Layout: **Table**
Filter:
- `Status = Blocked`

Purpose:
- surface impediments quickly

---

## Repository label strategy

Use repository labels for issue classification.

Recommended labels:
- `bug`
- `feature`
- `tech-debt`
- `documentation`
- `incident`
- `enhancement`
- `backend`
- `frontend`
- `mobile`
- `infra`

### Why use labels?
Labels are best for:
- issue category
- technical area
- triage tags

Project fields are best for:
- workflow state
- sprint
- priority
- scheduling

---

## Example issue lifecycle

### Feature request
1. PM creates issue:
   - title: `Add export to CSV for reports`
   - labels: `feature`, `frontend`
2. Issue auto-added to project
3. Status defaults to **Backlog**
4. During planning:
   - Priority = P2
   - Team = Frontend
   - Iteration = Sprint 14
   - Estimate = 3
5. When work starts:
   - Status = In Progress
6. PR opened:
   - linked to issue
7. Code review:
   - Status = In Review
8. PR merged and issue closed:
   - Status automatically becomes Done

---

## Example sprint workflow

### During backlog refinement
- Review all Backlog items
- Remove duplicates
- Clarify acceptance criteria
- Set Priority and Estimate
- Add Team field
- Move refined items to Ready

### Sprint planning
- Pick Ready items
- Assign Iteration
- Confirm assignee
- Move sprint items into current sprint view

### During development
- Developers move items:
  - Ready → In Progress
  - In Progress → In Review
  - In Review → Done

### During blockers
- Move items to Blocked
- Add note in comments or risk field

### At sprint end
- Review incomplete items
- Move to next iteration if needed
- Archive completed items later

---

## Good team conventions

### Definition of each status
- **Backlog**: captured but not yet ready
- **Ready**: refined and can be picked up
- **In Progress**: active implementation
- **In Review**: PR or QA review happening
- **Blocked**: cannot progress
- **Done**: completed and accepted

### Rules
- Every sprint item must have:
  - assignee
  - priority
  - iteration
- No item stays in **In Review** without a PR link
- Blocked items must have a comment explaining the blocker
- P0/P1 items must have target dates

---

## Minimal starter version
If you want to keep it simple, use only:
- Status
- Priority
- Iteration

Views:
- Board
- Backlog
- My Work

That’s often enough for small teams.

---

# 3. Working GitHub GraphQL/API example

GitHub Projects v2 is best handled through the **GraphQL API**.

Below is a realistic example flow:

## Goal
We want to:
1. Find an organization project
2. Add an issue to the project
3. Update its Status field

---

## Prerequisites

You need:
- a GitHub token with proper permissions
- project access
- issue/repository access

Usually you’ll use a **GitHub personal access token** or **GitHub App** token.

---

## Example 1: Get organization projects

### GraphQL query
```graphql
query {
  organization(login: "my-org") {
    projectsV2(first: 10) {
      nodes {
        id
        title
        number
        url
      }
    }
  }
}
```

### cURL example
```bash
curl -X POST https://api.github.com/graphql \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "query { organization(login: \"my-org\") { projectsV2(first: 10) { nodes { id title number url } } } }"}'
```

### Example response
```json
{
  "data": {
    "organization": {
      "projectsV2": {
        "nodes": [
          {
            "id": "PVT_kwDOAExample1",
            "title": "Engineering Delivery",
            "number": 3,
            "url": "https://github.com/orgs/my-org/projects/3"
          }
        ]
      }
    }
  }
}
```

Save the project `id`.

---

## Example 2: Get issue node ID

If you know the repository and issue number, query it:

```graphql
query {
  repository(owner: "my-org", name: "backend-api") {
    issue(number: 123) {
      id
      title
      url
    }
  }
}
```

### cURL
```bash
curl -X POST https://api.github.com/graphql \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "query { repository(owner: \"my-org\", name: \"backend-api\") { issue(number: 123) { id title url } } }"}'
```

Example response:
```json
{
  "data": {
    "repository": {
      "issue": {
        "id": "I_kwDOAExample2",
        "title": "Add audit log endpoint",
        "url": "https://github.com/my-org/backend-api/issues/123"
      }
    }
  }
}
```

Save the issue `id`.

---

## Example 3: Add issue to project

### Mutation
```graphql
mutation {
  addProjectV2ItemById(input: {
    projectId: "PVT_kwDOAExample1",
    contentId: "I_kwDOAExample2"
  }) {
    item {
      id
    }
  }
}
```

### cURL
```bash
curl -X POST https://api.github.com/graphql \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "mutation { addProjectV2ItemById(input: { projectId: \"PVT_kw
