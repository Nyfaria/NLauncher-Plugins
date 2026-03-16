# How to List Your Server on NLauncher

## Overview

Server owners can list their Minecraft server in NLauncher's **Public Server Browser** by submitting a pull request to the [NLauncher-Plugins](https://github.com/Nyfaria/NLauncher-Plugins) repository.

## Requirements

- Your server **must** be running the [NLauncher Server Sync](https://github.com/Nyfaria/NLauncherAPI) companion mod
- The sync port (default `25580`) must be accessible to players
- The server must be publicly accessible (no whitelist-only sync port)
- The server must have been online for at least 1 week
- Icon must be appropriate (no NSFW, no copyrighted material without permission)
- Description must be accurate and family-friendly

## Steps

### 1. Fork the Repository

Fork [Nyfaria/NLauncher-Plugins](https://github.com/Nyfaria/NLauncher-Plugins) on GitHub.

### 2. Create Your Server Folder

Create a new folder under `Servers/` with your server's ID. The folder name should be lowercase with hyphens:

```
Servers/
└── my-awesome-server/
    ├── server.json
    └── icon.png
```

### 3. Create `server.json`

```json
{
    "id": "my-awesome-server",
    "name": "My Awesome Server",
    "description": "A friendly modded survival server with 200+ mods",
    "address": "play.example.com",
    "gamePort": 25565,
    "syncPort": 25580,
    "website": "https://example.com",
    "discord": "https://discord.gg/example",
    "owner": "YourName",
    "tags": ["survival", "modded", "neoforge", "1.21.1"],
    "minecraftVersion": "1.21.1",
    "modLoader": "neoforge",
    "featured": false
}
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique ID — must match folder name |
| `name` | string | ✅ | Server display name |
| `description` | string | ✅ | Short description (1–2 sentences) |
| `address` | string | ✅ | Server hostname or IP |
| `gamePort` | int | ❌ | Minecraft game port (default `25565`) |
| `syncPort` | int | ❌ | NLauncher sync port (default `25580`) |
| `website` | string | ❌ | Server website URL |
| `discord` | string | ❌ | Discord invite URL |
| `owner` | string | ✅ | Server owner name or GitHub username |
| `tags` | string[] | ❌ | Searchable tags (version, loader, gamemode) |
| `minecraftVersion` | string | ✅ | Minecraft version the server runs |
| `modLoader` | string | ✅ | `neoforge`, `forge`, `fabric`, `quilt`, or `vanilla` |
| `featured` | bool | ❌ | Set by repo maintainers only |

### 4. Add `icon.png`

Add a server icon as `icon.png` in your server folder:

- **Size**: 64×64 or 128×128 pixels
- **Format**: PNG
- Keep file size reasonable (under 100KB)

### 5. Open a Pull Request

Open a PR to the `main` branch of `Nyfaria/NLauncher-Plugins`. A maintainer will review and merge it.

## What Happens Next

Once merged, your server will appear in NLauncher's **Public Server Browser** (Create Instance → Server Instance → Public tab). Players will see:

- Your server name, description, and icon
- Live online/offline status with player count and latency
- Minecraft version and mod loader
- One-click "Add Server" to create a synced instance

## Updating Your Listing

To update your server info (description, address, tags, etc.), submit another PR editing your `server.json`.

## Removing Your Listing

To remove your server, submit a PR deleting your server folder from `Servers/`.

