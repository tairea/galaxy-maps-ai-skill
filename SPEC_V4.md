# Galaxy Maps AI V4 - Agent Teams Specification

## Overview

Galaxy Maps AI V4 evolves the separated skills architecture (V3) by introducing **Claude Code Agent Teams** for phases that benefit from parallel collaboration. The orchestrator becomes a team lead in delegate mode, and phases that involve generation or critique use teams of specialized teammates that can communicate directly with each other.

### Key Improvements Over V3
- **Agent Teams**: Phases 2, 3, 5, and 6 use coordinated teams of teammates instead of isolated subagents
- **Direct Peer Communication**: Teammates message each other to maintain consistency and debate quality
- **Adversarial Review**: Critique phases use teammates with distinct lenses that argue with each other
- **Competing Alternatives**: Curriculum generation uses teammates with different design philosophies
- **Delegate Mode**: Orchestrator focuses purely on coordination, never generates content
- **Quality Gates**: Hooks enforce schema validation and redirect idle teammates
- **Plan Approval**: Critical phases require lead approval before teammates begin implementation

### What Stays the Same from V3
- Modular skill architecture (each agent has its own SKILL.md)
- Standardized file schemas (INTENT.md, MAP_V{n}.md, etc.)
- Repository structure
- Phase 1 (Intent) and Phase 7 (Finalize) remain single-session
- MCP server integrations per skill

---

## Agent Overview

| Agent | Purpose | Mode | Team Size |
|-------|---------|------|-----------|
| gm-ai-orchestrator | Team lead, coordinates all phases | Delegate | 1 (lead) |
| gm-ai-intent | Captures user intent through 6-canvas framework | Single session | 1 |
| gm-ai-curriculum | Generates curriculum structure (Stars/Missions) | Agent Team | 4 teammates |
| gm-ai-curriculum-critiquer | Reviews and debates structure quality | Agent Team | 4 teammates |
| gm-ai-branching | Creates optional side-quest branches | Agent Team | 1 per star |
| gm-ai-mission-builder | Generates HTML lesson content | Agent Team | 1 per star |
| gm-ai-mission-critiquer | Reviews and debates content quality | Agent Team | 4 teammates |

---

## System Architecture

### Phase Execution Model

```
Phase 1: Intent Capture          [Single Session - no team]
    |
Phase 2: Curriculum Generation   [AGENT TEAM - 4 competing designers]
    |
Phase 3: Curriculum Critique     [AGENT TEAM - 4 adversarial reviewers]
    |
Phase 4: Branching Side-Quests   [AGENT TEAM - 1 teammate per star]
    |
Phase 5: Mission Building        [AGENT TEAM - 1 teammate per star, peer messaging]
    |
Phase 6: Mission Critique        [AGENT TEAM - 4 content reviewers]
    |
Phase 7: Finalize & Publish      [Single Session - lead only]
```

### Communication Model

**V3 (hub-and-spoke)**: All communication routed through orchestrator. Skills never talk to each other.

**V4 (hybrid)**: The orchestrator remains the team lead, but within Agent Team phases:
- Teammates share a task list and self-claim work
- Teammates message each other directly (terminology coordination, cross-referencing findings)
- Lead synthesizes results and makes approval decisions
- Lead enforces quality gates via hooks

```
                        +---------------------------+
                        |    gm-ai-orchestrator     |
                        |    (Team Lead - Delegate)  |
                        +---------------------------+
                                    |
          +----------+--------------+--------------+----------+
          |          |              |              |          |
          v          v              v              v          v
    +-----------+  +============================+  +-----------+
    | Phase 1   |  | Phases 2, 3, 4, 5, 6      |  | Phase 7   |
    | Intent    |  | AGENT TEAMS                |  | Finalize  |
    | (single)  |  | Teammates <-> Teammates    |  | (single)  |
    +-----------+  | Shared task list            |  +-----------+
                   | Direct messaging            |
                   | Lead synthesizes results    |
                   +============================+
```

---

## Phase Specifications

### Phase 1: Intent Capture (Single Session)

**Agent**: gm-ai-intent
**Location**: `gm-ai-intent/SKILL.md`
**Mode**: Single session, no team needed

**Purpose**: Interactive 1-on-1 conversation to capture curriculum intent through the 6-canvas framework.

**The 6 Areas**:
1. Audience (Who) - target learners
2. Topic/Domain (What) - subject matter
3. Unique Ideas - differentiators
4. Outcomes - success criteria
5. Timing - duration and pacing
6. Assessment - mastery measurement

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Audience Profiler MCP | Learner personas |
| Topic Taxonomy MCP | Subject standards |
| Duration Estimator | Timing calculations |

