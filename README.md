# Stride Marketplace

Stride task management skills for AI agents.

## Installation

Add this marketplace to Claude Code:

```bash
/plugin marketplace add cheezy/stride-marketplace
```

## Available Plugins

### Stride

**Description:** Task lifecycle skills for Stride kanban: claiming, completing, creating tasks and goals, enriching tasks, and subagent orchestration

**Install:**
```bash
/plugin install stride@stride-marketplace
```

**Skills (all MANDATORY — each contains required API fields only documented in that skill):**
- `stride-claiming-tasks` - MANDATORY before GET /api/tasks/next or POST /api/tasks/claim
- `stride-completing-tasks` - MANDATORY before PATCH /api/tasks/:id/complete
- `stride-creating-tasks` - MANDATORY before POST /api/tasks (work tasks or defects)
- `stride-creating-goals` - MANDATORY before POST /api/tasks/batch (goals with nested tasks)
- `stride-enriching-tasks` - MANDATORY when task has empty key_files/testing_strategy/verification_steps
- `stride-subagent-workflow` - MANDATORY after claiming any task, before implementation (Claude Code only)

**Agents:**
- `stride:task-decomposer` - Breaks goals and large tasks into dependency-ordered child tasks
- `stride:task-explorer` - Targeted codebase exploration after claiming a task, guided by key_files and patterns_to_follow
- `stride:task-reviewer` - Pre-completion code review validating changes against acceptance_criteria and pitfalls
- `stride:hook-diagnostician` - Analyzes hook failure output and returns a prioritized fix plan

**Repository:** https://github.com/cheezy/stride

---

## Marketplace Structure

```
stride-marketplace/
├── .claude-plugin/
│   └── marketplace.json       # Plugin catalog
├── README.md                  # This file
└── LICENSE                    # MIT License
```

## Support

- **Issues**: https://github.com/cheezy/stride-marketplace/issues
- **Stride Plugin**: https://github.com/cheezy/stride

## License

MIT License
