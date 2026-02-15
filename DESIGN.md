# Design Document: Team Foundry

## Overview

Team Foundry is a single-page web application that transforms project ideas into executable roadmaps with AI-recommended team candidates. The system follows a linear flow: idea input → AI analysis → roadmap generation → candidate recommendation → human team selection.

The architecture emphasizes simplicity for the hackathon MVP, using a client-server model where the frontend handles user interaction and display, while the backend orchestrates AI processing and data retrieval. The system maintains session-based state without requiring user authentication.

Key design principles:
- **Human-in-the-loop**: AI recommends, humans decide
- **Speed**: Complete flow in under 60 seconds
- **Simplicity**: Single-page experience with minimal navigation
- **Transparency**: Clear feedback at each step

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                   Frontend (React + Memstack)                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Idea Input   │  │   Roadmap    │  │   Team       │      │
│  │ Component    │→ │   Display    │→ │   Builder    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Memstack State Management               │    │
│  │  • Session state (roadmap, candidates, team)        │    │
│  │  • API integration layer                            │    │
│  │  • Client-side persistence                          │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓ HTTP/REST
┌─────────────────────────────────────────────────────────────┐
│                      Backend API Server                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Roadmap    │  │  Candidate   │  │   Session    │      │
│  │   Service    │  │   Service    │  │   Manager    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    External Dependencies                     │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │  AI/LLM API  │  │  Candidate   │                         │
│  │  (OpenAI)    │  │  Database    │                         │
│  └──────────────┘  └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

**Frontend Components (React + Memstack):**
- **Idea Input Component**: Captures project description, validates minimum length, triggers roadmap generation
- **Roadmap Display Component**: Renders structured roadmap (summary, features, roles, tech stack, phases)
- **Team Builder Component**: Displays candidate cards, handles selection/deselection, shows current team
- **Memstack State Layer**: Manages application state (roadmap, candidates, team selection), handles API calls, provides reactive state updates to components

**Backend Services:**
- **Roadmap Service**: Interfaces with AI/LLM to generate structured roadmaps from project ideas
- **Candidate Service**: Retrieves and ranks candidates from database based on required roles
- **Session Manager**: Maintains session state (roadmap, team selection) without authentication

**External Dependencies:**
- **AI/LLM API**: Generates roadmaps from natural language input (e.g., OpenAI GPT-4)
- **Candidate Database**: Stores predefined candidate profiles with skills, experience, and cost data

## Components and Interfaces

### Memstack State Management

Memstack will handle all application state and API interactions. The state structure:

```typescript
// Memstack state schema
interface AppState {
  session: {
    sessionId: string | null;
    status: 'idle' | 'generating' | 'complete' | 'error';
    error: string | null;
  };
  projectIdea: string;
  roadmap: ExecutionRoadmap | null;
  candidates: Map<string, CandidateProfile[]>;
  teamSelection: Set<string>;
}

// Memstack actions
const actions = {
  setProjectIdea: (idea: string) => void;
  submitIdea: () => Promise<void>;
  selectCandidate: (candidateId: string) => void;
  removeCandidate: (candidateId: string) => void;
  retryGeneration: () => Promise<void>;
  loadSession: (sessionId: string) => Promise<void>;
};
```

### Frontend API Client

```typescript
interface IdeaSubmissionRequest {
  projectIdea: string;
  sessionId?: string;
}

interface ExecutionRoadmap {
  projectSummary: string;
  coreFeatures: string[];
  requiredRoles: RequiredRole[];
  technologyStack: string[];
  developmentPhases: DevelopmentPhase[];
}

interface RequiredRole {
  title: string;
  description: string;
  requiredSkills: string[];
}

interface DevelopmentPhase {
  name: string;
  description: string;
  estimatedDuration: string;
}

interface CandidateProfile {
  id: string;
  name: string;
  skills: string[];
  experienceYears: number;
  costRange: CostRange;
  matchScore: number;
  bio: string;
}

interface CostRange {
  min: number;
  max: number;
  currency: string;
}

interface TeamSelection {
  sessionId: string;
  selectedCandidates: CandidateProfile[];
}
```

### Backend API Endpoints