**Output**: `INTENT.md`

**Orchestrator Actions**:
1. Invoke gm-ai-intent skill
2. Interactive session with user
3. `git init` and commit `INTENT.md`

---

### Phase 2: Curriculum Generation (Agent Team - 4 Competing Designers)

**Agent**: gm-ai-curriculum
**Location**: `gm-ai-curriculum/SKILL.md`
**Mode**: Agent Team with 4 teammates

**Purpose**: Generate 4 structurally different curriculum alternatives from the same INTENT.md. Each teammate embodies a fundamentally different pedagogical philosophy.

**Core Principles** (shared by all teammates):
- Stars: 1-2 days, ONE clear milestone, tangible outcome
- Missions: 15-60 minutes, atomic, scaffolded LOs

#### The 4 Curriculum Designers

| # | Teammate | Philosophy | Structural Signature |
|---|----------|-----------|---------------------|
| 1 | **Foundations-Up Designer** | Master fundamentals before advancing | Linear progression, heavy scaffolding, strict prerequisite chains. Stars build on each other like a staircase. Theory before practice. Earlier stars are denser with foundational missions. |
| 2 | **Project-Driven Designer** | Learn by building real things | Each star produces a tangible artifact. Learning objectives emerge from what you need to build. Hands-on from mission 1. Stars are named after what you create, not what you learn. |
| 3 | **Challenge-First Designer** | Start with problems, backfill knowledge | Stars open with a challenge the learner can't yet solve. Missions teach what's needed to solve it. Motivation through productive struggle. Each star ends by revisiting the opening challenge. |
| 4 | **Spiral Designer** | Revisit topics at increasing depth | Broad early exposure, then circles back. Stars overlap intentionally. Concepts interconnect rather than isolate. Later stars deepen earlier ones. Good for complex, interconnected domains. |

#### Team Dynamic: Compete Then Cross-Review

1. **Generate** (parallel): Each teammate independently generates a complete MAP alternative
2. **Cross-review** (peer messaging): Teammates review each other's structures, surfacing tradeoffs
3. **Synthesize** (lead): Orchestrator compiles a comparison table with pros/cons from each reviewer
4. **User selects**: User chooses a preferred structure (or requests a hybrid)

#### Spawn Prompt Template
```
You are the {philosophy} curriculum designer. Your job is to generate a
complete Galaxy Map curriculum structure from INTENT.md.

Your design philosophy: {detailed_philosophy}

Read INTENT.md, then generate a complete MAP following MAP.template.md.
After generating, you will review the other 3 designers' maps. Be specific
about tradeoffs — what does your approach handle better, and what does theirs?

Output your map to alternatives/MAP_ALT_{n}.md
```

#### Plan Approval
The lead requires plan approval from each designer before they begin generating. Each designer must submit an outline of their star structure and explain how their philosophy shapes it. The lead approves only if the outline is genuinely distinct from the others.

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Learning Standards MCP | Educational standards |
| Bloom's Taxonomy Tool | Cognitive levels |
| Prerequisite Graph | Dependency analysis |

**Outputs**:
- `alternatives/MAP_ALT_1.md` (Foundations-Up)
- `alternatives/MAP_ALT_2.md` (Project-Driven)
- `alternatives/MAP_ALT_3.md` (Challenge-First)
- `alternatives/MAP_ALT_4.md` (Spiral)
- `alternatives/COMPARISON.md` (Lead-synthesized comparison)
- `MAP_V1.md` (User-selected or hybrid)

---

### Phase 3: Curriculum Critique (Agent Team - 4 Adversarial Reviewers)

**Agent**: gm-ai-curriculum-critiquer
**Location**: `gm-ai-curriculum-critiquer/SKILL.md`
**Mode**: Agent Team with 4 teammates

**Purpose**: Apply 4 distinct quality lenses to the chosen MAP. Teammates debate each other's findings to produce a consensus set of suggestions.

#### The 4 Curriculum Reviewers

