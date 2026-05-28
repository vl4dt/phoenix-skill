---
name: phoenix
version: 1.0.0
description: Expert Elixir and Phoenix Framework development for web apps, APIs, and real-time systems. Covers mix phx CLI workflows, HEEx components, Ecto contexts, LiveView, Channels, and deployment to Fly.io/Gigalixir/Heroku/Docker. Use when working with Elixir, Phoenix, mix phx.*, HEEx, LiveView, Ecto, Channels, or Phoenix-based web development.
source: github:vl4dt/phoenix-skill
license: MIT
---

# Phoenix

## Rules
1. **ALWAYS use `mix phx.*` tasks** — scaffolding, auth, releases.
2. **ALWAYS prefer LiveView** for interactive UIs unless API-only.
3. **ALWAYS keep business logic in contexts** (`lib/app/`), not controllers/views.

## Quick start
```bash
mix local.phx --force
mix phx.new my_app && cd my_app && mix setup && mix phx.server
```

## Mix CLI
| Command | Purpose |
|---|---|
| `mix local.phx --force` | Install phx_new archive |
| `mix phx.new <name>` | New Phoenix app |
| `mix setup` | Install deps, create DB, seed |
| `mix phx.server` | Dev server (hot reload) |
| `mix test` | Run tests (ExUnit) |
| `mix phx.routes` | List all routes |

## Scaffolding
| Command | Output |
|---|---|
| `mix phx.gen.html Ctx Model fields` | HTML CRUD, controller, view, schema, migration |
| `mix phx.gen.json Ctx Model fields` | JSON API, controller, schema, migration |
| `mix phx.gen.live Ctx Model fields` | LiveView CRUD, components, schema, migration |
| `mix phx.gen.auth Accounts User users` | Full auth (email, magic link, sessions) |
| `mix phx.gen.channel Room rooms` | WebSocket channel |
| `mix phx.gen.release` | Release scripts (+ `--docker` for Dockerfile) |
| `mix phx.gen.secret` | Generate SECRET_KEY_BASE |

## Core patterns
### Context (business logic)
```elixir
defmodule MyApp.Accounts do
  def list_users(), do: Repo.all(User)
  def create_user(attrs), do: %User{} |> User.changeset(attrs) |> Repo.insert()
end
```
### LiveView
```elixir
defmodule MyAppWeb.UserLive.Index do
  use MyAppWeb, :live_view
  def render(assigns), do: ~H"<%= for u <- @users do ... end %>"
  def mount(_p, _s, socket), do: {:ok, assign(socket, users: Accounts.list_users())}
end
```
### Channels (real-time)
```elixir
defmodule MyAppWeb.RoomChannel do
  use MyAppWeb, :channel
  def join("room:" <> id, _payload, socket), do: {:ok, %{room: id}, socket}
  def handle_in("msg", %{"body" => b}, socket), do: {:noreply, broadcast(socket, "msg", %{body: b})}
end
```

## Production
```bash
mix phx.gen.secret && export SECRET_KEY_BASE=<val>
mix deps.get --only prod && MIX_ENV=prod mix compile && MIX_ENV=prod mix assets.deploy
MIX_ENV=prod mix ecto.migrate && PORT=4001 MIX_ENV=prod mix phx.server
```
### Release
```bash
mix phx.gen.release && MIX_ENV=prod mix release
_build/prod/rel/app/bin/server   # Start
_build/prod/rel/app/bin/app remote  # Remote IEx
```
### Fly.io
`fly launch && fly deploy`

## Troubleshooting
| Problem | Fix |
|---|---|
| `mix phx.new` missing | `mix local.phx --force` |
| DB error | Check PostgreSQL; verify DATABASE_URL |
| LiveView reload | Check `mix phx.server` logs |
| Asset error | `MIX_ENV=prod mix assets.deploy` |

## Resources
- **Docs:** https://hexdocs.pm/phoenix/
- **LiveView:** https://hexdocs.pm/phoenix_live_view/
- **Elixir:** https://elixir-lang.org/getting-started/introduction.html
- **GitHub:** https://github.com/phoenixframework/phoenix
- **Forum:** https://elixirforum.com/