```
POST /api/roadmap
  Request: { projectIdea: string, sessionId?: string }
  Response: { 
    sessionId: string,
    roadmap: ExecutionRoadmap,
    candidates: { [roleTitle: string]: CandidateProfile[] }
  }

POST /api/team/select
  Request: { sessionId: string, candidateId: string }
  Response: { teamSelection: TeamSelection }

POST /api/team/remove
  Request: { sessionId: string, candidateId: string }
  Response: { teamSelection: TeamSelection }

GET /api/session/:sessionId
  Response: {
    roadmap: ExecutionRoadmap,
    candidates: { [roleTitle: string]: CandidateProfile[] },
    teamSelection: TeamSelection
  }
```

### Roadmap Service Interface

```typescript
interface IRoadmapService {
  generateRoadmap(projectIdea: string): Promise<ExecutionRoadmap>;
}

class AIRoadmapService implements IRoadmapService {
  constructor(private aiClient: IAIClient) {}
  
  async generateRoadmap(projectIdea: string): Promise<ExecutionRoadmap> {
    // Construct prompt for AI
    // Call AI API
    // Parse and validate response
    // Return structured roadmap
  }
}
```

### Candidate Service Interface

```typescript
interface ICandidateService {
  findCandidatesForRole(
    role: RequiredRole,
    count: number
  ): Promise<CandidateProfile[]>;
  
  calculateMatchScore(
    candidate: CandidateProfile,
    role: RequiredRole
  ): number;
}

class CandidateService implements ICandidateService {
  constructor(private database: ICandidateDatabase) {}
  
  async findCandidatesForRole(
    role: RequiredRole,
    count: number
  ): Promise<CandidateProfile[]> {
    // Query database for candidates with matching skills
    // Calculate match scores
    // Sort by match score descending
    // Return top N candidates
  }
}
```

### Session Manager Interface

```typescript
interface ISessionManager {
  createSession(): string;
  storeRoadmap(sessionId: string, roadmap: ExecutionRoadmap): void;
  storeCandidates(sessionId: string, candidates: Map<string, CandidateProfile[]>): void;
  getSession(sessionId: string): SessionData | null;
  addToTeam(sessionId: string, candidateId: string): TeamSelection;
  removeFromTeam(sessionId: string, candidateId: string): TeamSelection;
}

interface SessionData {
  roadmap: ExecutionRoadmap;
  candidates: Map<string, CandidateProfile[]>;
  teamSelection: Set<string>;
  createdAt: Date;
}
```

## Data Models

### ExecutionRoadmap

Represents the AI-generated project plan.

```typescript
class ExecutionRoadmap {
  projectSummary: string;
  coreFeatures: string[];
  requiredRoles: RequiredRole[];
  technologyStack: string[];
  developmentPhases: DevelopmentPhase[];
  
  validate(): boolean {
    return this.projectSummary.length > 0 &&
           this.coreFeatures.length > 0 &&
           this.requiredRoles.length > 0 &&
           this.technologyStack.length > 0 &&
           this.developmentPhases.length > 0;
  }
}
```

### CandidateProfile

Represents a potential team member with skills and cost information.

```typescript
class CandidateProfile {
  id: string;
  name: string;
  skills: string[];
  experienceYears: number;
  costRange: CostRange;
  matchScore: number;
  bio: string;
  
  hasSkill(skill: string): boolean {
    return this.skills.some(s => 
      s.toLowerCase().includes(skill.toLowerCase())
    );
  }
}
```

### Match Score Calculation

The match score algorithm evaluates how well a candidate fits a required role:

```typescript
function calculateMatchScore(
  candidate: CandidateProfile,
  role: RequiredRole
): number {
  let score = 0;
  const maxScore = 100;
  
  // Skill matching (70% weight)
  const matchedSkills = role.requiredSkills.filter(skill =>
    candidate.hasSkill(skill)
  );
  const skillScore = (matchedSkills.length / role.requiredSkills.length) * 70;
  score += skillScore;
  
  // Experience bonus (30% weight)
  // 0-2 years: 0 points
  // 3-5 years: 15 points
  // 6-10 years: 25 points
  // 10+ years: 30 points
  if (candidate.experienceYears >= 10) score += 30;
  else if (candidate.experienceYears >= 6) score += 25;
  else if (candidate.experienceYears >= 3) score += 15;
  
  return Math.min(score, maxScore);
}
```