| # | Teammate | Lens | What They Evaluate |
|---|----------|------|-------------------|
| 1 | **Pedagogy Reviewer** | Learning science rigor | Bloom's taxonomy progression within stars. Scaffolding quality — does each mission build on the last? Are prerequisites explicit? Are learning objectives measurable verbs, not vague? Is cognitive load managed well? |
| 2 | **Learner Advocate** | Engagement and motivation | Would a real learner stick with this? Is the pacing monotonous? Are there enough "aha" moments? Does it feel like a slog anywhere? Where will learners drop off? Is there variety in mission types? |
| 3 | **Scope Auditor** | Feasibility and sizing | Do stars actually fit in 1-2 days? Are missions actually 15-60 minutes? Does the total match INTENT.md timing? Is anything over-scoped or under-scoped? Are there missions that try to cover too much? |
| 4 | **Devil's Advocate** | Gaps, assumptions, failure modes | What's missing from the intent that the map doesn't cover? What prior knowledge is assumed but never stated? Where will the structure break for edge-case learners? What would a competitor's curriculum include that this one doesn't? |

#### Team Dynamic: Adversarial Debate

1. **Independent review** (parallel): Each reviewer analyzes MAP_V{n}.md through their lens
2. **Cross-challenge** (peer messaging): Reviewers directly challenge each other's findings
   - Pedagogy Reviewer: "Add a scaffolding mission before Star 3"
   - Scope Auditor: "That blows the timing budget by 2 hours"
   - Learner Advocate: "Without it, learners will hit a wall at Star 3"
   - Devil's Advocate: "The real problem is Star 2 assumes too much — fix that instead"
3. **Consensus** (lead synthesizes): Orchestrator resolves conflicts and produces ranked suggestions
4. **User decides**: User approves/declines suggestions; approved ones feed back to gm-ai-curriculum for MAP_V{n+1}.md

#### Spawn Prompt Template
```
You are the {role} for Galaxy Maps curriculum review. Your job is to
review MAP_V{n}.md through the lens of {lens}.

Read INTENT.md for context on the target audience, timing, and goals.
Read MAP_V{n}.md and produce a critique focused exclusively on {focus_areas}.

Be specific: reference exact star/mission numbers. Propose concrete changes,
not vague suggestions. After your review, you will debate the other reviewers.
Defend your position but concede when they make a stronger argument.

Output your critique to critiques/CRITIQUE_{role}.md
```

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Pedagogical Analyzer | Best practices check |
| Scope Validator | Coverage verification |
| Transition Analyzer | Gap detection |

**Outputs**:
- `critiques/CRITIQUE_PEDAGOGY.md`
- `critiques/CRITIQUE_LEARNER.md`
- `critiques/CRITIQUE_SCOPE.md`
- `critiques/CRITIQUE_DEVILS.md`
- `MAP_V{n}_SUGGESTIONS.md` (Lead-synthesized consensus)

---

### Phase 4: Branching Side-Quests (Agent Team - 1 per Star)

**Agent**: gm-ai-branching
**Location**: `gm-ai-branching/SKILL.md`
**Mode**: Agent Team with 1 teammate per star

**Purpose**: Generate optional side-quest branches in parallel. Teammates coordinate to avoid proposing redundant branches across stars.

**Branch Types**:
1. Deep Dive - deeper on touched topics
2. Adjacent Topic - related distinct areas
3. Tool/Technique - alternative approaches
4. Real-World Application - industry context
5. History/Context - background information

#### Team Dynamic
- Each teammate owns one star's branches
- Teammates message each other: "I'm creating a Deep Dive on recursion for Star 2 — don't duplicate in Star 3"
- Lead verifies no redundancy across stars

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Topic Explorer MCP | Knowledge graph traversal |
| Wikipedia/Wikidata API | Research |
| Industry Trends MCP | Real-world applications |

**Output**: `branches/STAR_{n}_BRANCH_{m}.md`

---

### Phase 5: Mission Building (Agent Team - 1 per Star)

**Agent**: gm-ai-mission-builder
**Location**: `gm-ai-mission-builder/SKILL.md`
**Mode**: Agent Team with 1 teammate per star

**Purpose**: Generate rich HTML lesson content in parallel. Each teammate owns all missions for one star. Teammates coordinate via direct messaging for cross-star consistency.

**Content Structure** (per mission):
1. Hook - engaging opening
2. Objective - clear statement
3. Content - core teaching
4. Practice - hands-on application
5. Check - understanding verification
6. Bridge - next mission connection

#### Team Dynamic: Parallel Build with Peer Coordination
- Each teammate generates all missions for their assigned star
- **Direct messaging for consistency**:
  - "I introduced the term 'closure' in Mission 1.3 — reference it without re-explaining in Star 2"
  - "I'm using a running code example (todo app) through Star 1 — should we carry it into Star 2?"
  - "What visual style are you using for diagrams? Let's stay consistent"
- Teammates self-claim stars from the shared task list
- Lead monitors progress and reassigns if a teammate gets stuck

