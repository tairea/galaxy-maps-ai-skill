# Galaxy Maps AI Skills (V4 - Agent Teams)

A modular, multi-agent system for creating curriculum-based learning journeys (Galaxy Maps). V4 introduces **Claude Code Agent Teams** — competing designers, adversarial reviewers, and cross-referencing critiquers that coordinate via direct peer messaging. The orchestrator operates in **delegate mode**, never writing content directly.

## Skills Overview

| Skill | Description | Mode | Team Size |
|-------|-------------|------|-----------|
| [gm-ai-orchestrator](./gm-ai-orchestrator) | Team lead — coordinates workflow, manages teams, commits at phase boundaries | Delegate | 1 (lead) |
| [gm-ai-intent](./gm-ai-intent) | Captures curriculum intent via 6-canvas framework | Single session | 1 |
| [gm-ai-curriculum](./gm-ai-curriculum) | Generates curriculum structure via 4 competing designers | Agent Team | 4 teammates |
| [gm-ai-curriculum-critiquer](./gm-ai-curriculum-critiquer) | Reviews structure via 4 adversarial reviewers | Agent Team | 4 teammates |
| [gm-ai-branching](./gm-ai-branching) | Creates optional side-quest branches | Agent Team | 1 per star |
| [gm-ai-mission-builder](./gm-ai-mission-builder) | Generates rich HTML lesson content | Agent Team | 1 per star |
| [gm-ai-mission-critiquer](./gm-ai-mission-critiquer) | Reviews content via 4 cross-referencing reviewers | Agent Team | 4 teammates |

## Architecture

```
                          gm-ai-orchestrator
                          (Team Lead - Delegate)
                                    |
      +----------+----------+------+------+----------+----------+
      |          |          |             |          |          |
  Phase 1    Phase 2    Phase 3      Phase 4    Phase 5    Phase 6    Phase 7
  Intent     Curriculum Critique     Branching  Missions   Critique   Finalize
  (single)   (team)     (team)       (team)     (team)     (team)     (single)
             +------+   +------+    +------+   +------+   +------+
             | 4    |   | 4    |    | 1/   |   | 1/   |   | 4    |
             | comp.|   | adv. |    | star |   | star |   | revw.|
             | desn.|   | revw.|    |      |   |      |   |      |
             +------+   +------+    +------+   +------+   +------+

Within Agent Team phases:
  - Teammates share a task list and self-claim work
  - Teammates message each other directly (peer messaging)
  - Lead synthesizes results and enforces quality gates
  - Lead commits outputs at phase cleanup
```

## Quick Start

1. **Invoke the orchestrator** to start Galaxy Map creation:
   ```
   /galaxy-map
   ```
   or trigger with: "create a galaxy map", "build a curriculum"

2. **The orchestrator guides you through**:
   - Intent capture (6-canvas framework)
   - Curriculum structure generation (4 competing designers with distinct philosophies)
   - Optional adversarial critique and refinement cycles
   - Side-quest branch generation
   - Mission content creation with peer coordination
   - Optional content critique with cross-referencing
   - Publication to Galaxy Maps database

## Key Concepts

### Agent Teams
Phases 2-6 use Agent Teams — coordinated groups of specialized teammates that work in parallel and communicate directly with each other. The orchestrator acts as team lead in delegate mode, never writing content directly.

### Peer Messaging
Teammates within a team can message each other directly. Designers cross-review alternatives, reviewers debate findings, and builders coordinate terminology and visual style across stars.

### Quality Gates
Hooks enforce rules when teammates finish work:
- **TaskCompleted**: Validates output format and quality before marking work done
- **TeammateIdle**: Redirects finished teammates to review others' work

### Delegate Mode
The orchestrator never writes content files. During team phases, it creates teams, spawns teammates, approves plans, synthesizes results, resolves conflicts, and commits at phase boundaries.

### Stars (Milestones)
- 1-2 days of learning
- ONE clear concept/skill
- Tangible outcome at completion

### Missions (Lessons)
- 15-60 minutes each
- Atomic, focused learning objective
- Scaffolds from previous mission

### Branches (Side-Quests)
- Optional enrichment paths
- 30-90 minutes total
- Deep dives, tools, real-world applications

## File Outputs

```
~/galaxy-maps/{map-slug}/
├── .git/
├── INTENT.md                           # Phase 1: User intent (6 areas)
├── MAP_V1_{TEAM_NAME}.md              # Phase 2: Team-specific curriculum designs
├── MAP_V1.md                           # Phase 2: Selected curriculum structure
├── critiques/                          # Reviewer outputs
│   ├── CRITIQUE_PEDAGOGY.md            #   Phase 3: Curriculum critiques
│   ├── CRITIQUE_LEARNER.md
│   ├── CRITIQUE_SCOPE.md
│   ├── CRITIQUE_DEVILS.md
│   ├── MISSION_CRITIQUE_TECHNICAL.md   #   Phase 6: Mission critiques
│   ├── MISSION_CRITIQUE_LEARNING_DESIGN.md
│   ├── MISSION_CRITIQUE_ENGAGEMENT.md
│   └── MISSION_CRITIQUE_AUDIENCE.md
├── MAP_V{n}_SUGGESTIONS.md             # Phase 3: Consensus critique suggestions
├── MAP_V2.md                           # Refined structure (V3, V4... per iteration)
├── branches/                           # Phase 4: Side-quest branches
│   ├── STAR_{n}_BRANCH_{m}.md          #   1-3 branches per star
│   └── ...
├── missions/                           # Phase 5: HTML lesson content
│   ├── star_1/
│   │   ├── MISSION_1_1.md
│   │   ├── MISSION_1_1.html
│   │   └── ...
│   └── ...
├── MISSION_SUGGESTIONS.md              # Phase 6: Mission critique suggestions
└── GALAXY_MAP.json                     # Phase 7: Final database payload
```

## Documentation

- [SPEC_V4.md](./SPEC_V4.md) - Complete V4 specification (Agent Teams)
- [SPEC_V3.md](./SPEC_V3.md) - Previous V3 specification (superseded)
- Each skill's `SKILL.md` contains detailed implementation guidance
- Role files in `roles/` directories define teammate specializations
- Hook files in `hooks/` directories define quality gate rules

## Version History

- **V4** (2026-02-15): Agent Teams architecture with competing designers, adversarial reviewers, cross-referencing critiquers, delegate mode orchestrator, and quality gates
- **V3** (2025-01-17): Separated skills architecture
- **V2** (2025-01-15): Multi-agent monolithic system
- **V1** (2024): Single-prompt curriculum generation

## License

MIT