### AI Prompt Structure

The system uses a structured prompt to generate consistent roadmaps:

```
You are a project planning assistant. Analyze the following project idea and generate a structured execution roadmap.

Project Idea: {projectIdea}

Generate a JSON response with the following structure:
{
  "projectSummary": "2-3 sentence overview",
  "coreFeatures": ["feature1", "feature2", ...],
  "requiredRoles": [
    {
      "title": "Role Title",
      "description": "Role description",
      "requiredSkills": ["skill1", "skill2", ...]
    }
  ],
  "technologyStack": ["tech1", "tech2", ...],
  "developmentPhases": [
    {
      "name": "Phase Name",
      "description": "Phase description",
      "estimatedDuration": "X weeks"
    }
  ]
}

Ensure the response is valid JSON and includes at least 3 core features, 3 required roles, and 3 development phases.
```


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Input validation enforces minimum length

*For any* string input, if the length is less than 50 characters, the system should reject it with a validation message; if the length is 50 or more characters, the system should accept it and proceed to roadmap generation.

**Validates: Requirements 1.2, 1.3, 1.4**

### Property 2: Roadmap structure completeness

*For any* generated execution roadmap, it should contain all required fields: a non-empty project summary, a non-empty list of core features, a non-empty list of required roles (each with a description), a non-empty technology stack, and a non-empty list of development phases.

**Validates: Requirements 2.2, 2.3, 2.4, 2.5, 2.6**

### Property 3: Candidate recommendation count

*For any* execution roadmap with required roles, the system should recommend at least 3 candidates for each role.

**Validates: Requirements 3.1**

### Property 4: Candidate profile completeness

*For any* candidate profile, it should contain all required fields: a non-empty name, a non-empty skills list, experience information, a valid cost range (with min, max, and currency), and a match score between 0 and 100 (inclusive).

**Validates: Requirements 3.2, 3.3, 3.4, 3.5, 3.6**

### Property 5: Team selection addition

*For any* candidate and team selection state, when a candidate is selected, the resulting team selection should contain that candidate.

**Validates: Requirements 4.3**

### Property 6: Team selection removal

*For any* candidate in the team selection, when that candidate is removed, the resulting team selection should not contain that candidate.

**Validates: Requirements 4.4**

### Property 7: Team selection display consistency

*For any* team selection state, the displayed team selection should exactly match the set of selected candidates in the state.

**Validates: Requirements 4.5**

### Property 8: Roadmap display after generation

*For any* completed roadmap generation, the system should display both the roadmap and the candidate recommendations.

**Validates: Requirements 5.4**

### Property 9: Session data persistence

*For any* session with a generated roadmap and team selection modifications, retrieving the session data should return the same roadmap and team selection that were stored.

**Validates: Requirements 8.1, 8.2, 8.3**

### Property 10: Error logging without exposure

*For any* error that occurs in the system, the error should be logged for debugging, but the user-facing error message should not contain technical implementation details (such as stack traces, internal variable names, or system paths).

**Validates: Requirements 9.4**

## Error Handling

### Input Validation Errors

**Scenario**: User submits project idea with fewer than 50 characters

**Handling**:
- Display validation message: "Please provide more detail about your project (minimum 50 characters)"
- Keep user on input screen
- Preserve entered text in input field
- Highlight input field with error styling

### AI Generation Errors

**Scenario**: AI service fails to generate roadmap (timeout, API error, invalid response)

**Handling**:
- Display user-friendly message: "We couldn't generate your roadmap right now. Please try again."
- Provide retry button
- Log detailed error information server-side
- Preserve user's original input for retry

**Scenario**: AI returns malformed JSON or incomplete roadmap

**Handling**:
- Validate roadmap structure before returning to client
- If validation fails, retry generation once automatically
- If second attempt fails, show error message and allow manual retry
- Log validation failures for debugging

### Candidate Retrieval Errors

**Scenario**: Database query fails or returns insufficient candidates

**Handling**:
- If fewer than 3 candidates found for a role, return all available candidates
- Display message: "Limited candidates available for [role]. We're showing all matches."
- Log insufficient candidate scenarios for data improvement

### Session Errors

**Scenario**: Session not found or expired

**Handling**:
- Redirect user to start screen
- Display message: "Your session has expired. Please start a new project."
- Clear any stale session data from client