#### Spawn Prompt Template
```
You are a mission builder for Star {n}: "{star_title}".

Read INTENT.md for audience context. Read MAP_V{n}.md for the full structure.
Your job is to generate complete HTML lesson content for every mission in
Star {n}.

For each mission, follow the 6-part content structure (Hook, Objective,
Content, Practice, Check, Bridge). Output both .md (planning) and .html
(final content) for each mission.

Coordinate with other star builders via messaging to maintain terminology
consistency and avoid re-explaining concepts covered in earlier stars.

Output to missions/star_{n}/MISSION_{n}_{m}.md and .html
```

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| YouTube MCP | Educational videos |
| D3/Visualization Generator | Diagrams |
| Code Playground Embedder | Runnable examples |
| Quiz Builder | Assessment questions |
| Code Validator | Syntax checking |

**Outputs**:
- `missions/star_{n}/MISSION_{n}_{m}.md`
- `missions/star_{n}/MISSION_{n}_{m}.html`

---

### Phase 6: Mission Critique (Agent Team - 4 Content Reviewers)

**Agent**: gm-ai-mission-critiquer
**Location**: `gm-ai-mission-critiquer/SKILL.md`
**Mode**: Agent Team with 4 teammates

**Purpose**: Apply 4 distinct quality lenses to all generated mission content. Reviewers cross-reference findings to catch connected issues across missions.

#### The 4 Mission Reviewers

| # | Teammate | Lens | What They Evaluate |
|---|----------|------|-------------------|
| 1 | **Technical Accuracy Reviewer** | Correctness | Do code examples actually work? Are explanations factually correct? Are there bugs, outdated APIs, or misleading simplifications? Would a learner hit errors following this? Are dependencies and imports complete? |
| 2 | **Learning Design Reviewer** | Pedagogical structure | Does the Hook actually hook? Does the Content teach what the Objective promises? Does the Check assess what was taught (not something else)? Does the Bridge connect logically to the next mission? Is LO alignment end-to-end? |
| 3 | **Engagement Reviewer** | Interest and interactivity | Is this dry or alive? Are interactive elements well-designed? Are examples interesting or generic? Would you read this willingly or only because you had to? Are practice exercises creative or boilerplate? |
| 4 | **Audience Fit Reviewer** | Target learner appropriateness | Does the language level match the audience from INTENT.md? Are there jargon gaps — terms used without introduction? Do examples resonate with the target demographic? Is the pace right for the stated skill level? |

#### Team Dynamic: Cross-Referencing Findings

1. **Independent review** (parallel): Each reviewer reads all missions through their lens
2. **Cross-reference** (peer messaging): Reviewers connect related findings
   - Audience Fit: "Mission 2.1 uses the term 'polymorphism' without introduction"
   - Learning Design: "Confirmed — Mission 1.4 was supposed to teach it but only mentions it in passing"
   - Technical Accuracy: "The code example in Mission 2.1 also uses polymorphism incorrectly"
   - Engagement: "Mission 2.1 is also the weakest engagement-wise — the hook is a definition"
3. **Synthesize** (lead): Orchestrator groups findings by mission with severity ratings
4. **User decides**: User approves/declines; approved findings feed back to mission builders

#### Task Distribution
Each reviewer reviews ALL missions (not a subset). The shared task list contains one task per mission per reviewer (e.g., "Technical Accuracy: Review Mission 1.1"). This ensures complete coverage.

#### Spawn Prompt Template
```
You are the {role} for Galaxy Maps mission review. Your job is to
review all generated mission content through the lens of {lens}.

Read INTENT.md for audience context. Read MAP_V{n}.md for structure.
Review each mission's .md and .html files in missions/star_*/

For each issue found, specify:
- Mission number (e.g., Mission 2.3)
- Severity (critical / major / minor)
- Specific issue description
- Suggested fix

After your review, cross-reference with other reviewers. Connected issues
across lenses should be flagged as compound problems.

Output to critiques/MISSION_CRITIQUE_{role}.md
```

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Readability Analyzer | Language level |
| Code Linter | Syntax validation |
| HTML Validator | Accessibility |
| LO Coverage Checker | Objective alignment |

**Outputs**:
- `critiques/MISSION_CRITIQUE_TECHNICAL.md`
- `critiques/MISSION_CRITIQUE_LEARNING_DESIGN.md`
- `critiques/MISSION_CRITIQUE_ENGAGEMENT.md`
- `critiques/MISSION_CRITIQUE_AUDIENCE.md`
- `MISSION_SUGGESTIONS.md` (Lead-synthesized, grouped by mission)

