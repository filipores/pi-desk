# Pi Desk

Your package workspace: inboxes, priorities, and next steps.

Pi Desk is a tiny workspace for one Pi package. Drop work into the inbox,
let it rank what matters first, and keep the next step visible without turning
your package into a project-management ceremony.

Notion-ish. Package-sized.

## Install

```bash
pi install git:github.com/filipores/pi-desk
```

## Commands

```text
/todo                 show the ranked inbox
/todo <text>          add an inbox entry and prioritize
/todo sort            reprioritize
/todo done <id>       delete an entry
/todo move <id> <n>   set a manual rank
/todo clear           delete all entries after confirmation
/todo setup           choose project context files again
```

Prioritization runs automatically from the current Pi/project context; `/todo setup`
only adds extra project files. Data is stored locally under `~/.pi/agent/pi-desk/`.
