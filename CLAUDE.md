# CLAUDE.md

This repository contains Claude Code skill definitions for building Dapr Workflow applications.

## Repository structure

- `.claude/skills/` — Skill files organized by language/framework
  - `workflow-dotnet/skill.md` — Skill for creating Dapr Workflow apps with .NET

## Guidelines for skill files

- Skill files are Markdown documents in `.claude/skills/<skill-name>/skill.md`
- Each skill should include: prerequisites, project setup, folder structure, and code examples
- Use `<PlaceholderName>` syntax for values the user should replace (e.g., `<ProjectName>`, `<ProjectNamespace>`)
- Code examples should be complete and runnable, not snippets
- Include "Key points" sections after code examples to explain important concepts
- Reference official Dapr quickstarts as the source of truth for patterns and conventions