---

### Phase 7: Finalize & Publish (Single Session)

**Agent**: gm-ai-orchestrator (lead only)
**Mode**: Single session, no team

**Purpose**: Transform all generated files into the final Galaxy Map JSON and publish.

**Steps**:
1. Clean up team from Phase 6
2. Transform repo files -> `saveGalaxyMap()` JSON
3. Call `saveGalaxyMap()` cloud function
4. Commit `GALAXY_MAP.json` (archive)
5. Push to remote
6. Return `courseId` to user

---

## Orchestrator Specification

**Agent**: gm-ai-orchestrator
**Location**: `gm-ai-orchestrator/SKILL.md`

**Purpose**: Team lead that coordinates the entire multi-phase workflow. Runs in **delegate mode** during team phases — never generates content, only coordinates.

**Responsibilities**:
1. Initialize and manage git repository
2. Orchestrate phases in correct sequence
3. Handle user decisions between phases
4. **Create and manage Agent Teams** for Phases 2-6
5. **Spawn teammates** with appropriate roles and prompts
6. **Synthesize team outputs** into actionable results
7. **Enforce quality gates** via hooks
8. **Approve/reject teammate plans** before implementation
9. Commit and push files at checkpoints
10. Transform output to `saveGalaxyMap()` JSON
11. Clean up teams between phases

**Delegate Mode Behavior**:
During team phases, the orchestrator:
- Creates tasks in the shared task list
- Spawns teammates with detailed role prompts
- Reviews and approves teammate plans
- Synthesizes findings when teammates complete
- Resolves conflicts between teammate recommendations
- **Never writes content files directly**

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Git MCP Server | Repository operations |
| GitHub MCP Server | Remote repo management |
| File System MCP | Read/write files |
| Firebase MCP | Database operations |

**Triggers**:
- "create a galaxy map"
- "build a curriculum"
- "/galaxy-map"

---

## Quality Gates (Hooks)

Agent Teams support hooks that enforce rules when teammates finish work. These are critical for maintaining quality across parallel work.

### TaskCompleted Hook
Runs when any teammate marks a task as complete. Prevents completion if quality criteria aren't met.

| Phase | Validation |
|-------|-----------|
| Phase 2 | MAP alternative follows MAP.template.md schema; frontmatter is valid YAML |
| Phase 3 | Critique references specific star/mission numbers; includes concrete suggestions |
| Phase 4 | Branch structure follows BRANCH.template.md; no duplicate topics across stars |
| Phase 5 | Mission .html is valid HTML; all 6 content sections present; code examples have syntax highlighting |
| Phase 6 | Critique includes severity ratings; references specific missions; includes suggested fixes |

### TeammateIdle Hook
Runs when a teammate finishes their work and is about to go idle. Redirects them to useful work.

| Phase | Redirect Action |
|-------|----------------|
| Phase 2 | Review another designer's alternative that hasn't been cross-reviewed yet |
| Phase 3 | Review another reviewer's critique for blind spots |
| Phase 5 | Help review completed missions from other stars for cross-star consistency |
| Phase 6 | Double-check highest-severity findings from other reviewers |

---

## Team Lifecycle Per Phase

Each team phase follows this lifecycle:

```
1. SETUP
   - Lead creates team: "Create agent team for Phase {n}"
   - Lead creates shared task list with all work items
   - Lead spawns teammates with role-specific prompts

2. PLAN APPROVAL (Phases 2 and 5)
   - Teammates work in read-only plan mode
   - Each submits an outline/approach to the lead
   - Lead approves if approach is sound and distinct from others
   - Rejected teammates revise and resubmit

3. EXECUTION
   - Teammates self-claim tasks from shared list
   - Teammates message each other for coordination
   - TaskCompleted hooks validate quality on each task
   - TeammateIdle hooks redirect finished teammates

4. SYNTHESIS
   - Lead waits for all teammates to complete
   - Lead synthesizes outputs into a unified result
   - Lead presents synthesis to user for decisions

5. CLEANUP
   - Lead shuts down all teammates
   - Lead cleans up team resources
   - Lead commits outputs to git
   - Lead proceeds to next phase (or creates new team)
```

---

## File Schemas

### INTENT.md
```yaml
---
intentId: <uuid>
mapTitle: <string>
mapDescription: <string>
createdAt: <ISO8601>
createdBy: <string>
status: draft | complete
---

# 1. Audience (Who)
# 2. Topic/Domain (What)
# 3. Unique Ideas
# 4. Outcomes
# 5. Timing
# 6. Assessment/Mastery
```

