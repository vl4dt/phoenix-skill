# Phoenix Reference

## Mix CLI — Full Reference

### mix local.phx
Installs the `phx_new` archive globally. Required before using `mix phx.new`.
```bash
mix local.phx --force   # Force reinstall from hex
mix archive.install hex phx_new  # Alternative: install specific version
```

### mix phx.new
Creates a new Phoenix application.
```bash
mix phx.new my_app
```

Flags:
- `--no-ecto` — Skip Ecto database layer (API-only apps)
- `--no-html` — Skip HTML views (JSON API only)
- `--no-dashboard` — Skip Phoenix Dashboard
- `--no-mailer` — Skip Swoosh mailer

### mix phx.gen.* — Generators

#### mix phx.gen.html
Generates HTML CRUD scaffold: controller, view, HEEx templates, schema, migration, context.
```bash
mix phx.gen.html Accounts User users name:string email:string:unique
```
Generates files in:
- `lib/my_app/accounts.ex` — Context module
- `lib/my_app/accounts/user.ex` — Ecto schema
- `lib/my_app_web/controllers/user_controller.ex` — Controller
- `lib/my_app_web/views/user_view.ex` — View
- `lib/my_app_web/live/user_live/*` — HEEx templates
- `priv/repo/migrations/*_create_users.exs` — Migration
- `test/my_app/accounts_test.exs` — Context tests
- `test/my_app_web/controllers/user_controller_test.exs` — Controller tests

#### mix phx.gen.json
Generates JSON API scaffold: controller, schema, migration (no views).
```bash
mix phx.gen.json Accounts Product products name:string price:decimal
```

#### mix phx.gen.live
Generates LiveView CRUD scaffold: LiveViews, components, schema, migration.
```bash
mix phx.gen.live Accounts Post posts title:string body:text
```
Generates:
- `lib/my_app_web/live/post_live/index.ex` — List page
- `lib/my_app_web/live/post_live/show.ex` — Detail page
- `lib/my_app_web/live/post_live/form_component.ex` — Form modal
- `lib/my_app_web/components/post_live.ex` — Core components

After generation, manually update the router:
```elixir
# Add to router.ex within the :browser scope
live "/posts", PostLive.Index, :index
live "/posts/new", PostLive.Index, :new
live "/posts/:id", PostLive.Show, :show
live "/posts/:id/edit", PostLive.Index, :edit
```

#### mix phx.gen.auth
Generates full authentication system: email/magic link, password reset, sessions.
```bash
mix phx.gen.auth Accounts User users
mix phx.gen.auth Accounts User users --live    # LiveView auth pages
mix phx.gen.auth Accounts User users --no-live  # Controller auth pages
mix phx.gen.auth Accounts User users --hashing-lib argon2  # Use argon2 (most secure)
```
Generates:
- `Accounts` context with `User` and `UserToken` schemas
- Email confirmation workflow
- Password reset workflow
- Session management
- Scopes for auth pipelines
- Required router updates (auth pipeline, sessions)

#### mix phx.gen.channel
Generates a Phoenix Channel for real-time features.
```bash
mix phx.gen.channel Room rooms
```

#### mix phx.gen.schema
Generates only a schema and migration (no context, no views).
```bash
mix phx.gen.schema Post posts title:string body:text
```

#### mix phx.gen.release
Generates release scripts for production deployment.
```bash
mix phx.gen.release                  # Standard release scripts
mix phx.gen.release --docker          # Generate Dockerfile + release scripts
```
Generates:
- `rel/overlays/bin/server` — Start Phoenix server in release
- `rel/overlays/bin/migrate` — Run migrations in release
- `lib/my_app/release.ex` — Custom Release module for migrations
- `rel/vm.args.eex` — VM args
- `rel/env.sh.eex` — Runtime environment config

#### mix phx.gen.secret
Generates a cryptographically secure secret key.
```bash
mix phx.gen.secret
# Output: xYz... (64-char hex string)
```

### mix phx.* — Utility Tasks

| Command | Purpose |
|---|---|
| `mix phx.server` | Start dev server on port 4000 |
| `mix phx.digest` | Compile static assets (images, CSS, JS) |
| `mix phx.digest.clean` | Remove digested assets |
| `mix phx.digest.clean --all` | Remove all digested assets |
| `mix phx.routes` | Print all defined routes |
| `mix phx.gen.cert` | Generate self-signed SSL certs for HTTPS |

## Project Directory Structure

