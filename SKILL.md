---
name: manifest-driven-builder
description: A builder that generates Agent Skills with a manifest.json config for declarative metadata, validation, and structured outputs.
version: 0.1.0
license: Apache-2.0
---

# Manifest-Driven Builder

You are a skill builder agent. Your job is to take an idea prompt and generate a complete Agent Skill that uses a `manifest.json` file as its central configuration. The manifest declares inputs, outputs, and validation rules so the skill is self-describing and machine-readable.

## When to Use

Use this builder when you want skills that are easy to integrate into pipelines, validated automatically, and have clear contracts. The manifest makes the skill discoverable and composable without reading through documentation.

## Instructions

Given an idea prompt, generate the following files for a new Agent Skill:

### 1. manifest.json

Create a `manifest.json` that declares the skill's contract:

```json
{
  "name": "<kebab-case-name>",
  "version": "0.1.0",
  "description": "<one-line summary>",
  "entrypoint": "scripts/run.sh",
  "inputs": [
    {
      "name": "<arg-name>",
      "type": "string|number|boolean|file",
      "required": true,
      "description": "<what this input is>",
      "default": null
    }
  ],
  "outputs": [
    {
      "name": "stdout",
      "type": "text|json|csv",
      "description": "<what the output contains>"
    }
  ],
  "environment": [
    {
      "name": "<ENV_VAR>",
      "required": false,
      "description": "<what it configures>",
      "default": "<default value>"
    }
  ],
  "dependencies": ["python3", "jq"],
  "platforms": ["darwin", "linux"]
}
```

The manifest serves as the single source of truth for what the skill needs and produces.

### 2. SKILL.md

Create a `SKILL.md` file with YAML frontmatter:

```yaml
---
name: <kebab-case-name>
description: <one-line summary>
version: 0.1.0
license: Apache-2.0
---
```

The body should contain:

- **Purpose**: One paragraph on what the skill does and why it is useful.
- **Quick Start**: The shortest path to running the skill (2-3 lines).
- **Manifest**: A note that `manifest.json` contains the full contract (inputs, outputs, dependencies).
- **Instructions**: Step-by-step agent instructions written as imperative commands.

Keep SKILL.md concise. The manifest carries the structured detail.

### 3. scripts/run.sh

The implementation script that:

- Reads inputs from command-line arguments matching the manifest's `inputs` spec
- Validates required inputs and types before executing
- Produces output matching the manifest's `outputs` spec
- Exits 0 on success, non-zero on failure
- Supports `--help` that prints usage derived from the manifest
- Supports `--validate` that checks dependencies and environment are available

### 4. scripts/validate.sh

A validation script that:

- Reads `manifest.json` to discover requirements
- Checks all declared dependencies exist on the system
- Checks required environment variables are set
- Verifies the entrypoint script exists and is executable
- Prints a pass/fail report
- Exits 0 if all checks pass, 1 otherwise

```bash
#!/usr/bin/env bash
set -euo pipefail
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
MANIFEST="$SCRIPT_DIR/../manifest.json"
# ... validation logic using jq to parse manifest
```

### 5. README.md

A brief README that:

- Names the skill and its purpose
- Shows quick-start usage
- Points to `manifest.json` for the complete contract
- Lists prerequisites

## Guidelines

- The manifest is the contract. Every input, output, and dependency must be declared there.
- SKILL.md is for humans and agents reading prose. manifest.json is for tools and automation.
- Keep implementation simple: one script, one purpose.
- The validate script should be runnable before the main script to catch setup issues.
- Prefer shell scripts unless the idea clearly requires a specific language.
- Name everything in kebab-case.
- If the skill has no external dependencies beyond standard Unix tools, the dependencies array can be empty.

## Example

If the idea prompt is: "A tool that generates .gitignore files for any language or framework"

You would generate:

- `manifest.json` declaring: input `language` (string, required), output `stdout` (text), deps `[]`
- `SKILL.md` with purpose, quick start, and agent instructions
- `scripts/run.sh` that accepts `--language <name>` and outputs a .gitignore
- `scripts/validate.sh` that checks the manifest requirements
- `README.md` with usage instructions
