# RUN.game CLI

A command-line interface tool for managing HTML5 games on the RUN.game platform. This CLI helps developers create, update, and manage their games efficiently.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Commands](#commands)
  - [login](#login)
  - [init](#init)
  - [deploy](#deploy)
  - [list-games](#list-games)
  - [download-docs](#download-docs)
  - [update](#update)
  - [game](#game-commands)
  - [generate](#generate-commands)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)

## Installation

### Quick Install (Recommended)

#### macOS/Linux

Install RUN.gameCLI with a single command:

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

After installation, verify that RUN.gameCLI is installed correctly:

```bash
rundot --help
```

You should see the list of available commands and options.

## Quick Start

### Publishing a NEW game to RUN.game

```bash
# 1. Login to RUN.game(required for authentication)
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

Before using the CLI, you need to authenticate with your RUN.gameaccount:

```bash
rundot login
```

This will open a browser window for you to sign in with your RUN.gamecredentials. Your session will be saved locally in `~/.rundot/` (or `%APPDATA%\.rundot\` on Windows) and automatically refreshed when needed.

You can also authenticate using an API key (`--api-key`) or a refresh token (`--refresh-token`) as alternatives to browser-based authentication.

**Login Options:**

- `--api-key`: API key for authentication (alternative to browser-based auth)
- `--refresh-token`: Refresh token for direct authentication
- `--env`: Specify the environment to login to

**Note:** Your credentials are stored securely on your local machine. The CLI never stores your password directly.

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

The RUN.gameCLI stores configuration data in the following locations:

- **Session data**: `~/.rundot_cli/` (macOS/Linux) or `%USERPROFILE%\.rundot_cli\` (Windows)
- **Game configuration**: `game.config.prod.json` in your project directory

## Commands

### login

Authenticate with your RUN.gameaccount. Supports three authentication methods: browser-based (default), API key, or refresh token.

```bash
rundot login
```

**Options:**

- `--api-key`: API key for authentication (alternative to browser-based auth)
- `--refresh-token`: Refresh token for direct authentication
- `--env`: Specify the environment to login to

**What it does:**

1. Authenticates via browser, API key, or refresh token
2. Saves your session locally in `~/.rundot_cli/`
3. Automatically refreshes your session when it expires

**Note:** You need to login before using commands like `init`, `deploy`, and `list-games`.

### init

Initializes a new game on RUN.game. This is the first command you should run when setting up a new game.

```bash
rundot init
```

**Options:**

- `--name`: The name of your game
- `--description`: Description of your game
- `--build-path`: Path to your game's distribution/build folder
- `--uses-preloader`: Whether the game uses the RUN.gameSDK
- `--override`: Should override old game config file if it exists
- `--env`: Environment to create the game in

**What it does:**

1. Prompts for game details (name, description, build path) if not provided
2. Creates a new game on RUN.game
3. Creates a `game.config.prod.json` file in your current directory with the game ID and settings

**Interactive Mode:**
If you don't provide options, the CLI will prompt you for:

- Game Name
- Game Description
- Path to game build folder (default: `./dist`)
- Whether your game uses the RUN.gameSDK

### deploy

Deploys a new version of your game. This is the main command for publishing updates.

```bash
rundot deploy
```

**Options:**

- `--game-id`: The game ID to deploy (reads from `game.config.prod.json` if not provided)
- `--build-path`: Path to your game's distribution/build folder
- `--bump`: Version bump type - `major`, `minor`, or `patch` (default: `minor`)
- `--uses-preloader`: Whether the game uses the RUN.gameSDK
- `--public`: Make this version visible on the explore page
- `--env`: Environment to deploy to

**Version Bumping:**

- `major`: 1.0.0 → 2.0.0 (breaking changes)
- `minor`: 1.0.0 → 1.1.0 (new features)
- `patch`: 1.0.0 → 1.0.1 (bug fixes)

**What it does:**

1. Zips your game distribution folder
2. Uploads the new version to RUN.gamestorage
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

Lists all your games on RUN.game.

```bash
rundot list-games
```

**Options:**

- `--env`: Environment to list games from

**Output includes:**

- Game ID
- Game name
- Current version
- Last update timestamp

### download-docs

Downloads the latest CLI and SDK documentation to a `.rundot-docs` folder in your current directory.

```bash
rundot download-docs
```

### update

Update the RUN.gameCLI to the latest version.

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

## Game Commands

Advanced commands for managing your game are available under the `game` subcommand.

### game create

Creates a new game on RUN.game. This is an alias for `init` under the `game` subcommand.

```bash
rundot game create
```

**Options:**

- `--name`: The name of your game
- `--description`: Description of your game
- `--build-path`: Path to your game's distribution/build folder
- `--uses-preloader`: Whether the game uses the RUN.gameSDK
- `--override`: Should override old game config file if it exists
- `--env`: Environment to create the game in

### game info

Prints detailed information about your game.

```bash
rundot game info
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to fetch info from

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
- `--uses-preloader`: Whether the game uses the RUN.gameSDK
- `--env`: Environment to use

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
- `--env`: Environment to update

### game set-description

Updates the description of your game.

```bash
rundot game set-description --description "New description"
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--description`: The new description for your game
- `--env`: Environment to update

### game list-versions

Lists all versions of your game.

```bash
rundot game list-versions
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to list versions from

### game upload-build

Uploads a new build of your game without updating tags. Use this for advanced workflows where you want to upload a version but configure tags separately.

```bash
rundot game upload-build
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--build-path`: Path to your game's distribution/build folder
- `--bump`: Version bump type - `major`, `minor`, or `patch` (default: `minor`)
- `--uses-preloader`: Whether the game uses the RUN.gameSDK
- `--env`: Environment to upload to

**Note:** After uploading, you'll need to run `rundot game update-tag` to make the version accessible.

### game list-server-configs

Lists all server configs for your game.

```bash
rundot game list-server-configs
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to list configs from

### game upload-server-config

Uploads a new server config for your game.

```bash
rundot game upload-server-config
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to upload to

### game list-runtime-configs

Lists all runtime configs for your game.

```bash
rundot game list-runtime-configs
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to list configs from

### game set-public

Sets your game visible on the explore page in RUN.game.

```bash
rundot game set-public
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--version`: Which version to set public (latest by default)
- `--env`: Environment to update

**What it does:**
Makes your game discoverable in search results and visible on your public profile. The share URL and a scannable QR code are printed in the terminal.

### game set-private

Hides your game from the explore page in RUN.game.

```bash
rundot game set-private
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to update

**Note:** The game will still be accessible via its share link, but won't appear in search results.

### game list-tags

Lists all tags for your game. For each tag, the share URL and a scannable QR code are shown in the terminal.

```bash
rundot game list-tags
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--tag`: Filter by specific tag
- `--env`: Environment to list tags from

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
- `--env`: Environment to update

### game delete-tag

Deletes a specific tag for your game.

```bash
rundot game delete-tag <tag-name>
```

**Arguments:**

- `tag-name`: Name of the tag to delete

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to delete from

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
- `--env`: Environment to use

### game list-editors

Lists the editors of your game.

```bash
rundot game list-editors
```

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to use

### game add-editors

Add people who can edit your game.

```bash
rundot game add-editors <emails>
```

**Arguments:**

- `emails`: Email addresses of the editors to add (space-separated)

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to update

### game remove-editors

Remove people who can edit your game.

```bash
rundot game remove-editors <emails>
```

**Arguments:**

- `emails`: Email addresses of the editors to remove (space-separated)

**Options:**

- `--game-id`: The game ID (reads from `game.config.prod.json` if not provided)
- `--env`: Environment to update

## Generate Commands

AI asset-generation commands live under the `generate` subcommand: images, audio
(music, SFX, text-to-speech), video, and sprites. These let you produce game-ready
assets from text prompts (and optional reference media) directly from the terminal.

```bash
rundot generate <kind> [options]
```

> **Beta:** `generate` and its subcommands are currently in beta and hidden from
> `rundot --help` by default. They are fully invokable; to make them appear in help
> output, set `RUNDOT_BETA_FEATURES=1` (or `true`) in your environment.

### Behavior shared by all generate commands

- **Auth required.** Run `rundot login` first. Generation runs against the active
  environment's venus-server; if that environment has no generation endpoint
  configured, switch with `rundot set-env <prod|staging|dev|local>`.
- **Game ID resolution.** `--game-id` is read from `game.config.prod.json` in the
  current directory when omitted. If neither is present, the command errors.
- **Output file.** The result is downloaded to `--out` if provided; otherwise a file
  name is derived from the prompt (e.g. `a-brave-knight.png`). Parent directories are
  created as needed.
- **Sidecar metadata.** A `<output>.json` sidecar is written next to each generated
  asset, recording the generation ID, prompt, model/provider, and other metadata.
- **`--json`.** Every command supports `--json` for machine-readable output (useful
  for scripting and agents).

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
- `--provider`: `seedance-2.0`, `seedance-2.0-fast`, or `kling-3.0-standard`. Default: `seedance-2.0`.
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

**Options:**

- `--prompt` (required): Text prompt for sprite generation.
- `--pixel`: Generate a pixel-art sprite.
- `--width`, `--height`: Sprite dimensions in pixels.
- `--bg`: Background color (e.g. `transparent`).
- `--smart-crop`: Auto-crop to content bounds. Default `true`; set `false` for exact dimensions.
- `--pixel-perfect`: Grid-aligned pixel post-processing. Default `true`; set `false` for exact dimensions.
- `--style`: Art style (e.g. `16-bit SNES`).
- `--model`: Model to use for generation.
- `--theme`: Visual theme hint.
- `--colors`: Comma-separated hex color palette (max 8).
- Reference slot (style hint, choose at most one): `--reference-asset-id`, `--reference-file-key`, or `--reference-file` (local image uploaded automatically).
- Edit slot (structure-preserving reskin anchor, choose at most one): `--edit-asset-id`, `--edit-file-key`, or `--edit-file` (local image uploaded automatically). The edit and reference slots may be combined.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
- `--json`: Machine-readable JSON output.

### generate animate-sprite

Animate an existing sprite into a spritesheet.

```bash
rundot generate animate-sprite --prompt "walk cycle, side view" \
  --source-generation-id <id> --frames 8 --format spritesheet
```

**Options:**

- `--prompt` (required): Animation prompt (e.g. `walk cycle, side view`).
- Source (exactly one required): `--source-generation-id`, `--source-file-key`, or `--source-url` (HTTPS).
- `--frames`: Number of animation frames.
- `--format`: Output format (e.g. `spritesheet`).
- `--remove-bg`: Background removal — `None`, `Basic`, or `Pro`. Default: `Basic`.
- `--game-id`: Game ID (reads from `game.config.prod.json` if not provided).
- `--out`: Output file path.
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

> **Related generation commands.** 3D-asset commands (`game generate-3d`,
> `game remesh-3d`, `game rig-3d`, `game animate-3d`) and image-utility commands
> (`image depth`, `image remove-bg`, `image upscale`) live outside the `generate`
> group and are not yet documented here.

## Usage Examples

### Example 1: Creating and Deploying a New Game

```bash
# Step 1: Login to RUN.game
rundot login

# Step 2: Initialize your game
rundot init
# Prompts for: Game Name, Description, Build Path, Uses RUN.gameSDK

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
