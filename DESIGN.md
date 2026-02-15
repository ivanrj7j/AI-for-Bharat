# Design

![Mockup1](preview/diag1.png)
![Mockup2](preview/diag2.png)
![Mockup3](preview/diag3.png)

## System Architecture

The platform follows a modular, service-oriented architecture combining AI orchestration, automation agents, and core project management services.

### High-Level Layers

* **Frontend Layer**
  Interfaces for founders, freelancers, and admins built with a reactive web framework.

* **Backend API Layer**
  Handles authentication, project management, sprint logic, reporting, and integrations.

* **AI Orchestration Layer**
  Responsible for idea analysis, task decomposition, candidate matching, and decision support.

* **Automation Agent Layer**
  Browser-based agents that interact with external freelance platforms for discovery and communication.

* **Data Layer**
  Persistent storage for users, projects, tasks, candidates, sprints, reports, and payments.

---

## Core Components

### Idea Intake & Feasibility Engine

**Purpose:** Convert founder input into structured project understanding.
**Inputs:** Founder idea, optional constraints, budget, timeline.
**Outputs:**

* Feasibility score
* Risk indicators
* High-level architecture outline

**Design Notes:**

* Uses LLM reasoning with validation rules.
* Stores versioned analysis for traceability.

---

### Task Breakdown Engine

**Purpose:** Transform validated ideas into executable work units.
**Outputs:**

* Hierarchical task tree
* Dependencies
* Estimated effort
* Required roles

**Design Notes:**

* Combines prompt-driven AI with rule-based normalization.
* Produces sprint-ready backlog.

---

### Candidate Discovery Agent

**Purpose:** Identify and contact suitable freelancers automatically.

**Subcomponents:**

* Platform crawler (Playwright automation)
* Profile parser and scorer
* Messaging automation
* Shortlist generator

**Flow:**

1. Receive required roles and skills.
2. Search external platforms.
3. Score candidates via AI ranking.
4. Send outreach messages.
5. Return shortlist for approval.

---

### Human Approval Interface

**Purpose:** Keep founders in control of hiring decisions.

**Capabilities:**

* View ranked candidates
* Inspect profiles and history
* Approve or reject
* Trigger recruitment finalization

---

### Sprint Management Engine

**Purpose:** Coordinate execution after hiring.

**Responsibilities:**

* Convert backlog into sprints
* Assign tasks to freelancers
* Track status, submissions, and revisions
* Maintain timeline and blockers

**State Model:**

* Planned → Active → Review → Completed

---

### Progress Evaluation & Reporting

**Purpose:** Provide transparency and accountability.

**Generated Artifacts:**

* Sprint summary
* Completion metrics
* Budget consumption
* Remaining workload

**Design Notes:**

* Reports are immutable once finalized.
* Historical comparison across sprints supported.

---

### Payment Management System

**Purpose:** Ensure fair and traceable compensation.

**Functions:**

* Track earnings per task and sprint
* Maintain escrow-style allocation (logical or integrated)
* Release payments after approval
* Provide transaction history

---

### Authentication & Authorization

**Roles:**

* Founder
* Freelancer
* Admin

**Design:**

* Token-based authentication with refresh flow.
* Role-based access control at API and UI levels.

---

## Data Model Overview

### Core Entities

* **User** (role, profile, credentials)
* **Project** (idea, feasibility, status, budget)
* **Task** (description, role, estimate, dependency, status)
* **Sprint** (timebox, tasks, progress, report)
* **Candidate** (source platform, score, approval state)
* **Payment** (amount, milestone, release status)
* **Report** (metrics, cost, summary)

### Relationships

* A **Founder** owns multiple **Projects**.
* A **Project** contains many **Tasks** and **Sprints**.
* **Tasks** are assigned to **Freelancers**.
* **Sprints** generate **Reports** and **Payments**.

---

## External Integrations

* Freelance platforms via browser automation.
* Payment gateway for fund transfer.
* LLM provider for reasoning and generation.

---

## Security Design

* Encrypted token storage and transmission.
* Scoped automation credentials.
* Audit logs for AI decisions, hiring, and payments.

---

## Scalability Strategy

* Stateless backend services.
* Queue-based agent execution.
* Horizontal scaling for AI and automation workers.
* Separate reporting and analytics storage if needed.

---

## Future Design Extensions

* Multi-agent collaboration planning.
* Autonomous code generation and validation loops.
* Continuous deployment orchestration.
* Marketplace abstraction across many hiring sources.