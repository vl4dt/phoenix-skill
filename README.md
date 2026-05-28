# Phoenix Skill

Expert Elixir and Phoenix Framework development toolkit for [pi](https://pi.dev/) coding agents.

Covers web app scaffolding, HEEx/HEX components, Ecto contexts, LiveView, Channels, and deployment to Fly.io/Gigalixir/Heroku/Docker — using proper `mix phx.*` SDK commands instead of manual file manipulation.

## Install

```bash
pi install git:github.com/vl4dt/phoenix-skill
```

## Structure

```
phoenix-skill/
├── SKILL.md          # Main instructions (quick reference)
├── REFERENCE.md      # Detailed docs (generators, testing, deployment, clustering)
└── .gitignore        # Build artifacts
```

## Contents

- Mix CLI commands reference (`mix local.phx`, `mix phx.new`, `mix setup`, etc.)
- All generators: `phx.gen.html`, `phx.gen.json`, `phx.gen.live`, `phx.gen.auth`, `phx.gen.channel`, `phx.gen.release`
- Core patterns: contexts, LiveView, Channels, HEEx templates
- Production deployment: releases, Docker, Fly.io, Gigalixir, Heroku
- Testing: ExUnit, context tests, controller tests, channel tests, LiveView tests
- Clustering and service discovery
- Asset management (esbuild, Tailwind)
- Security best practices

## License

MIT