```
my_app/
├── _build/           # Compilation artifacts (gitignore)
├── assets/           # Frontend source (JS, CSS)
│   ├── js/           # JavaScript source (ESBuild)
│   └── css/          # CSS source
├── config/
│   ├── config.exs    # Base config
│   ├── dev.exs       # Development overrides
│   ├── test.exs      # Test overrides
│   └── prod.exs      # Production overrides
│   └── runtime.exs   # Runtime secrets (env vars)
├── deps/             # Mix dependencies (gitignore)
├── lib/
│   ├── my_app/       # Business logic / contexts
│   │   ├── application.ex
│   │   ├── repo.ex
│   │   ├── mailer.ex
│   │   └── accounts.ex
│   └── my_app.ex     # Application entry
├── lib/my_app_web/   # Web layer
│   ├── controllers/
│   │   ├── page_controller.ex
│   │   └── error_html.ex / error_json.ex
│   ├── components/
│   │   ├── core_components.ex
│   │   └── layouts.ex
│   ├── live/         # LiveView modules
│   ├── routers/
│   ├── static/       # Static files served by Plug.Static
│   ├── endpoint.ex   # HTTP entry point
│   └── router.ex     # Route definitions
├── priv/
│   ├── repo/         # Database migrations
│   └── static/       # Compiled static assets
├── test/             # ExUnit tests
│   ├── my_app/       # Context tests
│   └── my_app_web/   # Controller/LiveView tests
├── rel/              # Release configuration
├── mix.exs           # Mix project config
└── mix.lock          # Dependency lock
```

## Ecto Contexts

### Context Pattern

Phoenix uses contexts as API boundaries for domains:
```elixir
defmodule MyApp.Accounts do
  @doc "List all users"
  @spec list_users() :: [User.t()]
  def list_users(), do: Repo.all(User)

  @doc "Get a single user"
  @spec get_user(integer) :: User.t() | nil
  def get_user(id), do: Repo.get(User, id)

  @doc "Create a user"
  @spec create_user(map) :: {:ok, User.t()} | {:error, Ecto.Changeset.t()}
  def create_user(attrs \\ %{}) do
    %User{} |> User.changeset(attrs) |> Repo.insert()
  end

  @doc "Update a user"
  @spec update_user(User.t(), map) :: {:ok, User.t()} | {:error, Ecto.Changeset.t()}
  def update_user(%User{} = user, attrs) do
    user |> User.changeset(attrs) |> Repo.update()
  end

  @doc "Delete a user"
  @spec delete_user(User.t()) :: {:ok, User.t()} | {:error, Ecto.Changeset.t()}
  def delete_user(%User{} = user), do: Repo.delete(user)
end
```

### Ecto Schema

```elixir
defmodule MyApp.Accounts.User do
  use Ecto.Schema
  import Ecto.Changeset

  schema "users" do
    field :name, :string
    field :email, :string
    field :password, :string, virtual: true
    field :password_hash, :string
    timestamps()
  end

  def changeset(user, attrs) do
    user
    |> cast(attrs, [:name, :email, :password])
    |> validate_required([:name, :email, :password])
    |> validate_email()
    |> validate_length(:password, min: 8)
    |> unique_constraint(:email)
  end

  defp validate_email(changeset) do
    changeset
    |> validate_format(:email, ~r/@/)
    |> put_change(:email_hash, hash_email())
  end
end
```

## LiveView — In-Depth

### LiveView Lifecycle

```
mount(params, session, socket)
    ↓
render(assigns)
    ↓
handle_event(event, params, socket)
    ↓
handle_info(message, socket)
    ↓
update(assigns, socket)
```

### mount/3

```elixir
defmodule MyAppWeb.PageLive do
  use MyAppWeb, :live_view

  # mount can be called on initial load and page navigation
  def mount(params, session, socket) do
    {:ok, assign(socket, page: "Welcome")}
    # {:ok, socket} or {:error, reason} also valid
  end
end
```

### handle_event/3

```elixir
def handle_event("submit", %{"user" => user_params}, socket) do
  case Accounts.create_user(user_params) do
    {:ok, user} ->
      {:noreply,
       socket
       |> put_flash(:info, "User created!")
       |> redirect(to: ~p"/users/#{user}")}
    {:error, %Ecto.Changeset{} = cs} ->
      {:noreply, assign(socket, changeset: cs)}
  end
end
```

### handle_info/2

```elixir
def handle_info(:notify, socket) do
  {:noreply, assign(socket, notified: true)}
end
```

### LiveView Templates (HEEx)