### Network Errors

**Scenario**: Client cannot reach backend API

**Handling**:
- Display message: "Connection error. Please check your internet connection and try again."
- Provide retry button
- Implement exponential backoff for retries (1s, 2s, 4s)

## Testing Strategy

### Unit Testing

Unit tests will focus on specific examples, edge cases, and error conditions:

**Input Validation**:
- Test exact boundary: 49 characters (invalid), 50 characters (valid)
- Test empty string
- Test very long input (10,000+ characters)

**Roadmap Parsing**:
- Test valid AI response parsing
- Test malformed JSON handling
- Test missing required fields

**Match Score Calculation**:
- Test candidate with 0 matching skills (score = 0)
- Test candidate with all matching skills and 10+ years experience (score = 100)
- Test various experience levels (0, 3, 6, 10 years)

**Session Management**:
- Test session creation
- Test session retrieval with valid ID
- Test session retrieval with invalid ID
- Test session expiration

**Team Selection**:
- Test adding first candidate to empty team
- Test adding duplicate candidate (should be idempotent)
- Test removing candidate not in team (should be no-op)

### Property-Based Testing

Property-based tests will verify universal properties across randomized inputs. Each test should run a minimum of 100 iterations.

**Testing Library**: Use `fast-check` for JavaScript/TypeScript or `hypothesis` for Python

**Property Test Configuration**:
```typescript
// Example configuration for fast-check
fc.assert(
  fc.property(
    fc.string({ minLength: 50, maxLength: 1000 }),
    (projectIdea) => {
      // Test property
    }
  ),
  { numRuns: 100 }
);
```

**Property Tests to Implement**:

1. **Input Validation Property** (Property 1)
  - Generate random strings of varying lengths
  - Verify acceptance/rejection based on length threshold
  - Tag: `Feature: team-foundry, Property 1: Input validation enforces minimum length`

2. **Roadmap Structure Property** (Property 2)
  - Generate random valid roadmaps
  - Verify all required fields are present and non-empty
  - Tag: `Feature: team-foundry, Property 2: Roadmap structure completeness`

3. **Candidate Count Property** (Property 3)
  - Generate random roadmaps with varying numbers of roles
  - Verify each role has at least 3 candidates
  - Tag: `Feature: team-foundry, Property 3: Candidate recommendation count`

4. **Candidate Profile Property** (Property 4)
  - Generate random candidate profiles
  - Verify all required fields present and match score in valid range
  - Tag: `Feature: team-foundry, Property 4: Candidate profile completeness`

5. **Team Selection Addition Property** (Property 5)
  - Generate random team states and candidates
  - Verify selection adds candidate to team
  - Tag: `Feature: team-foundry, Property 5: Team selection addition`

6. **Team Selection Removal Property** (Property 6)
  - Generate random team states with candidates
  - Verify removal removes candidate from team
  - Tag: `Feature: team-foundry, Property 6: Team selection removal`

7. **Team Display Consistency Property** (Property 7)
  - Generate random team selection states
  - Verify displayed team matches internal state
  - Tag: `Feature: team-foundry, Property 7: Team selection display consistency`

8. **Session Persistence Property** (Property 9)
  - Generate random session data (roadmaps, team selections)
  - Verify stored data matches retrieved data
  - Tag: `Feature: team-foundry, Property 9: Session data persistence`

9. **Error Message Safety Property** (Property 10)
  - Generate random errors with technical details
  - Verify user-facing messages don't contain stack traces, paths, or internal names
  - Tag: `Feature: team-foundry, Property 10: Error logging without exposure`

### Integration Testing

Integration tests will verify end-to-end flows:

- Complete flow: idea input → roadmap generation → candidate display → team selection
- Session persistence across multiple API calls
- Error recovery flows (retry after failure)

### Manual Testing Checklist

For the hackathon demo:
- [ ] Submit various project ideas and verify roadmap quality
- [ ] Verify UI responsiveness and loading indicators
- [ ] Test team selection/deselection interactions
- [ ] Verify session persistence across page refresh
- [ ] Test error scenarios (disconnect network, invalid input)
- [ ] Verify mobile responsiveness
- [ ] Test with different browsers (Chrome, Firefox, Safari)
