# SDK Lead — Team Backlog

## Sprint Goals
*(Updated by SDK Lead at start of each sprint)*

| # | Goal | Owner | Status | Notes |
|---|------|-------|--------|-------|
|   | *(none yet)* | | | |

## In-Flight Tasks

| Issue | Title | Assignee | Priority | Status | Notes |
|-------|-------|----------|----------|--------|-------|
| | | | | | |

## Capacity Indicator
*(Updated each orchestrator pulse)*

```
Team: SDK | Members: 2 | Load: [●●○○○○] | In-flight: N | Open slots: M
Members: SDK-Dev, Plugin-Dev
```

## Adding a Task
1. Lead creates issue in the appropriate SDK repo (molecule-sdk-python, molecule-cli, molecule-mcp-server)
2. Lead assigns to SDK-Dev or Plugin-Dev with available capacity
3. Engineer implements, opens PR targeting staging
4. Lead reviews and merges

## Rules
- SDK changes require E2E verification (integration-tester validates the full workspace lifecycle)
- Breaking changes to SDK must be approved by Dev Lead before merge
- Plugin API changes must maintain backward compatibility
