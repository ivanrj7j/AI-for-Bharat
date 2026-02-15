# REQUIREMENTS DOCUMENT

## AI Team Formation Assistant (Hackathon MVP)

---

## 1. Purpose

Define the functional and non-functional requirements for a hackathon-scale system that helps a user:

* describe a project idea
* receive an AI-generated execution roadmap
* view recommended people for required roles
* select a team while remaining fully in control

The system removes the **hassle of finding the right collaborators**, without automating hiring or execution.

---

## 2. Scope

### Included in MVP

* Free-form project idea input
* AI-generated:

    * project summary
    * feature list
    * required roles
    * suggested tech stack
    * rough timeline
* Recommended candidate list per role
* Human team selection interface
* Single-page visual dashboard
* Clear end-to-end **UX flow from idea → team**

### Explicitly Excluded

* Real hiring or contracts
* Payments or escrow
* Messaging or collaboration tools
* Live scraping of freelance platforms
* Full project management system

---

## 3. Functional Requirements

### 3.1 Idea Submission

The system shall:

* Allow textual project description input
* Validate non-empty input
* Send idea to backend API

---

### 3.2 AI Roadmap Generation

The backend shall:

* Send idea to an external language-model inference service

* Generate structured output:

    * summary
    * features
    * roles
    * tech stack
    * phases

* Return **structured JSON**

---

### 3.3 Candidate Recommendation

For each role, the system shall:

* Display **3–5 suggested candidates**
* Show:

    * skills
    * experience indicator
    * cost estimate
    * match score

Data may be **mock or public**.
Real scraping is not required.

---

### 3.4 Team Selection

User shall:

* Add/remove candidates
* View final **selected team**
* Retain full control

AI only **recommends**, never auto-assigns.

---

### 3.5 Dashboard Visualization

Single-screen display must include:

* Project overview
* Feature roadmap
* Required roles
* Recommended candidates
* Selected team

---

## 4. UX Flow Requirements

### Step-by-Step User Journey

1. **Landing / Input**

    * User enters project idea
    * Clicks *Generate*

2. **Processing State**

    * Loading indicator shows AI thinking
    * Prevents duplicate submissions

3. **Results Dashboard**

    * Roadmap appears first
    * Roles displayed next
    * Candidate cards shown per role

4. **Team Selection**

    * User clicks **Add to Team**
    * Selected team panel updates live

5. **Completion State**

    * User sees final starter team
    * Ready for real-world execution outside MVP

Total interaction steps ≤ **3 major actions**.

---

## 5. Non-Functional Requirements

### Performance

* Generation ≤ **60 seconds**
* UI response ≤ **2 seconds**

### Reliability

* Graceful AI/network failure handling
* Clear error messages
* No crashes

### Usability

* Minimal learning curve
* Clear visual hierarchy
* Guided linear UX flow

### Security (Hackathon Level)

* No sensitive data storage
* API keys via environment variables
* Basic validation

### Deployability

* Container-ready backend
* Cloud-compatible configuration

---

## 6. Acceptance Criteria

A user can:

1. Enter an idea
2. Receive roadmap
3. View recommended people
4. Select a team

All within **one minute**.

---

**End of Requirements Document**
