# DESIGN DOCUMENT

## AI Team Formation Assistant (Hackathon MVP)

---

## 1. Objective

Design a **hackathon-ready web system** that converts a project idea into:

* structured execution roadmap
* recommended collaborators
* human-selected starter team

AI assists **planning and discovery**, while humans retain **control**.

Constraint: compelling demo within **48 hours**.

---

## 2. High-Level Architecture

### Components

**Frontend**

* Idea input screen
* Roadmap & roles dashboard
* Candidate recommendation UI
* Team selection panel

**Backend (Python FastAPI)**

* REST endpoints
* Prompt orchestration
* Structured validation
* Candidate data service

**External AI Service**

* Generates roadmap reasoning

**Cloud Layer (Optional)**

* Container deployment
* Environment configuration

---

## 3. End-to-End UX Flow

### Screen 1 — Idea Input

* Multiline text box
* Primary **Generate** action
* Clean, distraction-free layout

**Transition:** submit → loading state.

---

### Screen 2 — Processing State

* Centered loader / progress message
* Prevent repeated submissions
* Duration target: **< 60s**

**Transition:** AI response → dashboard.

---

### Screen 3 — Results Dashboard (Single Page)

#### Section Order

1. **Project Overview**
2. **Feature Roadmap**
3. **Required Roles**
4. **Recommended Candidates**
5. **Selected Team Panel**

All visible **without navigation**.

---

### Screen 4 — Team Assembly Interaction

* Candidate cards include:

    * skills
    * match score
    * cost estimate
    * **Add to Team** button

* Selected team updates **in real time**.

End state:

> User clearly sees a **complete starter team**.

---

## 4. Backend Design

### Modules

* **api/** → routes
* **services/** → AI + matching
* **schemas/** → Pydantic models
* **data/** → mock candidates
* **utils/** → helpers & errors

### Endpoint

`POST /generate`

**Input**

```
{ "idea": "string" }
```

**Output**

* summary
* features
* roles
* tech stack
* phases
* candidates per role

---

## 5. Data Strategy

* Stateless MVP
* No required database
* Static JSON candidate pool

Chosen for **speed + reliability**.

---

## 6. Error Handling

* Catch AI failures
* Structured error response
* UI retry option

System must **fail gracefully**.

---

## 7. Design Principles

* Simplicity over completeness
* Human control over AI autonomy
* Linear, obvious UX flow
* Strong visual clarity for judges
* Fast deployment readiness

---

## 8. Future Extensions

* Persistent projects
* Real freelancer onboarding
* Messaging & payments
* Advanced AI skill matching
* Full project execution layer

---

## 9. Success Criteria

A judge can:

1. Enter an idea
2. Instantly understand roadmap
3. See suggested collaborators
4. Watch human team selection

Within **60 seconds**.

---

**End of Design Document**
