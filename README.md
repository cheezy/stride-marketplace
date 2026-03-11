# Stride Marketplace

Stride task management skills for AI agents.

## Installation

Add this marketplace to Claude Code:

```bash
/plugin marketplace add cheezy/stride-marketplace
```

## Available Plugins

### Stride

**Description:** Task lifecycle skills for Stride kanban: claiming, completing, creating tasks and goals

**Install:**
```bash
/plugin install stride@stride-marketplace
```

**Skills:**
- `stride-claiming-tasks` - Proper task claiming with hook execution
- `stride-completing-tasks` - Proper task completion with validation hooks
- `stride-creating-tasks` - Comprehensive task specification enforcement
- `stride-creating-goals` - Goal and batch creation with dependency management
- `stride-subagent-workflow` - Subagent orchestration for exploration, planning, and code review (Claude Code only)

**Agents:**
- `stride:task-explorer` - Targeted codebase exploration after claiming a task, guided by key_files and patterns_to_follow
- `stride:task-reviewer` - Pre-completion code review validating changes against acceptance_criteria and pitfalls

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
