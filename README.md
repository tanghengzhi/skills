# Agent Skills

A collection of agent skills for Laravel API development.

## Skills

### Laravel API Development

- **laravel-api** — Build RESTful APIs with Laravel 11+ using Controller → Service → Model architecture, FormRequest validation, API Resources, and PSR-12 coding standards.

  **Install with Claude Code:**
  ```
  npx skills@latest add tanghengzhi/skills/laravel-api
  ```

  **Install with GitHub Copilot:**
  ```bash
  mkdir -p .github
  curl -o .github/copilot-instructions.md \
    https://raw.githubusercontent.com/tanghengzhi/skills/main/.github/copilot-instructions.md
  ```

## Usage

### Claude Code

Install a skill into your project using the `npx skills` CLI:

```bash
npx skills@latest add tanghengzhi/skills/laravel-api
```

This downloads the skill files into `.claude/skills/laravel-api/` in your project. Claude Code automatically discovers and loads skills from this directory based on the task context described in each skill's `description` frontmatter field.

### GitHub Copilot

GitHub Copilot reads `.github/copilot-instructions.md` automatically. Copy the file to your project:

```bash
mkdir -p .github
curl -o .github/copilot-instructions.md \
  https://raw.githubusercontent.com/tanghengzhi/skills/main/.github/copilot-instructions.md
```

Copilot will then apply the Laravel API development guidelines to all code suggestions in your project.