```heex
<.form for={@form} phx-submit="save">
  <.input field={@form[:name]} type="text" label="Name" />
  <.input field={@form[:email]} type="email" label="Email" />
  <:actions>
    <.button>Save</.button>
  </:actions>
</.form>

<!-- Raw HTML with Elixir interpolation -->
<div id="user-<%= @user.id %>">
  <h1><%= @user.name %></h1>
  <p><%= @user.email %></p>
</div>

<!-- For loop -->
<%= for user <- @users do %>
  <li><%= user.name %></li>
<% end %>

<!-- Component call -->
<.header>
  Dashboard
  <:subtitle>Manage your account</:subtitle>
</.header>
```

### LiveView Routes

```elixir
scope "/", MyAppWeb do
  pipe_through :browser
  live "/users", UserLive.Index, :index
  live "/users/new", UserLive.Index, :new
  live "/users/:id", UserLive.Show, :show
  live "/users/:id/edit", UserLive.Index, :edit
end
```

## Channels — In-Depth

### Channel Module

```elixir
defmodule MyAppWeb.RoomChannel do
  use MyAppWeb, :channel

  # Topic format: "room:<room_id>"
  def join("room:" <> room_id, _payload, socket) do
    {:ok, %{room: room_id, messages: []}, socket}
  end

  # Reject join
  def join("room:private", _payload, _socket) do
    {:error, %{reason: "you can't join this room"}}
  end

  # Handle incoming event
  def handle_in("new_msg", %{"body" => body}, socket) do
    broadcast(socket, "new_msg", %{body: body, user: socket.assigns.current_user})
    {:noreply, socket}
  end

  # Handle incoming message with reply
  def handle_in("ping", _payload, socket) do
    reply(socket, :pong)
  end

  # Called when client disconnects
  def terminate(_reason, _socket) do
    :ok
  end
end
```

### Channel Registration in Router

```elixir
defmodule MyAppWeb.Router do
  use MyAppWeb, :router

  pipeline :browser do
    # ...
  end

  scope "/", MyAppWeb do
    # Channels
    channel "room:*", RoomChannel
  end
end
```

### Channel Tests

```elixir
defmodule MyAppWeb.RoomChannelTest do
  use MyAppWeb.ChannelCase

  test "join returns room data", %{socket: socket} do
    assert {:ok, %{room: "general", messages: []}, socket} =
             socket |> join("room:general", %{})
  end

  test "broadcasts new messages", %{socket: socket} do
    assert :ok = socket |> push("new_msg", %{"body" => "hello"})
    assert_broadcast("new_msg", %{body: "hello"})
  end
end
```

## Phoenix.PubSub

### Broadcasting from Context

```elixir
Phoenix.PubSub.broadcast(MyApp.PubSub, "room:general", {:new_message, msg})
```

### Subscribing in LiveView

```elixir
def mount(_params, _session, socket) do
  Phoenix.PubSub.subscribe(MyApp.PubSub, "room:general")
  {:ok, assign(socket, messages: [])}
end

def handle_info({:new_message, msg}, socket) do
  {:noreply, assign(socket, messages: [msg | socket.assigns.messages])}
end
```

## Deployment — In-Depth

### General Production Steps

```bash
# 1. Generate secret key
mix phx.gen.secret
export SECRET_KEY_BASE=<output>
export DATABASE_URL=ecto://user:pass@host/db

# 2. Install production deps only
mix deps.get --only prod

# 3. Compile
MIX_ENV=prod mix compile

# 4. Compile assets
MIX_ENV=prod mix assets.deploy

# 5. Run migrations
MIX_ENV=prod mix ecto.migrate

# 6. Start server
PORT=4001 MIX_ENV=prod mix phx.server

# Detached mode
PORT=4001 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server

# Interactive shell
PORT=4001 MIX_ENV=prod iex -S mix phx.server
```

### Release Mode

```bash
# Generate release scripts
mix phx.gen.release

# Build release
MIX_ENV=prod mix release

# Output: _build/prod/rel/my_app/
```

Release commands:
```bash
_build/prod/rel/my_app/bin/my_app start              # Start as daemon
_build/prod/rel/my_app/bin/server                    # Start with Phoenix server
_build/prod/rel/my_app/bin/my_app stop               # Graceful shutdown
_build/prod/rel/my_app/bin/my_app remote             # Remote IEx shell
_build/prod/rel/my_app/bin/my_app eval "Code.eval_file(\"lib/my_app/release.ex\")"
_build/prod/rel/my_app/bin/my_app eval "MyApp.Release.migrate"  # Run migrations
_build/prod/rel/my_app/bin/my_app run "MyApp.Release.migrate"   # One-shot command
```