### MAP_V{n}.md
```yaml
---
mapId: <uuid>
intentId: <reference>
version: <n>
name: <slug>
title: <string>
description: <string>
tags: [<tags>]
estimatedDuration: <time>
totalStars: <count>
totalMissions: <count>
createdAt: <ISO8601>
status: draft | review | approved
designPhilosophy: foundations-up | project-driven | challenge-first | spiral | hybrid
---

- Star 1 - <Title> - <Milestone Objective>
  - Mission 1.1 - <Title> - <Learning Objective>
  - Mission 1.2 - <Title> - <Learning Objective>
```

### Handoff Protocol
```json
{
  "from": "gm-ai-{skill}",
  "to": "gm-ai-orchestrator",
  "status": "complete | needs-input | error",
  "files": ["file1.md", "file2.html"],
  "stats": { ... },
  "message": "Human-readable status message"
}
```

---

## Repository Structure

```
~/galaxy-maps/{map-slug}/
├── .git/
├── INTENT.md
├── alternatives/
│   ├── MAP_ALT_1.md          (Foundations-Up)
│   ├── MAP_ALT_2.md          (Project-Driven)
│   ├── MAP_ALT_3.md          (Challenge-First)
│   ├── MAP_ALT_4.md          (Spiral)
│   └── COMPARISON.md         (Lead-synthesized)
├── MAP_V1.md
├── critiques/
│   ├── CRITIQUE_PEDAGOGY.md
│   ├── CRITIQUE_LEARNER.md
│   ├── CRITIQUE_SCOPE.md
│   ├── CRITIQUE_DEVILS.md
│   ├── MISSION_CRITIQUE_TECHNICAL.md
│   ├── MISSION_CRITIQUE_LEARNING_DESIGN.md
│   ├── MISSION_CRITIQUE_ENGAGEMENT.md
│   └── MISSION_CRITIQUE_AUDIENCE.md
├── MAP_V1_SUGGESTIONS.md
├── MAP_V2.md
├── branches/
│   ├── STAR_1_BRANCH_1.md
│   ├── STAR_2_BRANCH_1.md
│   └── STAR_2_BRANCH_2.md
├── missions/
│   ├── star_1/
│   │   ├── MISSION_1_1.md
│   │   ├── MISSION_1_1.html
│   │   └── ...
│   └── star_2/
│       └── ...
├── MISSION_SUGGESTIONS.md
└── GALAXY_MAP.json            (final archive)
```

---

## Skill Directory Structure

```

├── gm-ai-orchestrator/
│   ├── SKILL.md
│   ├── schemas/
│   │   ├── intent.schema.json
│   │   ├── map.schema.json
│   │   └── handoff.schema.json
│   ├── templates/
│   │   └── workflow-state.yaml
│   └── hooks/
│       ├── task-completed.sh
│       └── teammate-idle.sh
├── gm-ai-intent/
│   ├── SKILL.md
│   ├── prompts/
│   │   └── elicitation-hints.md
│   └── templates/
│       └── INTENT.template.md
├── gm-ai-curriculum/
│   ├── SKILL.md
│   ├── roles/
│   │   ├── foundations-up.md
│   │   ├── project-driven.md
│   │   ├── challenge-first.md
│   │   └── spiral.md
│   ├── rules/
│   │   └── design-principles.md
│   └── templates/
│       └── MAP.template.md
├── gm-ai-curriculum-critiquer/
│   ├── SKILL.md
│   ├── roles/
│   │   ├── pedagogy-reviewer.md
│   │   ├── learner-advocate.md
│   │   ├── scope-auditor.md
│   │   └── devils-advocate.md
│   └── frameworks/
│       └── critique-checklist.md
├── gm-ai-branching/
│   ├── SKILL.md
│   └── templates/
│       └── BRANCH.template.md
├── gm-ai-mission-builder/
│   ├── SKILL.md
│   ├── templates/
│   │   ├── mission.template.md
│   │   └── mission.template.html
│   └── styles/
│       └── mission.css
└── gm-ai-mission-critiquer/
    ├── SKILL.md
    ├── roles/
    │   ├── technical-accuracy.md
    │   ├── learning-design.md
    │   ├── engagement.md
    │   └── audience-fit.md
    └── frameworks/
        └── quality-rubric.md
```

---

## Complete Workflow

