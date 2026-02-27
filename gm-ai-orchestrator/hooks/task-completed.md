# TaskCompleted Hook — Validation Rules

Runs when any teammate marks a task as complete. Prevents completion if quality criteria aren't met. The lead should reject the task and ask the teammate to fix the issues before re-completing.

---

## Phase 2: Curriculum Generation

When a designer marks their MAP alternative as complete, validate:

- [ ] File exists at `alternatives/MAP_ALT_{n}.md`
- [ ] YAML frontmatter is valid and parseable
- [ ] Frontmatter contains required fields: `mapId`, `intentId`, `version`, `title`, `description`, `estimatedDuration`, `totalStars`, `totalMissions`, `designPhilosophy`
- [ ] `designPhilosophy` matches the designer's assigned philosophy
- [ ] Stars follow the format: `- Star N - {Title} - {Milestone Objective}`
- [ ] Missions follow the format: `  - Mission N.M - {Title} - {Learning Objective}`
- [ ] At least 3 stars present
- [ ] Each star has at least 3 missions
- [ ] Learning objectives use measurable verbs (not "understand", "learn about", "explore")

---

## Phase 3: Curriculum Critique

When a reviewer marks their critique as complete, validate:

- [ ] File exists at `critiques/CRITIQUE_{ROLE}.md`
- [ ] Critique references specific star/mission numbers (not vague "some stars")
- [ ] Each finding includes a severity rating (critical/major/minor)
- [ ] Each finding includes a concrete suggested change (not "improve this")
- [ ] At least 3 findings present
- [ ] Findings reference actual stars/missions that exist in the MAP file

---

## Phase 4: Branching

When a teammate marks their star's branches as complete, validate:

- [ ] At least 1 branch file exists for the assigned star
- [ ] Each branch file has valid YAML frontmatter with required fields: `branchId`, `parentStarIndex`, `title`, `type`, `estimatedDuration`
- [ ] `type` is one of: `deep-dive`, `adjacent-topic`, `tool`, `real-world`, `history`
- [ ] Each branch has 2-4 missions with learning objectives
- [ ] Branch topic is not a duplicate of another star's branch (check existing branch files)
- [ ] Branch is clearly related to the parent star's topic

---

## Phase 5: Mission Building

When a builder marks a mission as complete, validate:

- [ ] Both `.md` and `.html` files exist for the mission
- [ ] HTML file contains all 6 required sections:
  - `<section class="mission-hook">`
  - `<section class="mission-objective">`
  - `<section class="mission-content-main">`
  - `<section class="mission-check">`
  - `<section class="mission-bridge">`
  - At least one `<div class="try-it">` or `<div class="code-example">`
- [ ] Code examples use syntax highlighting: `class="language-{lang}"`
- [ ] `.md` file has valid YAML frontmatter with `starIndex`, `missionIndex`, `learningObjective`
- [ ] Content length is reasonable (not a stub — at least 300 words in HTML)

---

## Phase 6: Mission Critique

When a reviewer marks their critique as complete, validate:

- [ ] File exists at `critiques/MISSION_CRITIQUE_{ROLE}.md`
- [ ] Each finding includes a severity rating (critical/major/minor)
- [ ] Each finding references a specific mission number (e.g., "Mission 2.3")
- [ ] Each finding includes a specific suggested fix (not "make it better")
- [ ] Reviewer has reviewed all missions (check mission count against MAP file)
- [ ] YAML frontmatter includes `missionsReviewed` count