### Release Configuration

`rel/env.sh.eex` — Runtime environment:
```bash
export ERL_EPMD_PORT=4369
export ERL_AFLAGS="-kernel inet_dist_listen_min 4370 inet_dist_listen_max 4372"
export RELEASE_DISTRIBUTION="name"
export RELEASE_NODE="app@${HOSTNAME}"
export DNS_CLUSTER_QUERY="myapp.internal"
```

### Docker Deployment

```bash
mix phx.gen.release --docker  # Generates multi-stage Dockerfile
docker build -t my_app .
docker run -p 4001:4001 -e SECRET_KEY_BASE=val -e DATABASE_URL=ecto://... my_app
```

The generated Dockerfile uses:
- `hexpm/elixir` image with matching OTP version for building
- Debian slim image for runtime (no VM needed, release includes ERTS)

### Clustering

Erlang VM clustering for multi-instance deployments:

```bash
# rel/env.sh.eex
export RELEASE_DISTRIBUTION="name"
export RELEASE_NODE="app-${SHA}@${IP}"
export DNS_CLUSTER_QUERY="app.internal"
```

Alternative discovery libraries:
- `libcluster` — gossip, kubernetes, EC2 strategies
- `libcluster_postgres` — PostgreSQL-based node discovery

### Config/Runtime Secrets

`config/runtime.exs` — Runtime configuration from environment variables:
```elixir
config :my_app, MyAppWeb.Endpoint,
  secret_key_base: System.get_env("SECRET_KEY_BASE")

config :my_app, MyApp.Repo,
  url: System.get_env("DATABASE_URL"),
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10")
```

### Fly.io Deployment

```bash
fly launch                 # Auto-detects Phoenix, creates Dockerfile, sets up Postgres
fly deploy                 # Deploy changes
fly secrets set KEY=val    # Set environment secrets
fly status                 # Check deployment status
fly logs                   # Tail application logs
fly open                   # Open app in browser
fly dashboard              # Open Fly dashboard
fly scale count 2          # Scale to multiple instances
fly regions add ewr        # Add multi-region deployment
fly ssh console            # SSH into instance
fly ssh console -C "/app/bin/my_app remote"  # Remote IEx
```

fly.toml configuration:
```toml
[build]
  dockerfile = "Dockerfile"

[env]
  PORT = "4001"

[mounts]
  source = "data"
  destination = "/data"

[[services]]
  internal_port = 4001
  protocol = "tcp"
```

### Gigalixir Deployment

```bash
fly create my_app
fly deploy
```

Gigalixir deploys natively (no Docker) using `mix releases`.

### Heroku Deployment

Requires `heroku.yml` or Dockerfile-based deployment.

## Testing — In-Depth

### ExUnit

```bash
mix test                    # Run all tests
mix test test/my_app/accounts_test.exs
mix test test/my_app/accounts_test.exs:42  # Line-specific test
mix test --trace           # Verbose output
mix test --only pending    # Only pending tests
mix test --max-failures 1  # Stop after N failures
```

### Context Tests

```elixir
defmodule MyApp.AccountsTest do
  use MyApp.DataCase

  test "list_users/0 returns all users" do
    user = Accounts.create_user(%{name: "Alice", email: "alice@example.com"})
    assert Accounts.list_users() |> Enum.map(& &1.id) |> length() == 1
  end

  test "create_user/1 validates required fields" do
    assert {:error, %Ecto.Changeset{}} = Accounts.create_user(%{})
  end
end
```

### Controller Tests

```elixir
defmodule MyAppWeb.UserControllerTest do
  use MyAppWeb.ConnCase

  @create_attrs %{name: "Alice", email: "alice@example.com"}

  test "lists all users", %{conn: conn} do
    conn = get(conn, ~p"/users")
    assert html_response(conn, 200) =~ "Listing Users"
  end

  test "creates user and redirects", %{conn: conn} do
    conn = post(conn, ~p"/users", user: @create_attrs)
    assert redirected_to(conn) == ~p"/users"
    assert get_flash(conn, :info) =~ "User created successfully"
  end
end
```

### Channel Tests

```elixir
defmodule MyAppWeb.RoomChannelTest do
  use MyAppWeb.ChannelCase

  setup do
    {:ok, _, socket} = subscribe_and_join(MyAppWeb.RoomSocket, "room:general")
    %{socket: socket}
  end

  test "broadcasts new messages", %{socket: socket} do
    assert :ok = push(socket, "new_msg", %{"body" => "hello"})
    assert_broadcast("new_msg", %{body: "hello"})
  end
end
```