```
PHASE 1: INTENT CAPTURE (Single Session)
-----------------------------------------
User -> gm-ai-orchestrator -> gm-ai-intent
                                   |
                                   v
                            [Interactive session]
                            [6 areas of intent]
                                   |
                                   v
                            Output: INTENT.md
                            git init, commit INTENT.md


PHASE 2: CURRICULUM GENERATION (Agent Team - 4 Designers)
----------------------------------------------------------
gm-ai-orchestrator creates team, spawns 4 teammates:
  +------------------+  +------------------+  +------------------+  +------------------+
  | Foundations-Up    |  | Project-Driven   |  | Challenge-First  |  | Spiral           |
  | Designer         |  | Designer         |  | Designer         |  | Designer         |
  +------------------+  +------------------+  +------------------+  +------------------+
          |                      |                      |                      |
          v                      v                      v                      v
    [Plan approval]        [Plan approval]        [Plan approval]        [Plan approval]
          |                      |                      |                      |
          v                      v                      v                      v
    MAP_ALT_1.md           MAP_ALT_2.md           MAP_ALT_3.md           MAP_ALT_4.md
          |                      |                      |                      |
          +----------+-----------+-----------+----------+
                     |                       |
                     v                       v
              [Cross-review via             [Lead synthesizes
               peer messaging]              COMPARISON.md]
                                             |
                                             v
                                      User selects structure
                                             |
                                             v
                                      commit MAP_V1.md
                                      cleanup team


PHASE 3: STRUCTURE CRITIQUE (Agent Team - 4 Reviewers, Repeatable)
-------------------------------------------------------------------
gm-ai-orchestrator creates team, spawns 4 teammates:
  +------------------+  +------------------+  +------------------+  +------------------+
  | Pedagogy         |  | Learner          |  | Scope            |  | Devil's          |
  | Reviewer         |  | Advocate         |  | Auditor          |  | Advocate         |
  +------------------+  +------------------+  +------------------+  +------------------+
          |                      |                      |                      |
          v                      v                      v                      v
    [Independent review]   [Independent review]   [Independent review]   [Independent review]
          |                      |                      |                      |
          +----------+-----------+-----------+----------+
                     |
                     v
              [Adversarial debate
               via peer messaging]
                     |
                     v
              [Lead resolves conflicts,
               synthesizes MAP_V{n}_SUGGESTIONS.md]
                     |
                     v
              User approves/declines suggestions
                     |
                     v
              If approved: gm-ai-curriculum regenerates MAP_V{n+1}.md
              commit MAP_V{n+1}.md
              cleanup team
              (Repeat Phase 3 if user wants another round)


PHASE 4: BRANCHING SIDE-QUESTS (Agent Team - 1 per Star)
---------------------------------------------------------
gm-ai-orchestrator creates team, spawns N teammates (1 per star):
  +------------------+  +------------------+       +------------------+
  | Star 1 Brancher  |  | Star 2 Brancher  |  ...  | Star N Brancher  |
  +------------------+  +------------------+       +------------------+
          |                      |                          |
          v                      v                          v
    [Generate branches]    [Generate branches]        [Generate branches]
          |                      |                          |
          +----------+-----------+-----------+--------------+
                     |
                     v
              [Peer messaging to avoid
               redundant branches]
                     |
                     v
              commit branches/*
              cleanup team


PHASE 5: MISSION BUILDING (Agent Team - 1 per Star)
-----------------------------------------------------
gm-ai-orchestrator creates team, spawns N teammates (1 per star):
  +------------------+  +------------------+       +------------------+
  | Star 1 Builder   |  | Star 2 Builder   |  ...  | Star N Builder   |
  +------------------+  +------------------+       +------------------+
          |                      |                          |
          v                      v                          v
    [Plan approval]        [Plan approval]            [Plan approval]
          |                      |                          |
          v                      v                          v
    [Build missions]       [Build missions]           [Build missions]
          |                      |                          |
          +<--- peer messaging for terminology, style, cross-references --->+
          |                      |                          |
          v                      v                          v
    missions/star_1/*      missions/star_2/*          missions/star_N/*
                     |
                     v
              commit missions/*
              cleanup team


PHASE 6: MISSION CRITIQUE (Agent Team - 4 Reviewers, Repeatable)
-----------------------------------------------------------------
gm-ai-orchestrator creates team, spawns 4 teammates:
  +------------------+  +------------------+  +------------------+  +------------------+
  | Technical        |  | Learning Design  |  | Engagement       |  | Audience Fit     |
  | Accuracy         |  | Reviewer         |  | Reviewer         |  | Reviewer         |
  +------------------+  +------------------+  +------------------+  +------------------+
          |                      |                      |                      |
          v                      v                      v                      v
    [Review ALL missions]  [Review ALL missions]  [Review ALL missions]  [Review ALL missions]
          |                      |                      |                      |
          +----------+-----------+-----------+----------+
                     |
                     v
              [Cross-reference findings
               via peer messaging]
                     |
                     v
              [Lead synthesizes MISSION_SUGGESTIONS.md
               grouped by mission, with severity ratings]
                     |
                     v
              User approves/declines
                     |
                     v
              If approved: mission builders regenerate affected missions
              cleanup team
              (Repeat Phase 6 if user wants another round)


PHASE 7: FINALIZE & PUBLISH (Single Session)
---------------------------------------------
gm-ai-orchestrator:
  1. Transform repo files -> saveGalaxyMap JSON
  2. Call saveGalaxyMap() cloud function
  3. Commit GALAXY_MAP.json (archive)
  4. Push to remote
  5. Return courseId to user
```

