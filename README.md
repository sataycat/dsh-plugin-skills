# dsh-plugin-skills

Reusable skills for agents working on external DeepSeek Harness plugins.

## Skills

- [`deepseek-harness-plugin`](skills/deepseek-harness-plugin/SKILL.md): create, extend, debug, and maintain external `dsh-plugin` repositories built on Cordis.

## Install For An Agent

Install the skill with the `skills` CLI. Run this from the project where you want the agent to use it:

```sh
npx skills add sataycat/dsh-plugin-skills --skill deepseek-harness-plugin
```

The CLI prompts you to select the agent(s) to configure and installs the skill into the selected agent's skills directory. The command works with supported agents including Claude Code, Codex, Cursor, GitHub Copilot, Windsurf, Gemini, Cline, and others supported by the CLI.

To install directly from GitHub, use the repository URL instead:

```sh
npx skills add https://github.com/sataycat/dsh-plugin-skills --skill deepseek-harness-plugin
```

The skill is then available when the selected agent is working on an external DeepSeek Harness plugin repository.

The skill keeps its main instructions concise and links to progressive references. DeepSeek Harness documentation and source remain the authority because the project is in developer preview and changes quickly.