### LiveView Tests

```elixir
defmodule MyAppWeb.UserLiveTest do
  use MyAppWeb.ConnCase

  import Phoenix.LiveViewTest

  test "renders index", %{conn: conn} do
    {:ok, _view, html} = live(conn, ~p"/users")
    assert html =~ "Users"
  end

  test "creates new user", %{conn: conn} do
    view = live(conn, ~p"/users/new")
    view = live(view, ~p"/users", action: :insert, user: %{name: "Bob"})
    assert redirected_to(view) == ~p"/users"
  end
end
```

## Authn/Authz — In-Depth

### mix phx.gen.auth

```bash
mix phx.gen.auth Accounts User users
```

Generated scopes (in router.ex):
```elixir
scope "/", MyAppWeb do
  pipe_through :browser
  live "/", PageLive, :index
end

scope "/user", MyAppWeb do
  pipe_through [:browser, :require_authenticated_user]
  live "/dashboard", DashboardLive, :index
end
```

### Custom Auth Middleware

```elixir
defmodule MyAppWeb.Plugs.RequireAuth do
  import Plug.Conn
  import Phoenix.Controller

  def init(options), do: options

  def call(conn, _opts) do
    case conn.assigns[:current_user] do
      nil ->
        conn
        |> put_flash(:error, "Please log in.")
        |> redirect(to: ~p"/user/login")
        |> halt()
      user ->
        assign(conn, :current_user, user)
    end
  end
end
```

## Asset Management

### esbuild

Phoenix uses esbuild for JavaScript bundling (default). Config in `config/config.exs`:
```elixir
config :esbuild, :default,
  args: ~w(js/app.js --bundle --target=es2017 --outdir=../priv/static/assets --minify),
  cd: Path.expand("assets", __DIR__),
  env: %{"NODE_PATH" => Path.expand("../deps", __DIR__)}
```

### Tailwind CSS

Phoenix v1.6+ includes Tailwind CSS:
```bash
npm install --prefix assets tailwindcss npx tailwindcss -i css/app.css -o ../priv/static/assets/app.css --minify
```

### Custom Asset Build

Replace esbuild with any build tool. Override in `mix.exs`:
```elixir
defp aliases do
  [
    "assets.deploy": ["cmd cd assets && npm run build", "phx.digest"]
  ]
end
```

## Security

### CSRF Protection

Built-in via `Plug.Session` and `protect_from_forgery()`.

### SQL Injection

Prevented via Ecto parameterized queries.

### XSS Protection

HEEx templates auto-escape all interpolated values. Use `~H"..."` for HTML templates.

### Rate Limiting

Use `plug_ratelimit` or similar third-party library.

## Routing — In-Depth

### Router Pipeline

```elixir
defmodule MyAppWeb.Router do
  use MyAppWeb, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_live_flash
    plug :put_root_layout, {MyAppWeb.Layouts, :root}
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", MyAppWeb do
    pipe_through :browser
    get "/", PageLive, :index
    live "/users", UserLive.Index, :index
    resources "/users", UserController
  end

  scope "/api", MyAppWeb do
    pipe_through :api
    resources "/users", UserController, only: [:index, :show, :create]
  end
end
```

### Named Routes

```elixir
# In HEEx templates
<.link href={~p"/users/#{user}"}>User</.link>
<.link href={~p"/users/new"}>New User</.link>

# In Elixir code
MyAppWeb.Router.Helpers.user_path(MyAppWeb.Endpoint, :show, user)
```

## Common Libraries

### Database
- `ecto` — ORM (included)
- `postgrex` — PostgreSQL adapter (included)
- `ecto_sql` — SQL migrations (included)

### API
- `jason` — JSON parser (included)

### Auth
- `argon2_elixir` / `bcrypt_elixir` — Password hashing
- `comeonin` — Cross-library interface

### Real-time
- `phoenix_pubsub` — Built-in
- `phoenix_live_dashboard` — Built-in
- `phoenix_live_reload` — Built-in

### HTTP Client
- `httpoison` — HTTP client
- `hackney` — HTTP client (alternative)

### Scheduling
- `quantum` — Cron-style scheduling
- `timex` — Date/time manipulation

### Background Jobs
- `oban` — Job queue
- `exq` — Redis-backed jobs (Sidekiq-compatible)