---

## Token Cost Considerations

Agent Teams use significantly more tokens than single sessions. Each teammate is a full Claude instance with its own context window.

| Phase | Teammates | Relative Cost | Justification |
|-------|-----------|--------------|---------------|
| Phase 1 | 1 | Low | Single interactive session |
| Phase 2 | 4 | High | Structurally different alternatives worth the cost |
| Phase 3 | 4 | Medium-High | Adversarial debate catches issues a single reviewer misses |
| Phase 4 | N (stars) | Medium | Parallel generation, light coordination |
| Phase 5 | N (stars) | Highest | Largest content volume, but parallelism saves wall-clock time |
| Phase 6 | 4 | Medium-High | Cross-referencing finds compound issues |
| Phase 7 | 1 | Low | Single transformation session |

**Cost optimization**: For smaller maps (3 or fewer stars), Phases 4 and 5 can use subagents instead of Agent Teams since cross-star coordination is minimal.

---

## Implementation Priority

| Priority | Component | Rationale |
|----------|-----------|-----------|
| 1 | gm-ai-orchestrator | Core team lead, must exist first |
| 2 | gm-ai-intent | Entry point, user-facing, no team needed |
| 3 | gm-ai-curriculum + 4 roles | Critical path, highest-value team improvement |
| 4 | gm-ai-mission-builder | Content generation, parallel builds |
| 5 | gm-ai-curriculum-critiquer + 4 roles | Structure quality via adversarial debate |
| 6 | gm-ai-mission-critiquer + 4 roles | Content quality via cross-referencing |
| 7 | gm-ai-branching | Enhancement, not critical path |
| 8 | Hooks (TaskCompleted, TeammateIdle) | Quality enforcement, can add incrementally |

---

## Known Limitations

Agent Teams are experimental. Plan for these constraints:

1. **No session resumption**: If a session disconnects, in-process teammates are lost. The lead must spawn new teammates. Design phases to commit work frequently.
2. **One team per session**: The lead must clean up each team before starting the next phase's team. This enforces the sequential phase structure naturally.
3. **No nested teams**: Teammates cannot spawn their own teams. All coordination goes through the lead.
4. **Task status lag**: Teammates sometimes fail to mark tasks complete. The lead should periodically check actual file outputs, not just task status.
5. **Permissions inherited**: All teammates start with the lead's permission mode. Individual teammate permissions can be changed after spawn but not at spawn time.
6. **Split panes require tmux/iTerm2**: In-process mode works anywhere but is harder to monitor. For a 4-teammate team, split panes are strongly recommended.

---

## Future Considerations

### gm-ai-realtime-tutor (Post-MVP)
- Assists learners during mission completion
- Provides hints and explanations
- Assesses mastery before progression

### Branch Content Generation
- Currently branches are structure-only
- Future: gm-ai-mission-builder team generates HTML for branch missions too

### Shared Infrastructure
- gm-ai-schema-validator: Validates all .md file formats (integrate with TaskCompleted hooks)
- gm-ai-storage: Unified file I/O
- gm-ai-handoff: Standardized inter-skill communication

### Dynamic Team Sizing
- Automatically adjust teammate count based on map complexity
- Small maps (2-3 stars): use subagents for Phases 4-5 instead of full teams
- Large maps (8+ stars): split Phase 5 into batches to manage token cost

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 4.0 | 2026-02-15 | Agent Teams architecture with competing designers, adversarial reviewers, and cross-referencing critiquers |
| 3.0 | 2025-01-17 | Separated skills architecture |
| 2.0 | 2025-01-15 | Multi-agent monolithic system |
| 1.0 | 2024-XX-XX | Original single-prompt system |
