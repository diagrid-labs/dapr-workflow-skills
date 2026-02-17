# CLAUDE.md

This repository contains Claude Code skill definitions for building Dapr Workflow applications.

## Repository structure

- `.claude/skills/` — Skill files organized by language/framework
  - `create-workflow-dotnet/SKILL.md` — Skill for creating Dapr Workflow apps with .NET

## Guidelines for skill files

- Skill files are Markdown documents in `.claude/skills/<skill-name>/SKILL.md`
- Each skill should include: front-matter with the name and description, and sections for: prerequisites, project setup, folder structure, and code examples
- Use `<PlaceholderName>` syntax for values the user should replace (e.g., `<ProjectName>`, `<ProjectNamespace>`)
- Code examples should be complete and runnable, not snippets
- The SKILL.md file should be minimal and refer to a reference.md file with more explanations and details
- Include "Key points" sections after code examples to explain important concepts
- Reference official Dapr quickstarts as the source of truth for patterns and conventions
