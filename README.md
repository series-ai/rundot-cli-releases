# RUN.world CLI

A command-line interface tool for managing HTML5 games on the RUN.world platform. This CLI helps developers create, update, and manage their games efficiently.

Looking for the SDK API docs? They ship inside the `@series-inc/rundot-game-sdk` npm package — run `npx rundot-sdk-setup` in your game project to install them (the CLI does not download docs).

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Progress output](#progress-output)
- [Commands](#commands)
  - [login](#login)
  - [init](#init)
  - [deploy](#deploy)
  - [list-games](#list-games)
  - [update](#update)
  - [skills](#skills-commands)
  - [ai setup](#ai-setup)
  - [game](#game-commands)
  - [generate](#generate-commands)
  - [intel](#intel-commands)
  - [socials](#social-launch-packet-commands)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)

## Installation

### Quick Install (Recommended)

#### macOS/Linux

Install RUN.world CLI with a single command:

```bash
curl -fsSL https://github.com/series-ai/rundot-cli-releases/releases/latest/download/install.sh | bash
```

The installer will:

- Automatically detect your OS and architecture (supports x64 and ARM64)
- Download the correct binary
- Install it to `~/.local/bin` (user directory, no sudo required)
- Make it executable and ready to use
- Automatically add the install directory to your PATH if needed

#### Windows (PowerShell)

```powershell
irm https://github.com/series-ai/rundot-cli-releases/releases/latest/download/install.ps1 | iex
```

The installer will:

- Automatically detect your architecture (supports x64 and ARM64)
- Download the correct binary
- Install it to `%LOCALAPPDATA%\Programs\Rundot`
- Optionally add the installation directory to your PATH (you'll be prompted)

#### Verify Installation

After installation, verify that RUN.world CLI is installed correctly:

```bash
rundot --help
```

You should see the list of available commands and options.

## Quick Start

### Publishing a NEW game to RUN.world

```bash
# 1. Login to RUN.world(required for authentication)
rundot login

# 2. Initialize a new game (creates game.config.prod.json automatically)
rundot init

# 3. Deploy your game
rundot deploy

# List your games
rundot list-games
```

## Configuration

### Authentication

Before using the CLI, you need to authenticate with your RUN.world account:

```bash
rundot login
```

This will open a browser window for you to sign in with your RUN.world credentials. Your session will be saved locally in `~/.rundot/` (or `%APPDATA%\.rundot\` on Windows) and automatically refreshed when needed.

You can also authenticate using an API key or a refresh token as alternatives to browser-based authentication. To create, list, regenerate, or revoke per-game API keys, see [`game api-keys`](#game-api-keys).

**Login Options:**

- `--api-key-stdin`: Read the API key from stdin (preferred for headless/CI — keeps the key out of shell history). Example: `printf '%s' "$RUNDOT_API_KEY" | rundot login --api-key-stdin`
- `--api-key`: **Discouraged.** Passes the API key as a command-line argument, which leaks it into shell history and the process list. Prefer `--api-key-stdin`.
- `--refresh-token`: **Discouraged.** Passes a long-lived refresh token on the command line (same shell-history leak).
- `--env`: Specify the environment to login to (Series-internal)

**Credential storage:** Your session is saved locally under `~/.rundot/` (or
`%APPDATA%\.rundot\` on Windows) and automatically refreshed when needed. The CLI
never stores your password directly.

### Game Configuration

The CLI uses a `game.config.prod.json` file to store your game's configuration:

```json
{
  "gameId": "your-game-id",
  "relativePathToBuildFolder": "./dist",
  "usesPreloader": false
}
```

This file is created automatically when you run `rundot init` and makes future deployments easier by storing your game ID and build path.

### Data Storage Locations

The RUN.world CLI stores configuration data in the following locations:

- **Session data**: `~/.rundot/` (macOS/Linux) or `%APPDATA%\.rundot\` (Windows)
- **Game configuration**: `game.config.prod.json` in your project directory
- **Project-local config & marketing**: the visible `rundot/` folder in your project directory (server config, `marketing/`, `cli_hooks.json`, `simulation/`). The legacy hidden `.rundot/` folder is still read; run `rundot migrate-config` to rename it to `rundot/`.

## Progress output

Long-running commands show an animated status with phase or percentage updates
when the terminal supports ANSI rendering. In redirected output, CI, agent
shells, or terminals such as `TERM=dumb`, `rundot` prints a plain elapsed-time
heartbeat every 10 seconds instead.

Progress is written to stderr in the plain fallback. Command results remain on
stdout, so redirects and machine-readable output such as `--json` can be
consumed without progress lines mixed into the payload.

## Commands

> **Note:** The `--env` flag is internal to Series and is hidden from external creators' `--help`.

### login

Authenticate with your RUN.world account. Supports three authentication methods: browser-based (default), API key, or refresh token.

```bash
rundot login
```

**Options:**

- `--api-key-stdin`: Read the API key from stdin (preferred — keeps it out of shell history). Mint one with [`rundot game api-keys create`](#game-api-keys).
- `--api-key`: **Discouraged** (shell-history leak). Pass the API key on the command line. Prefer `--api-key-stdin`.
- `--refresh-token`: **Discouraged** (shell-history leak). Refresh token for direct authentication.
- `--env`: Specify the environment to login to (Series-internal)

**What it does:**

1. Authenticates via browser, API key (stdin or flag), or refresh token
2. Saves your session locally under `~/.rundot/`
3. Automatically refreshes your session when it expires

**Note:** You need to login before using commands like `init`, `deploy`, `import`, and `list-games`.

### init

Initializes a new game on RUN.world. This is the first command you should run when setting up a new game.

```bash
rundot init
```

**Options:**

- `--name`: The name of your game
- `--description`: Description of your game
- `--build-path`: Path to your game's distribution/build folder
- `--uses-preloader`: Whether the game uses the RUN.world SDK
- `--override`: Should override old game config file if it exists
- `--env`: Environment to create the game in (Series-internal)

**What it does:**

1. Prompts for game details (name, description, build path) if not provided
2. Creates a new game on RUN.world
3. Creates a `game.config.prod.json` file in your current directory with the game ID and settings

**Interactive Mode:**
If you don't provide options, the CLI will prompt you for:

- Game Name
- Game Description
- Path to game build folder (default: `./dist`)
- Whether your game uses the RUN.world SDK

### import

Registers an existing project directory with RUN. For recognized Vite/npm web projects, optionally wires the RUN game SDK (dependency, Vite plugins, entry init) after explicit consent. Other project shapes register only and print guidance. Does not copy project files or invent import counts.

```bash
rundot import ./my-game --yes
rundot import ./my-game --name "Glacier Run" --yes
```

**Options:**

- `dir` (argument): Path to the existing project directory
- `--name`: Game name (defaults to `package.json` name when present)
- `--description`: Game description
- `--yes` / `-y`: Skip the confirmation prompt (required non-interactively)
- `--env`: Environment to create the game in (Series-internal)

**What it does:**

1. Detects whether the target is a constrained Vite/npm web project that can be auto-wired
2. Asks for consent, then creates a durable `rundot/.import-journal.json` with an idempotency key
3. Creates the remote game (retries reuse the same key; no duplicate games)
4. Applies planned SDK wiring when supported, then writes `game.config.<env>.json` atomically
5. Clears the journal after success

### deploy

Deploys a new version of your game. This is the main command for publishing updates.

```bash
rundot deploy
```

**Options:**

- `--game-id`: The game ID to deploy (reads from `game.config.prod.json` if not provided)
- `--build-path`: Path to your game's distribution/build folder
- `--bump`: Version bump type - `major`, `minor`, or `patch` (default: `minor`)
- `--uses-preloader`: Whether the game uses the RUN.world SDK
- `--public`: Make this version visible on the explore page
- `--env`: Environment to deploy to (Series-internal)

**Version Bumping:**

- `major`: 1.0.0 → 2.0.0 (breaking changes)
- `minor`: 1.0.0 → 1.1.0 (new features)
- `patch`: 1.0.0 → 1.0.1 (bug fixes)

**What it does:**

1. Zips your game distribution folder
2. Uploads the new version to RUN.world storage
3. Creates a new version entry for your game
4. Updates the `dev` tag to point to the new version
5. Optionally sets the version as public (visible in explore page)
6. Returns the share URL for both public and unlisted access, with a scannable QR code printed in the terminal

**Example:**

```bash
# Deploy with default settings (uses game.config.prod.json)
rundot deploy

# Deploy with a patch bump
rundot deploy --bump patch

# Deploy and make public immediately
rundot deploy --public
```

### list-games

Lists all your games on RUN.world.

```bash
rundot list-games
```

**Options:**

- `--env`: Environment to list games from (Series-internal)

**Output includes:**

- Game ID
- Game name
- Current version
- Last update timestamp

### update

Update the RUN.world CLI to the latest version.

```bash
rundot update
```

**Options:**

- `--beta`: Switch to the beta update channel
- `--stable`: Switch to the stable update channel

**What it does:**

1. Checks your current installed CLI version
2. Fetches the latest version available from GitHub releases
3. If a newer version is available, automatically downloads and installs it
4. If you're already on the latest version, informs you that no update is needed

**How it works:**

- Automatically detects your operating system (Windows, macOS, or Linux) and architecture (x64 or ARM64)
- Downloads the appropriate binary package for your platform
- Replaces your current installation with the new version
- Creates a backup of the old executable (.bak file)
- The update process is automatic and seamless

**Note:** The CLI automatically checks for updates when you run any command and will display a notification if a new version is available.

## Skills Commands

RUN ships a set of public AI skills (`SKILL.md` guides your coding agent reads —
Claude Code, Codex, Cursor, Pi) for building, deploying, and monetizing games.

The skills are **embedded in the `rundot` binary**, so there is nothing to fetch
and no separate version to track: `rundot update` updates your skills along with
the CLI. Install copies each skill into your agent's skills directory
(`.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.pi/skills/`), and a
ledger under `rundot/skills/installed.json` records a checksum of every file so
updates never overwrite edits you've made — your changes are always preserved
unless you pass `--force`.

The catalog has three classes of skill: platform workflows
(`rundot-deploy`, `rundot-monetization`, `rundot-marketing`), game-feature
implementations (`rundot-feature-*` — copy-in TypeScript templates with
integration guides, one skill per system), and references
(`rundot-sdk`, `rundot-new-game`).

Skill content lives under `venus_cli/Assets/Skills/` (one folder per skill +
`manifest.json`). The `systems/`, `shared/`, `starter/`, and
`references/run-sdk-notes.md` payloads inside the feature/new-game/sdk skills
are **generated** — vendored from the private
[run-game-helpers](https://github.com/series-ai/run-game-helpers) repo by
`node scripts/vendor-game-helpers.mjs <path-to-checkout>` (provenance:
`Assets/Skills/vendored-game-helpers.json`). Never edit vendored payload by
hand; change the source repo and re-run the script. `SKILL.md` files are
hand-authored and owned here.

**Syncing with run-game-helpers.** `vendored-game-helpers.json` pins the exact
source commit the payload was scraped from. CI can't reach the private repo, so
drift is checked on demand against a local checkout:

```bash
node scripts/vendor-game-helpers.mjs --check ../run-game-helpers   # what would change?
node scripts/vendor-game-helpers.mjs        ../run-game-helpers   # re-vendor + re-pin
```

`--check` compares the pin to the checkout's `HEAD`, lists the payload files
that changed between them, and exits non-zero on drift (0 when up to date) —
so it works in a pre-release script. It never writes anything. After a real
re-vendor, **re-read the file map in each affected skill's `SKILL.md`**: the
script deliberately doesn't touch `SKILL.md`, so a system that gains or loses a
file needs that table updated by hand.

Both modes also verify the script's `FEATURE_SYSTEMS` table (which skills carry
`shared/serverTime.ts` and `references/run-sdk-notes.md`) against what the source
actually imports, and refuse to run on a mismatch — otherwise a system that
starts importing the SDK upstream would silently ship without its SDK notes.

### ai setup

The one-command front door. Auto-detects which coding agents you use (from
`.claude/`, `.agents/`, `.cursor/`, `.pi/`, or `CLAUDE.md`) and installs all RUN
skills for them at project scope. Falls back to Claude if nothing is detected.

```bash
rundot ai setup            # confirm the detected agents, then install
rundot ai setup --yes      # install without prompting
```

### skills list

Lists every available skill and, per agent, its install state for the resolved
scope: `not-installed`, `installed` (matches the embedded version), `needs-update`
(an unmodified install from an older binary), or `modified` (you edited it).

```bash
rundot skills list
rundot skills list --scope global --json
```

### skills install

Installs a named skill (or all skills if omitted). With no `--agent`, it installs
for every detected agent, falling back to Claude when none are detected.

```bash
rundot skills install                          # all skills, detected agents
rundot skills install rundot-deploy            # one skill
rundot skills install --agent cursor --agent claude
rundot skills install --scope global           # install into your home dir
```

**Options:** `--agent <id>` (repeatable), `--scope project|global`, `--force`
(overwrite files you've edited), `--json`.

### skills update

Re-installs the embedded version over skills already tracked in the ledger,
preserving any files you've edited (unless `--force`). Never installs new skills —
an empty ledger is a no-op.

```bash
rundot skills update
rundot skills update rundot-deploy --agent claude
rundot skills update --force                    # take the embedded version everywhere
```

### skills uninstall

Removes tracked skill files. Files you've edited are left on disk and reported
(not deleted) unless `--force`.

```bash
rundot skills uninstall rundot-deploy --agent claude --yes
```

**Options:** `--agent <id>` (repeatable), `--scope project|global`, `--yes`,
`--force`, `--json`.

## Game Commands

Advanced commands for managing your game are available under the `game` subcommand.

### game create

Creates a new game on RUN.world. This is an alias for `init` under the `game` subcommand.

```bash
rundot game create
```

**Options:**

- `--name`: The name of your game
- `--description`: Description of your game
- `--build-path`: Path to your game's distribution/build folder
- `--uses-preloader`: Whether the game uses the RUN.world SDK
- `--override`: Should override old game config file if it exists
- `--env`: Environment to create the game in (Series-internal)

### game info

Prints detailed information about your game.

```bash
rundot game info
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to fetch info from (Series-internal)

**Output includes:**

- Game ID
- Name
- Description
- Created date
- Latest version
- Tags and their configurations

### game configure

Update the local game configuration file.

```bash
rundot game configure
```

**Options:**

- `--game-id`: The game ID
- `--build-path`: Path to your game's distribution/build folder
- `--uses-preloader`: Whether the game uses the RUN.world SDK
- `--env`: Environment to use (Series-internal)

**What it does:**
Creates or updates the `game.config.prod.json` file in your current directory.

### game set-name

Updates the name of your game.

```bash
rundot game set-name "New Game Name"
```

**Arguments:**

- `name`: The new name for your game

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to update (Series-internal)

### game set-description

Updates the description of your game.

```bash
rundot game set-description --description "New description"
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--description`: The new description for your game
- `--env`: Environment to update (Series-internal)

### game list-versions

Lists all versions of your game.

```bash
rundot game list-versions
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to list versions from (Series-internal)

### game upload-build

Uploads a new build of your game without updating tags. Use this for advanced workflows where you want to upload a version but configure tags separately.

```bash
rundot game upload-build
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--build-path`: Path to your game's distribution/build folder
- `--bump`: Version bump type - `major`, `minor`, or `patch` (default: `minor`)
- `--uses-preloader`: Whether the game uses the RUN.world SDK
- `--env`: Environment to upload to (Series-internal)

**Note:** After uploading, you'll need to run `rundot game update-tag` to make the version accessible.

### game list-server-configs

Lists all server configs for your game.

```bash
rundot game list-server-configs
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to list configs from (Series-internal)

### game upload-server-config

Uploads a new server config for your game.

```bash
rundot game upload-server-config
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to upload to (Series-internal)

### game list-runtime-configs

Lists all runtime configs for your game.

```bash
rundot game list-runtime-configs
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to list configs from (Series-internal)

### game set-public

Sets your game visible on the explore page in RUN.world.

```bash
rundot game set-public
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--version`: Which version to set public (latest by default)
- `--env`: Environment to update (Series-internal)

**What it does:**
Makes your game discoverable in search results and visible on your public profile. The share URL and a scannable QR code are printed in the terminal.

### game set-private

Hides your game from the explore page in RUN.world.

```bash
rundot game set-private
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to update (Series-internal)

**Note:** The game will still be accessible via its share link, but won't appear in search results.

### game list-tags

Lists all tags for your game. For each tag, the share URL and a scannable QR code are shown in the terminal.

```bash
rundot game list-tags
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--tag`: Filter by specific tag
- `--env`: Environment to list tags from (Series-internal)

### game update-tag

Updates or creates a tag for your game. Tags are used to point to specific versions with optional configurations.

```bash
rundot game update-tag <tag-name>
```

**Arguments:**

- `tag-name`: Name of the tag to update

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--version`: Version ID to point the tag to
- `--server-config-id`: Server config ID to use
- `--runtime-config-id`: Runtime config ID to use
- `--unset-version`: Unset the version ID
- `--unset-server-config-id`: Unset the server config ID
- `--env`: Environment to update (Series-internal)

### game delete-tag

Deletes a specific tag for your game.

```bash
rundot game delete-tag <tag-name>
```

**Arguments:**

- `tag-name`: Name of the tag to delete

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to delete from (Series-internal)

### game copy-tag

Copies a tag configuration to another tag.

```bash
rundot game copy-tag <source> <target>
```

**Arguments:**

- `source`: The source tag to copy from
- `target`: The target tag to copy to

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to use (Series-internal)

### game list-editors

Lists the editors of your game.

```bash
rundot game list-editors
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to use (Series-internal)

### game add-editors

Add people who can edit your game.

```bash
rundot game add-editors <emails>
```

**Arguments:**

- `emails`: Email addresses of the editors to add (space-separated)

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to update (Series-internal)

### game remove-editors

Remove people who can edit your game.

```bash
rundot game remove-editors <emails>
```

**Arguments:**

- `emails`: Email addresses of the editors to remove (space-separated)

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to update (Series-internal)

### game api-keys

Manage per-game API keys (`rk_` deploy keys) used for headless CI/CD auth with
`rundot login --api-key`. Key management requires an interactive owner login — it is
intentionally blocked when you are authenticated with an `rk_` key.

```bash
rundot game api-keys create --label "GitHub Actions" --expires-in-days 90
rundot game api-keys list
rundot game api-keys regenerate --key-id <KEY_ID>
rundot game api-keys revoke --key-id <KEY_ID>
```

**Subcommands:**

- `create`: Create a **deploy** key (`rk_<gameId>_<hex>`). Prints the secret **once** — save it immediately.
  - `--label`: Human-readable label
  - `--expires-in-days`: Days until expiry (1–365, default 365)
- `list`: List keys with id, label, type (deploy/playground), status (active/expired/revoked), and dates.
- `regenerate`: Revoke a key and issue a replacement secret **of the same type** in one step.
  - `--key-id` (required), `--yes` to skip confirmation
- `revoke`: Revoke a key immediately.
  - `--key-id` (required), `--yes` to skip confirmation

All subcommands accept `--game-id` (reads from `game.config.prod.json` if omitted) and `--env` (Series-internal).

#### Two key types: `rk_` (deploy) vs `pk_` (playground)

Keys are **prefix-typed** — the type is intrinsic to the key, not a setting:

- **`rk_` (deploy/CI)** — deploy, marketing, content-gen capable. Minted by
  `api-keys create`. Lives in `RUNDOT_API_KEY`.
- **`pk_` (playground)** — a low-blast-radius credential that can **only** open a
  headless playground-login session; rejected by every deploy/marketing/spend route,
  and the playground sign-in endpoint accepts only `pk_` keys. Minted by
  [`rundot playground grant-access`](#playground); lives in `RUNDOT_PLAYGROUND_KEY`.

A leaked `pk_` key can only create a wipeable playground session — it cannot spend
money or ship a build. Use it (never an `rk_` deploy key) for the headless dev-server
on-ramp.

### playground

The headless dev-server on-ramp. `grant-access` mints a **playground key** (a
`pk_…` key, label `headless-dev`, **365-day CLI default**) for the current game
and writes it to `.env.local` as `RUNDOT_PLAYGROUND_KEY=<secret>`. The SDK
playground plugin reads that key from `.env.local` so `npm run dev` signs in
headlessly with no `vite.config` editing. The key never leaves your dev server.
(It must run against the prod environment — playground sign-in is prod-federated.)

```bash
rundot playground grant-access                        # mint (365-day default) + write .env.local
rundot playground grant-access --expires-in-days 14   # shorter TTL (1–365)
rundot playground revoke-access                       # revoke server-side + strip the line
```

Options:

- `--expires-in-days`: Days until the key expires (1–365, **default 365**). Same
  range and default as `game api-keys create` — so CI secrets and local-dev keys
  both last a year without a re-mint unless you pass a shorter value.
- `--force`: Allow writing into a git-tracked `.env.local` (otherwise refused).

This writes a **durable secret to disk** — it is for deliberate headless use. The
default interactive Google sign-in in the dev toolbar needs no key at all; prefer
it when you can. `grant-access` refuses to write into a git-tracked `.env.local`
(use `--force` to override) and refuses to persist anything the server didn't hand
back as a `pk_` key (so a deploy-capable key never lands in the playground slot).

`revoke-access` reads the key from `.env.local` (refusing if it isn't a `pk_`
key), revokes it by the keyId recorded at grant time (falling back to an
unambiguous prefix match via `api-keys list`), then removes the line. The raw
secret can't be revoked directly — only hashed secrets are stored. A subsequent
`npm run dev` falls back to the toolbar Google sign-in.

Both subcommands accept `--game-id` (reads from `game.config.prod.json` if omitted) and `--env` (Series-internal).

### Campaign app deep links

Meta iOS and Android app legs use the game id and selected tag to build a
server-side `run.game://app` deferred target. Meta receives that value in its
deep-link field and receives the HTTPS app-store URL separately. Do not add a
deep-link string to `campaign.json`. Unity and Google use separate provider
paths. See [the deferred deep-link specification](../docs/marketing-deep-links.md).

## Unity Ads marketing

Unity is available when enabled. Create separate paused campaigns for iOS and
Android:

    rundot marketing prepare --name launch-ios --network unity --platforms ios --target-cpi 3.00

Unity requires exactly one square creative and one MP4. `marketing generate` creates
the MP4 from the prepared video brief after its `[[AGENT: ...]]` direction is filled.
Submit creates a paused campaign and waits for creative approval before it can be
launched. Use marketing status to see the provider, platform, campaign ID, and readiness.

## Generate Commands

AI asset-generation commands live under the `generate` subcommand: images, audio
(music, SFX, text-to-speech), video, and sprites. These let you produce game-ready
assets from text prompts (and optional reference media) directly from the terminal.

```bash
rundot generate <kind> [options]
```

> **Note:** `generate` and its subcommands are now generally available and appear
> in `rundot --help`. A few other command groups remain beta-gated — hidden until
> you set `RUNDOT_BETA_FEATURES=1` (or `true`) in your environment: `marketing`,
> `ugc`, `stats`, `collectibles`, and the `image` utility
> group (`image depth` / `remove-bg` / `upscale` / `turnaround`). The `image`
> utilities are also prod-only and require an interactive `rundot login` (they
> reject `rk_` API-key sessions). The `analytics` group is shown to everyone (not
> gated behind `RUNDOT_BETA_FEATURES`) but is still labeled **(beta)**.

### Behavior shared by all generate commands

- **Auth required.** Run `rundot login` first. Generation runs against the active
  environment's venus-server; if that environment has no generation endpoint
  configured, switch with `rundot set-env <prod|staging|dev|local>`.
- **Game ID is optional.** `--game-id` is read from `game.config.prod.json` in the
  current directory when omitted. **Every credit-spending command also works with
  no game ID at all** — the generation is then billed directly to the
  authenticated creator's credits instead of to the game. That covers `text`,
  `image`, `music`, `sfx`, `tts`, `video`, `sprite`, `animate-sprite`,
  `sprite-character-animate`, `sprite-jobs`, `character-workflows`,
  `design-voice`, `save-voice`, `list-voices`, `estimate`, the `image` utilities
  (`depth`, `remove-bg`, `upscale`, `turnaround`), and the 3D commands
  (`generate-3d`, `remesh-3d`, `rig-3d`, `animate-3d`).

  Two command families stay game-scoped by nature and still require a game ID:
  `game generate-thumbnail` (it *is* a game's thumbnail) and the `marketing`
  commands (campaigns belong to a game).
- **File-key inputs need a game.** Options that take a *file key*
  (`--image-file-key`, `--model-file-key`, `--reference-file-key`,
  `--edit-file-key`) resolve through the game-scoped files API, so they are
  rejected on a gameless call. Pass a direct URL — or, for sprites, an asset id —
  instead.
- **Where gameless assets live.** Gameless generations are stored under a
  per-creator partition rather than a game's, and their background jobs are
  polled/drained there too. This is transparent in normal use: a gameless
  `sprite-character-animate` is recoverable with a gameless `sprite-jobs --drain`.
  List and remove them with [`rundot assets`](#assets) — same `--game-id`
  resolution, so the command run in a game directory operates on that game.
- **Generated assets count against your storage quota.** Every generated image,
  sprite, audio clip, and 3D model is stored in RUN's bucket and counted against
  your creator storage cap — which is **global across all your games**, not
  per-game. Over the cap, generation is refused with a `429`. Free space with
  `rundot assets rm`; see what is using it with `rundot assets list`.
- **Server compatibility.** Gameless generation requires a venus-server that
  recognizes creator-direct billing on the target route. **Deploy the Cloud Run
  changes before distributing a CLI that emits gameless requests** — an older
  backend rejects them with `Game ID is required for this endpoint`.
- **Output file.** The result is downloaded to `--out` if provided; otherwise a file
  name is derived from the prompt (e.g. `a-brave-knight.png`). Parent directories are
  created as needed.
- **Sidecar metadata.** A `<output>.json` sidecar is written next to each generated
  asset, recording the generation ID, prompt, model/provider, and other metadata.
- **`--json`.** Every command supports `--json` for machine-readable output (useful
  for scripting and agents).
- **Credit-usage reporting.** On success, the image, audio (music/SFX/TTS), video,
  and sprite commands print the credits charged for the call and your remaining
  balance — e.g. `Used 800 credits · 47,200 remaining`. The remaining figure is
  omitted (`Used 800 credits`) when the balance can't be read, and no credit line
  appears for platform-funded generations. Under `--json`, the same data is on a
  `credits: { used, remaining }` object. See `rundot credits` for aggregate spend
  history and `rundot intel balance` for a standalone balance.

### generate estimate

Get an unbilled credit quote before generating. Pass the generation kind followed
by the options that affect its price:

```bash
rundot generate estimate image --model gemini-3-pro-image-preview --image-size 4K
rundot generate estimate music --duration 60
rundot generate estimate sprite --quantity 4
rundot generate estimate text --model claude-sonnet-4-6 --messages-file ./messages.json --max-tokens 2000
```

Fixed RUN-priced operations are labeled `exact at current pricing`. Variable-cost
operations print the likely estimate, a low–high credit range, the percentage
below/above the estimate, and why the final charge can vary. Use `--json` for the
same structured fields (`credits`, `lowCredits`, `highCredits`,
`lowerPercent`, `upperPercent`, `exact`, and `reason`). The command never runs a
model or debits credits.

For text generation, the CLI sends `--messages-file`, `--system`, and the output
token cap to the server. The server applies the same prompt-token estimator and
model pricing used by real generation preflight; the CLI contains no pricing or
token-estimation logic.

### generate image

Generate an image from a text prompt, with optional reference images and background
removal. Output defaults to `<prompt-slug>.png`.

```bash
rundot generate image --prompt "A brave knight in golden armor, fantasy game art"
```

**Options:**

- `--prompt` (required): Text prompt for image generation.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
- `--aspect-ratio`: Aspect ratio (e.g. `16:9`, `1:1`).
- `--model`: Model to use for generation (default: `gemini-3.1-flash-image-preview`).
- `--image-size`: Native output resolution `1K`, `2K`, or `4K` (only supported by `gemini-3-pro-image-preview`).
- `--negative-prompt`: Negative guidance text (max 1000 chars).
- `--seed`: Reproducibility seed.
- `--reference-image`: Reference image — local file path, HTTPS URL, data URI, or creator-storage key. Repeat up to 10 times. Local files are uploaded automatically.
- `--remove-background`: Remove background from the generated image (uses model defaults; pair with the `--remove-background-*` options to tune).
- `--remove-background-model`: `bria` (fast) or `birefnet` (high quality). Default: `bria`.
- `--remove-background-variant`: BiRefNet variant `light`, `heavy`, or `portrait` (only with `--remove-background-model birefnet`).
- `--remove-background-resolution`: BiRefNet resolution `1024x1024` or `2048x2048` (only with `--remove-background-model birefnet`).
- `--json`: Machine-readable JSON output.

### generate music

Generate music from a text prompt. Output defaults to `<prompt-slug>.mp3`.

```bash
rundot generate music --prompt "Upbeat 8-bit chiptune boss theme" --duration 30
```

**Options:**

- `--prompt` (required): Text prompt for music generation.
- `--duration` (required): Duration in seconds (3–300).
- `--provider`: Audio provider (default: `elevenlabs`).
- `--client-ref`: Opaque correlation ID echoed back in job events.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
- `--json`: Machine-readable JSON output.

### generate sfx

Generate a sound effect. Output defaults to `<description-slug>.mp3`.

```bash
rundot generate sfx --description "Glass shattering on stone, sharp and bright" --duration 2
```

**Options:**

- `--description`: Description of the sound effect (materials, intensity, context). Either `--description` or `--prompt` is required.
- `--prompt`: **Deprecated** — use `--description`. If both are given, `--description` wins and `--prompt` is ignored.
- `--duration`: Duration in seconds (0.5–30).
- `--provider`: Audio provider (default: `elevenlabs`).
- `--client-ref`: Opaque correlation ID echoed back in job events.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
- `--json`: Machine-readable JSON output.

### generate video

Generate a video from a text prompt. Supports text-to-video, image-to-video, and
reference-to-video across multiple providers. Output defaults to `<prompt-slug>.mp4`.

```bash
rundot generate video --prompt "A spaceship flying through an asteroid field" \
  --provider seedance-2.0 --duration 6 --resolution 720p
```

**Options:**

- `--prompt` (required): Text prompt for video generation.
- `--provider`: `seedance-2.0`, `seedance-2.0-fast`, `seedance-2.0-v2-fast` (enterprise/v2 fast tier), or `kling-3.0-standard`. Default: `seedance-2.0`.
- `--mode`: `text-to-video`, `image-to-video`, or `reference-to-video`. Default: `text-to-video`.
- `--duration`: Duration in seconds. Seedance: 4–15. Kling: 3–15.
- `--seed`: Reproducibility seed.
- `--negative-prompt`: Negative guidance text (Kling only).
- `--aspect-ratio`: e.g. `16:9`, `9:16`. Seedance also supports `21:9`, `4:3`, `3:4`. Kling: `16:9`, `9:16`, `1:1`.
- `--resolution`: `480p`, `720p`, or `1080p` (Seedance only).
- `--generate-audio`: Generate accompanying audio.
- `--camera-fixed`: Fix the camera position (Seedance only).
- `--cfg-scale`: Classifier-free guidance scale (Kling only).
- `--shot-type`: `customize` or `intelligent` (Kling only).
- `--start-image-url`: Starting frame image URL (HTTPS). **Required** for `--mode image-to-video`.
- `--end-image-url`: Ending frame image URL (HTTPS; Seedance/Kling support varies).
- `--image-reference`: Style/content reference image (HTTPS URL). Repeat up to 9 times (Seedance reference-to-video).
- `--video-reference`: Motion reference video (HTTPS URL). Repeat up to 3 times (Seedance reference-to-video).
- `--audio-reference`: Voice-cloning audio reference (HTTPS URL, 3–5s each, total ≤ 15s). Repeat up to 3 times.
- `--multi-prompt`: Kling-only multi-prompt segment in the form `prompt=<text>,duration=<seconds>`. Repeat for multiple segments.
- `--client-ref`: Opaque correlation ID echoed back in job events.
- `--request-origin`: Origin tag for analytics/audit.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
- `--json`: Machine-readable JSON output.

### generate sprite

Generate a game sprite (optionally pixel art) with style references and reskin
support. Output defaults to `<prompt-slug>.png`.

```bash
rundot generate sprite --prompt "A cute slime enemy, side view" --pixel --width 64 --height 64
```

Generate several candidates in one call with `--variations` (each file gets its
own `.json` sidecar, and `--json` emits an `assets` array):

```bash
rundot generate sprite --prompt "A cute slime enemy, side view" --variations 3 --out hero-{n}.png
```

Generate a texture-oriented tile candidate:

```bash
rundot generate sprite --prompt "Mossy cobblestone ground" --tileable
```

**Options:**

- `--prompt` (required): Text prompt for sprite generation.
- `--pixel`: Generate a pixel-art sprite.
- `--width`, `--height`: Sprite dimensions in pixels.
- `--bg`: Background color (e.g. `transparent`).
- `--smart-crop`: Auto-crop to content bounds. Default `true`; pass `--smart-crop false` or `--no-smart-crop` for exact dimensions.
- `--pixel-perfect`: Grid-aligned pixel post-processing. Default `true`; set `false` for exact dimensions.
- `--style`: Art style (e.g. `16-bit SNES`).
- `--model`: Model to use for generation.
- `--theme`: Visual theme hint.
- `--colors`: Comma-separated hex color palette (max 8).
- `--palette-file`: Reusable comma- or whitespace-separated hex palette file (max 8); mutually exclusive with `--colors`.
- `--variations`: Number of candidate images per call (1–4). With 2+ variations, `--out` must contain a `{n}` placeholder (1-based index, e.g. `hero-{n}.png`); when `--out` is omitted, `-{n}` is inserted before the extension of the derived name.
- `--mode`: Generation mode — `assets`, `texture`, or `ui`. Texture mode does not guarantee seamless edges.
- `--tileable`: Requests texture-oriented output; alias for `--mode texture`. Verify seams after generation.
- `--resolution`: Output resolution — `1K`, `2K`, or `4K`.
- `--quality`: Generation quality — `low`, `medium`, or `high`.
- `--aspect-ratio`: Aspect ratio — `1:1`, `16:9`, or `9:16`.
- Reference slot (style hint, choose at most one): `--reference-asset-id`, `--reference-file-key`, or `--reference-file` (local image uploaded automatically).
- Edit slot (structure-preserving reskin anchor, choose at most one): `--edit-asset-id`, `--edit-file-key`, or `--edit-file` (local image uploaded automatically). The edit and reference slots may be combined.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path (supports a `{n}` placeholder).
- `--json`: Machine-readable JSON output.

### generate animate-sprite

Animate an existing sprite into a spritesheet.

```bash
rundot generate animate-sprite --prompt "walk cycle, side view" \
  --source-generation-id <id> --frames 8 --format spritesheet
```

Steer the animation away from artifacts and control the alpha matte:

```bash
rundot generate animate-sprite --prompt "walk cycle, side view" \
  --source-generation-id <id> --negative-prompt "blurry, extra limbs" --matte-color "#00ff00"
```

**Options:**

- `--prompt` (required): Animation prompt (e.g. `walk cycle, side view`).
- Source (exactly one required): `--source-generation-id`, `--source-file-key`, or `--source-url` (HTTPS).
- `--frames`: Number of animation frames.
- `--format`: Output format (e.g. `spritesheet`).
- `--remove-bg`: Background removal — `None`, `Basic`, or `Pro`. Default: `Basic`.
- `--pixel`: Force pixel-art animation mode. When omitted, SpriteCook infers the mode from the source asset.
- `--palette-size`: Pixel-animation palette size, such as `16` or `32`. This controls color count; the animation API does not accept a fixed hex palette.
- `--negative-prompt`: Negative guidance text for the animation.
- `--matte-color`: Hex color (`#RRGGBB`) alpha is matted against during animation/background removal. Provider default: `#808080`.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
- `--json`: Machine-readable JSON output.

### generate sprite-character-animate

Animate a sprite into a full animation set using character workflow presets
(idle, walk, jump, ...). Purpose-built for complete character pipelines: each
animation reports its own result, so one failure does not void the run. Writes
one spritesheet + `.json` sidecar per completed animation into `--out`.

```bash
rundot generate sprite-character-animate --game-id <id> \
  --source-generation-id <id> --animations idle,walk --out ./character
```

**Options:**

- Source (exactly one required): `--source-generation-id`, `--source-file-key`, or `--source-url` (HTTPS).
- `--prompt`: Character description (guides the animation model). Defaults to the source generation's prompt when available.
- `--animations` (required): Comma-separated animation preset names (see `generate character-workflows`).
- `--workflow`: Workflow/perspective ID — `platformer` (default), `isometric`, or `topdown` (see `generate character-workflows`).
- `--frames`: Frame count per animation.
- `--format`: Output format (e.g. `spritesheet`).
- `--remove-bg`: Background removal — `None`, `Basic`, or `Pro`.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output directory.
- `--json`: Machine-readable JSON output with per-animation status.

### generate character-workflows

List the available character animation presets (workflow IDs and their
animation names).

```bash
rundot generate character-workflows --game-id <id> --json
```

### generate sprite-models

List sprite-generation models and their per-call credit pricing, as reported
by the provider. Model slugs and pricing change over time — query them instead
of hardcoding.

```bash
rundot generate sprite-models --json
```

### generate sprite-costs

Show per-operation sprite-generation credit costs without a billed call. Use
this to preflight the cost of a batch.

```bash
rundot generate sprite-costs --json
```

Returns credits per `generate` call (each variation bills as one), per
`animate` call, and per completed animation in a character-animate run.

### generate sprite-jobs

Drain completed/failed sprite-generation jobs (e.g. a character-animate whose
CLI invocation was killed while the job finished server-side). Results stay
available until the job expires; draining is non-destructive.

```bash
rundot generate sprite-jobs --drain --game-id <id> --json

# Also download the outputs and write the same sidecars the original
# command would have written:
rundot generate sprite-jobs --drain --game-id <id> --download ./recovered
```

**Options:**

- `--drain` (required): Fetch completed/failed sprite-generation jobs.
- `--download <dir>`: Download job outputs into the directory + write `.json` sidecars.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--json`: Machine-readable JSON output.

### generate tts

Generate text-to-speech audio with a chosen voice. Output defaults to
`tts-<generationId>.mp3`.

```bash
rundot generate tts --text "Welcome, adventurer!" --voice-id <voice-id>
```

**Options:**

- `--text` (required): Text to synthesize.
- `--voice-id` (required): Voice ID to use (from `generate list-voices` or `generate save-voice`).
- `--model`: TTS model `eleven_v3` or `eleven_multilingual_v2`. Default: `eleven_v3`.
- `--provider`: Audio provider (default: `elevenlabs`).
- `--stability`: Voice stability 0–1. Lower = more expressive. Default: `0.5`.
- `--similarity-boost`: Voice similarity boost 0–1. Default: `0.8`.
- `--speed`: Speech speed 0.5–2.0. Default: `1.0`.
- `--client-ref`: Opaque correlation ID echoed back in job events.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
- `--json`: Machine-readable JSON output.

### generate list-voices

List the TTS voices available to your game.

```bash
rundot generate list-voices
```

**Options:**

- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--json`: Machine-readable JSON output.

**Output:** A table of `Voice ID`, `Name`, and `Category`. Use a voice ID with
`generate tts --voice-id`.

### generate design-voice

Design custom voice candidates from a text description. Returns temporary voice IDs
plus preview audio URLs; persist one with `generate save-voice`.

```bash
rundot generate design-voice --description "A warm, gravelly old wizard"
```

**Options:**

- `--description` (required): Voice description (e.g. `A warm female voice`).
- `--sample-text`: Sample text for the voice preview (auto-generated if omitted).
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--json`: Machine-readable JSON output.

### generate save-voice

Persist a designed voice so it can be used for TTS. Returns a permanent voice ID.

```bash
rundot generate save-voice --generated-voice-id <temp-id> --voice-name "Wizard"
```

**Options:**

- `--generated-voice-id` (required): Temporary voice ID from a previous `design-voice` call.
- `--voice-name` (required): Display name for the saved voice (1–100 chars).
- `--voice-description`: Optional description (up to 1000 chars).
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--json`: Machine-readable JSON output.

### generate text

Run an LLM chat completion from a messages file. Result is written to **stdout**
(there is no `--out`). `--game-id` is optional: with a game ID (flag or
`game.config.prod.json`) the completion is billed to the game; without one the
completion is billed directly to the authenticated creator's credits.

```bash
rundot generate text --model claude-sonnet-4-5 --messages-file ./messages.json
```

The messages file is a JSON `AiMessage[]` array.

**Options:**

- `--model` (required): Model identifier (see `generate text-models`).
- `--messages-file` (required): Path to a JSON file containing an `AiMessage[]` array.
- `--system`: System prompt — a literal string, or `@path/to/file.txt` to read it from a file.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided). Optional — when neither resolves, the completion bills the authenticated creator directly.
- `--response-format`: `text` (default), `json_object`, or `json_schema`.
- `--schema-file`: Path to a JSON Schema file. **Required** with `--response-format json_schema`; rejected otherwise.
- `--strict-schema`: Enable strict schema adherence (only valid with `--response-format json_schema`).
- `--temperature`, `--top-p`, `--top-k`, `--max-tokens`, `--max-completion-tokens`: Sampling controls.
- `--tools-file`: Path to a JSON `Tool[]` array. `--tool-choice`: `auto`, `any`, `none`, or a tool name.
- `--stop`: Stop sequence (repeat up to 4×). `--seed`, `--presence-penalty`, `--frequency-penalty`, `--n` (1–10).
- `--logprobs` / `--top-logprobs`, `--user`, `--safety-identifier`, `--tag` (repeatable).
- `--stream`: Stream deltas to stdout via SSE.
- `--json`: Machine-readable JSON output.

### generate text-models

List the text-completion models available to your game.

```bash
rundot generate text-models          # one model id per line
rundot generate text-models --json   # JSON array
```

### `image turnaround` _(beta)_

Produce a new camera-angle view of a source image via the Qwen multi-angle
model (`fal-ai/qwen-image-edit-2511-multiple-angles`). The output preserves the
input image's aspect ratio. Beta-gated (set `RUNDOT_BETA_FEATURES=1`); prod-only;
requires an interactive `rundot login` session and a game id.

**Synopsis:**

```bash
rundot image turnaround --input <url|file|key> [--horizontal-angle N] [--vertical-angle N] [--num-images N] [--out path] [--force] [--game-id id]
```

**Options:**

- `--input` (required): HTTPS URL, app file key, or local file path of the input image.
- `--horizontal-angle`: Azimuth in degrees (0–360): `0`=front, `90`=right, `180`=back, `270`=left.
- `--vertical-angle`: Elevation in degrees (-30–90): `-30`=low-angle, `0`=eye-level, `90`=bird's-eye.
- `--num-images`: Number of variants to generate in one call (1–4, default 1).
- `--out`: Output path for the result (default: `{input-stem}_turnaround.png`). With `--num-images > 1`, results are written as `{stem}_1.png`, `{stem}_2.png`, … next to that path.
- `--force`: Overwrite existing output without prompting.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--env`: Environment — prod-only. (Series-internal)

At least one of `--horizontal-angle` / `--vertical-angle` is required (a
turnaround with no angle is a no-op). Cost scales with the input resolution
(priced per-megapixel) **and** with `--num-images` — N images are billed N× the
single-image cost.

**Example** — a 90° right-side view of a landscape background:

```bash
rundot image turnaround --input ./bg.png --horizontal-angle 90 --out ./bg_right.png --game-id my-game
```

**Example** — four angle variants in one call (billed 4×):

```bash
rundot image turnaround --input ./hero.png --horizontal-angle 45 --num-images 4 --out ./hero_angle.png --game-id my-game
```

### `image edit` _(beta)_

Apply a text-instructed edit to one or more source images via Seedream 5.0 Pro
Edit (`bytedance/seedream/v5/pro/edit`). Unlike `generate`, this model *requires*
input images — it cannot produce an image from a bare prompt. Beta-gated (set
`RUNDOT_BETA_FEATURES=1`); prod-only; requires an interactive `rundot login`
session. The game id is optional — omit it to bill the authenticated creator.

**Synopsis:**

```bash
rundot image edit --prompt <text> --input <url|file|key> [--input ...] [--image-size 1K|2K] [--out path] [--force] [--game-id id]
```

**Options:**

- `--prompt` (required): Instruction describing the edit to apply.
- `--input` (required, repeatable): HTTPS URL, app file key, or local file path of an input image. Up to 10; more than 10 is rejected. App file keys resolve through the game's files API, so they require `--game-id`; a gameless call rejects them with a 400.
- `--image-size`: Output resolution — `1K` (default) or `2K`. `2K` costs double per image.
- `--out`: Output path for the result (default: `{first-input-stem}_edit.png`).
- `--force`: Overwrite existing output without prompting.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided; omit for a creator-billed call).
- `--env`: Environment — prod-only. (Series-internal)

Each extra input image beyond the first adds a small surcharge, so pass only the
references the edit actually needs.

**Example** — restyle a single sprite:

```bash
rundot image edit --prompt "make it winter, add falling snow" --input ./bg.png --out ./bg_winter.png --game-id my-game
```

**Example** — compose using several references at 2K:

```bash
rundot image edit --prompt "put the character from image 1 into the scene from image 2" \
  --input ./hero.png --input ./scene.png --image-size 2K --out ./composed.png
```

> **Related generation commands.** 3D-asset commands (`game generate-3d`,
> `game remesh-3d`, `game rig-3d`, `game animate-3d`) and image-utility commands
> (`image depth`, `image remove-bg`, `image upscale`) live outside the `generate`
> group and are not yet documented here.

## Assets

Generated assets (images, sprites, audio, 3D models) are stored in RUN's bucket
and count against your **creator storage cap**, which is global across all your
games. `rundot assets` is how you see what is using that space and reclaim it.

Like the generate commands, `--game-id` is optional: omit it (and run outside a
game directory) to operate on your **gameless** creator assets; pass it — or run
inside a directory whose `game.config.*.json` names a game — to operate on that
game's.

### assets list

```bash
rundot assets list                        # your gameless assets, all services
rundot assets list --service imagegen     # one service
rundot assets list --game-id my-game      # a game's assets
rundot assets list --status removed       # already-removed entries
```

Sweeps all four generation services by default and prints each asset's id, size,
prompt, and creation time, followed by the total counted against your quota.
Use the printed id with `assets rm`.

### assets rm

```bash
rundot assets rm <generation-id> --service imagegen
rundot assets rm <generation-id> --service spritegen --game-id my-game
```

`--service` is required: each service partitions its generations separately, so
the id alone is ambiguous. Removal quarantines the stored object and **returns
its bytes to your storage quota**.

## Intel Commands

Competitive market intelligence for creators — rankings, modeled downloads,
Steam concurrency, ad-creative galleries, and full competitor dossiers.

**Access & pricing.** Intel is gated to paying creators (creator tier or
higher) and switchable via a server-side kill-switch. It's priced in RUN.world
credits (1,000 credits = $1):

| Action | Cost |
|---|---|
| `search`, `snapshot`, `whats-hot`, `balance` | Free (rate-limited) |
| `dossier` download of an existing dossier | A free weekly allowance per tier (creator 1 … max 5), then **1,000 credits** each |
| `dossier --generate` (build a dossier that doesn't exist yet) | **10,000 credits**, charged only on success |
| Weekly "What's Hot" email | Free with an active subscription |

Every command is **agent-operable**: each supports `--json` for structured
output, and non-interactive spending is flag-driven — no command ever charges
credits without an explicit `--confirm-spend` or `--generate` flag. In an
interactive terminal, `--generate` also asks for confirmation; pass `--yes` to
skip it. Without the flag, a paid action prints the structured cost (and your
balance) and exits non-zero.

### intel search

```bash
rundot intel search "Slay the Spire"     # match by game name
rundot intel search "Mega Crit" --json   # match by developer; JSON output
```

Lists matching games (by name **or** developer). Each hit is flagged whether a
dossier is `ready` to download or must be `generate`d. Free.

### intel snapshot

```bash
rundot intel snapshot "Slay the Spire"
rundot intel snapshot 553834731 --json
```

Free headline metrics: top rank, rating velocity, modeled downloads (`Est.`
band), Steam concurrent players and owners where listed, and active-creative
counts.

### intel dossier

```bash
# See every dossier that already exists and whether you own it:
rundot intel dossier list
rundot intel dossier list --limit 10 --offset 10 --json

# Preview contents and pricing without charging:
rundot intel dossier "Slay the Spire" --preview

# Free within your weekly allowance:
rundot intel dossier "Slay the Spire"

# Beyond the allowance, authorize the 1,000-credit overage:
rundot intel dossier "Slay the Spire" --confirm-spend

# If no dossier exists yet, authorize generating one (10,000 credits):
rundot intel dossier "Slay the Spire" --generate
rundot intel dossier "Slay the Spire" --generate --yes  # skip the interactive prompt
```

The full report: snapshot + qualitative teardown (signed, time-limited document
and screenshot URLs) + the UA creative gallery. Without `--confirm-spend` an
overage prints the cost and exits non-zero; without `--generate` a missing
dossier prints the generation cost and exits non-zero. The free weekly allowance
applies only to downloading a dossier that already exists; it cannot fund
generation. Generation is charged only once the dossier is downloadable. A
successful human-readable download prints a receipt showing either the free
coupon consumed and reset date or the credits charged and balance afterward.

### intel whats-hot

```bash
rundot intel whats-hot
```

The global "What's Hot" market digest (aggregate movers). Free.

### intel balance

```bash
rundot intel balance
```

Your credit balance, free allowance total/reset, existing dossier count, and any
pending generations. Pending generations show the credits committed to them and
are charged when each completes. The free allowance is explicitly for existing
dossier downloads only — it cannot be used to generate one.

### intel subscribe

```bash
rundot intel subscribe         # opt in to the weekly What's Hot email
rundot intel subscribe --off   # opt out
```

## Social Launch Packet Commands

_(beta)_ Generate platform-specific launch posts for a game, work the posting
checklist, and verify which steps are genuinely finished. All commands resolve the
game from the local game config or accept `--game-id`.

| Command | What it does |
|---|---|
| `rundot socials prepare` | Generate a launch packet (3 caption variants + a tracked link per platform) |
| `rundot socials status` | Show the posting checklist (`--json` for machine output) |
| `rundot socials next` | Show the next unposted platform to act on (`--json`) |
| `rundot socials open` | Print composer URL + copy text for a platform (`--json`) |
| `rundot socials promo` | Generate game-grounded platform promo image(s) and save every provider result |
| `rundot socials mark-posted` | Record a published post URL for a platform |
| `rundot socials verify` | Check which steps are **finished** |
| `rundot socials profile set` | Configure your creator social profile (Discord webhook, tone, hashtags, footer, CTAs) |
| `rundot socials profile show` | Show the current social profile (the webhook is reported as configured/not, never printed) |

### Configure your social profile

Your social profile is **per-creator, not per-game** — set it once and it applies
to every game you publish. It customizes the generated copy and enables Discord
auto-posting. It's optional (`prepare` works without it), but setting at least a
Discord webhook is recommended.

```bash
rundot socials profile set \
  --discord-webhook "https://discord.com/api/webhooks/…" \
  --discord-username "yourname" \
  --discord-role-ping "123456789012345678" \
  --tone "hyped but humble" \
  --hashtags "indiegame,h5games" \
  --cta "Play now,Drop a comment" \
  --footer "Made with RUN.world"
```

| Option | Purpose |
|---|---|
| `--discord-webhook <url>` | Discord incoming webhook. Stored as a write-only secret and used to auto-post. |
| `--discord-username <name>` | Your handle in RUN's community Discord. |
| `--discord-role-ping <roleId>` | Discord role id to ping on announcements. |
| `--tone <text>` | Copy tone, e.g. `"hyped but humble"`. |
| `--hashtags a,b,c` | Hashtags to weave into captions. |
| `--footer <text>` | Footer appended to copy. |
| `--cta "a,b"` | Preferred calls-to-action. |

Pass at least one option. Inspect the current profile any time with `rundot socials profile show`.

### When is a step finished?

A step counts as **finished** once it's both:

1. **Posted** — Discord auto-posts; the others are recorded via `mark-posted` (and,
   where possible, confirmed live via the platform).
2. **Clicked by someone who isn't you** — at least one click on the step's tracked
   link from a profile that isn't the creator's.

`verify` reports one of three states per platform:

- `not posted` — nothing posted yet.
- `awaiting click` — posted, but no non-creator click yet. **Not done** — it's
  waiting for 1 click that isn't you.
- `finished ✓` — posted and clicked by someone other than you.

TikTok and Instagram have no tracked link (search-only), so they can be
`mark-posted` but aren't finishable in this version.

```bash
# Human-readable table with a State column
rundot socials verify

# Verify a specific packet, machine-readable
rundot socials verify --packet <packetId> --json

# Verify a specific game
rundot socials verify --game-id my-game
```

## Storage Commands

Inspect and recover a player's `appStorage` (the per-player key-value store games
read and write at runtime). These commands operate on a single player's bucket,
identified by their profile ID.

All storage commands accept `--scope` (`app` default, or `owner` — the bucket
shared across every game the creator owns) and `--game-id` (reads from
`game.config.prod.json` if omitted).

### storage keys

List every key in a player's bucket.

```bash
rundot storage keys <profile-id> [--scope app|owner] [--save <file>]
```

### storage get

Read a single value.

```bash
rundot storage get <profile-id> <key> [--scope app|owner] [--format auto|json|raw] [--save <file>]
```

`--format` defaults to `auto`.

### storage set

Write a single key. The value is stored as a string.

```bash
rundot storage set <profile-id> <key> <value> [--scope app|owner]
```

### storage remove

Remove a single key. **Not** confirmation-gated — it runs immediately.

```bash
rundot storage remove <profile-id> <key> [--scope app|owner]
```

### storage clear

Wipe a player's entire bucket. **Destructive** — prompts for confirmation
(aborts in a non-interactive shell unless you pass `--yes`).

```bash
rundot storage clear <profile-id> [--scope app|owner] [--yes]
```

### storage usage

Show item/byte counts against quota for a player's bucket.

```bash
rundot storage usage <profile-id> [--scope app|owner] [--save <file>]
```

### storage export

```bash
rundot storage export (<profile-id> | --username <name>) [--game-id <id>] [--scope app|owner] [--as-of <iso>] [--save <file>]
```

Grabs a player's storage bucket and writes a portable snapshot JSON
(`{ env, profileId, gameId, scope, asOf, capturedAt, data }`). With `--save`, the
snapshot is written to a file and a summary is printed; without it, the snapshot
JSON is printed to stdout.

* Identify the player by `<profile-id>` **or** `--username <name>` (not both).
  `--username` resolves the name to its profile ID for you (same lookup as
  `rundot profile search`); it errors, listing candidates, when the name has no
  exact match or is ambiguous, so pass the ID directly in that case.

* `--scope` is `app` (default) or `owner`. **`owner` snapshots are
  export/inspect-only and cannot be imported** — the owner bucket is shared
  across every game the creator owns for that player.
* `--as-of <iso>` reads the bucket as it existed at a past minute via Firestore
  Point-In-Time Recovery (PITR). The reachable window is **the last 1 hour unless
  PITR is enabled on the database, in which case up to 7 days** (counting forward
  from when PITR was enabled). A timestamp outside that window returns a recovery
  window error.
* The snapshot file contains **player data (PII)** — store it somewhere
  gitignored and delete it when you are done.
* PITR data may include records the player later deleted. **Do not use `restore`
  to undo a player's erasure.**

### storage import

```bash
rundot storage import <profile-id> --file <snapshot> [--game-id <id>] [--yes]
```

Restores a snapshot's `data` into the target profile's `app` bucket via the
restore endpoint. This is destructive — it **replaces** all of the target
profile's `app` storage — so it prompts for confirmation unless `--yes` is
passed.

* Requires the **app owner** role (editors are rejected).
* Only `app`-scope snapshots are importable; `owner` snapshots are rejected.
* `import` refuses to proceed if the snapshot's `gameId` does not match the
  resolved target game, or if the snapshot's `env` does not exactly match the
  CLI's current environment (game IDs can be reused across environments).
* Restoring into a different `profile-id` than the snapshot's (the normal repro
  path — into a test profile you control) is allowed but warns first.

### storage data

```bash
rundot storage data <profile-id> [--game-id <id>] [--scope app|owner] [--as-of <iso>] [--format table|json|raw] [--save <file>]
```

Views a player's current storage. `--as-of <iso>` views a past version via PITR
without writing a snapshot file (same window and recovery-window behavior as
`storage export --as-of`).

## Usage Examples

### Example 1: Creating and Deploying a New Game

```bash
# Step 1: Login to RUN.world
rundot login

# Step 2: Initialize your game
rundot init
# Prompts for: Game Name, Description, Build Path, Uses RUN.world SDK

# Step 3: Deploy your game
rundot deploy

# Step 4: Make it public (optional)
rundot game set-public
```

### Example 2: Deploying Updates

```bash
# Make changes to your game...
# Build your game to ./dist

# Deploy with a patch bump (1.0.0 → 1.0.1)
rundot deploy --bump patch

# Deploy with a minor bump (1.0.1 → 1.1.0)
rundot deploy --bump minor

# Deploy with a major bump (1.1.0 → 2.0.0)
rundot deploy --bump major

# Deploy and make public immediately
rundot deploy --bump minor --public
```

### Example 3: Typical Development Workflow

```bash
# Initial setup
rundot login
rundot init

# First deployment
rundot deploy

# Iterate on your game...
# Build your changes

# Quick deploy (uses game.config.prod.json for all settings)
rundot deploy

# Check your game info
rundot game info

# View all versions
rundot game list-versions

# Check all your games
rundot list-games
```

### Example 4: Advanced Tag Management

```bash
# Upload a build without updating tags
rundot game upload-build --bump patch

# Update the 'dev' tag to point to the new version
rundot game update-tag dev --version 1.0.2

# List all tags
rundot game list-tags

# Delete a custom tag
rundot game delete-tag beta
```

### Example 5: Team Collaboration

```bash
# Add editors to your game
rundot game add-editors "teammate@example.com"

# Add multiple editors
rundot game add-editors "dev1@example.com dev2@example.com"

# Remove an editor
rundot game remove-editors "former-teammate@example.com"
```

## Kinetix Pack Build and Catalog (Beta)

The current beta builds, signs, stores, and catalogs data-only Kinetix packs;
native iOS mounting is a separate follow-up. Install
`@series-inc/rundot-syncplay` (which carries the deterministic core) in the game
project and use Git plus Node.js 22+:

```bash
rundot pack preflight --commit HEAD
rundot login --api-key-stdin < key.txt
rundot pack submit --version <versionId> --commit HEAD
rundot pack status --version <versionId>
```

Submit uploads a private temporary ZIP from `git archive` for the resolved full
commit SHA. Worktree modifications and untracked files are never included. Add
`--json` to any command for one compact stdout object; progress and errors remain
on stderr. Status exposes `queued`, `running`, `succeeded`, or `failed`, attempt
count, stable failure code, and successful artifact/runtime identity fields.

## Troubleshooting

### Common Issues

1. **"Session expired" or authentication errors**
   - Run `rundot login` to authenticate
   - Your session is automatically refreshed, but if you encounter issues, re-login

2. **"Failed to upload file" error**
   - Check your internet connection
   - Ensure you're logged in with `rundot login`
   - Verify the game distribution folder exists and is not empty

3. **"Game dist folder does not exist" error**
   - Verify the path to your game's build folder is correct
   - Ensure you're using the full path or correct relative path
   - Check that the path in `game.config.prod.json` is correct if using auto-detection

4. **"Unable to load game config" error**
   - Make sure you're running the command from the directory containing `game.config.prod.json`
   - Run `rundot init` to create a new game config
   - Or use `rundot game configure` to update an existing config
   - Verify the `game.config.prod.json` file is valid JSON

5. **"Game not found" error**
   - Ensure you've created the game using `rundot init` first
   - Verify the game ID in `game.config.prod.json` is correct
   - Check if you have access to edit this game with `rundot list-games`

6. **"No changes detected in build folder" warning**
   - This means your build folder hasn't changed since the last deploy
   - Make sure you've rebuilt your game before deploying

7. **Version conflicts**
   - When deploying, the version must be higher than the current version
   - Use appropriate bump type (major, minor, patch)

8. **PATH not updated after installation (macOS/Linux)**
   - The installer automatically adds `~/.local/bin` to your PATH
   - You may need to reload your shell: `source ~/.bashrc` (or `~/.zshrc` for zsh)
   - Or simply open a new terminal window
   - To verify: run `echo $PATH` and check if `~/.local/bin` is listed
   - If you used a custom install directory, make sure it's in your PATH

### Getting Help

- Check the command help: `rundot --help`
- Check specific command help: `rundot <command> --help` (e.g., `rundot login --help`, `rundot deploy --help`)
- Check game subcommand help: `rundot game <command> --help` (e.g., `rundot game info --help`)
- Review the error messages carefully - they often contain helpful information
- Make sure you're on the latest version by running `rundot update`
- Check the [GitHub releases](https://github.com/series-ai/rundot-cli-releases/releases) for changelogs and known issues

### Additional Tips

- **Always work from your project directory**: The CLI looks for `game.config.prod.json` in your current directory
- **Use `--help` liberally**: Every command has detailed help available with the `--help` flag
- **Keep the CLI updated**: Run `rundot update` regularly to get the latest features and bug fixes
- **Backup your game.config.prod.json**: Keep this file in version control for team collaboration
