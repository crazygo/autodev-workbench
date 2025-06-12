# CodeX Agent vs Augment Tools

This document provides an overview of CodeX Agent's capabilities and compares them with the toolset found in this repository's Augment implementation. The information below highlights which features overlap and where each system differs.

## CodeX Agent Overview

CodeX Agent is a development assistant designed to operate on a local codebase. Key capabilities include:

- **Shell execution** – run commands inside a temporary workspace with resource limits
- **File management** – read, write, and patch files with automatic version control
- **Git operations** – create commits and manage pull requests
- **Task automation** – run tests, linters, or build scripts as part of iterative workflows
- **Structured reasoning** – generate plans and summarise changes for the user

These abilities allow CodeX to automate typical development tasks such as applying patches, running test suites, and creating pull requests.

## OpenAI Tool Use Protocol

CodeX Agent exposes its capabilities to the underlying language model using the
standard **OpenAI tool use** specification. Each tool is declared as a function
with a name, description, and JSON schema for parameters. The model chooses a
tool by name and supplies the arguments in JSON. A typical tool definition looks
like this:

```json
{
  "type": "function",
  "function": {
    "name": "tool_name",
    "description": "What the tool does",
    "parameters": {
      "type": "object",
      "properties": {
        "example": { "type": "string", "description": "Example parameter" }
      },
      "required": ["example"]
    }
  }
}
```

This contract ensures that the agent and the model share a consistent interface
when invoking tools.

### Core CodeX Tools

The following tools are used by CodeX Agent. All are implemented with the above
protocol:

```json
{
  "type": "function",
  "function": {
    "name": "run",
    "description": "Execute a shell command inside the workspace",
    "parameters": {
      "type": "object",
      "properties": {
        "cmd": { "type": "string", "description": "Shell command to run" },
        "timeout": { "type": "integer", "description": "Timeout in seconds" }
      },
      "required": ["cmd"]
    }
  }
}
```

```json
{
  "type": "function",
  "function": {
    "name": "read_file",
    "description": "Read a file from the repository",
    "parameters": {
      "type": "object",
      "properties": {
        "path": { "type": "string", "description": "File path" }
      },
      "required": ["path"]
    }
  }
}
```

```json
{
  "type": "function",
  "function": {
    "name": "apply_patch",
    "description": "Apply a unified diff patch to the repository",
    "parameters": {
      "type": "object",
      "properties": {
        "patch": { "type": "string", "description": "Unified diff" }
      },
      "required": ["patch"]
    }
  }
}
```

```json
{
  "type": "function",
  "function": {
    "name": "run_tests",
    "description": "Execute the project's test suite",
    "parameters": {
      "type": "object",
      "properties": {
        "cmd": { "type": "string", "description": "Command to run tests" }
      },
      "required": ["cmd"]
    }
  }
}
```

```json
{
  "type": "function",
  "function": {
    "name": "create_pr",
    "description": "Create a pull request with the committed changes",
    "parameters": {
      "type": "object",
      "properties": {
        "title": { "type": "string", "description": "Pull request title" },
        "body": { "type": "string", "description": "Pull request description" }
      },
      "required": ["title", "body"]
    }
  }
}
```

## Augment Tooling

The Augment implementation inside this repository exposes a broad set of tools. The current capabilities include filesystem operations, terminal interaction, process management, code analysis, and GitHub integration:

- `list-directory`, `read-file`, `write-file`, `delete-file`, `str-replace-editor`
- `run-terminal-command`
- `read-terminal`, `write-process`, `kill-process`
- `launch-process`, `list-processes`, `read-process`
- `analyze-basic-context`, `search-keywords`, `code-search-regex`
- GitHub helpers like `github-get-issue`, `github-create-issue`, and `github-analyze-issue`
- Web utilities such as `web-fetch-content` and `web-search`

These tools are summarised in the documentation【F:packages/github-agent/TOOL_CAPABILITIES.md†L9-L44】.

The same document identifies several missing capabilities, notably semantic code search (`codebase-retrieval`), IDE diagnostics, visualization tools, and long-term memory storage【F:packages/github-agent/TOOL_CAPABILITIES.md†L46-L93】.

Another comparison file lists the differences between Augment and the AutoDev Remote Agent, showing that Augment uniquely provides `diagnostics`, `codebase-retrieval`, `render-mermaid`, `remember`, and bulk file deletion, while the AutoDev agent offers richer GitHub and process management support【F:packages/github-agent/docs/TOOL_COMPARISON_ANALYSIS.md†L33-L45】.

Tool counts are also contrasted, with Augment containing 15 core tools versus 26 in the AutoDev Remote Agent【F:packages/github-agent/docs/TOOL_COMPARISON_ANALYSIS.md†L24-L29】.

## Comparison to CodeX Agent

| Capability | CodeX Agent | Augment |
|------------|------------|---------|
| Shell/command execution | ✓ built-in with timeout control | ✓ `run-terminal-command` |
| File reading/writing | ✓ patch-based editing | ✓ `read-file`, `write-file`, `str-replace-editor` |
| Process management | limited background job support | ✓ `launch-process` and related tools |
| Code search | text search and grep | basic keyword search; semantic search planned (`codebase-retrieval`) |
| Git integration | commit creation and PR submission | extensive GitHub API tools |
| Diagnostics | uses test output & linter commands | planned (`diagnostics`) |
| Visualization & memory | not directly provided | planned: `render-mermaid`, `remember` |

CodeX focuses on patching and testing workflows, whereas Augment exposes a larger variety of environment management and GitHub-specific commands. Augment still plans features (semantic search, diagnostics, diagrams, memory) that CodeX does not currently supply out of the box.

## Conclusion

CodeX Agent aims to automate iterative coding tasks with strong Git and test integration. Augment's tools extend further into repository analysis, process control, and web utilities. Combining CodeX's automated workflow management with Augment's broader toolbox could yield a comprehensive development assistant.
