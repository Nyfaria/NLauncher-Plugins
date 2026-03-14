# NLauncher Publishing Guide — Plugins & Themes

How to submit plugins and themes to the NLauncher repository for distribution.

- For creating plugins, see [PLUGIN_DEVELOPMENT.md](PLUGIN_DEVELOPMENT.md)
- For creating themes, see [THEME_DEVELOPMENT.md](THEME_DEVELOPMENT.md)

## Overview

Plugins and themes are hosted in the [Nyfaria/NLauncher-Plugins](https://github.com/Nyfaria/NLauncher-Plugins) GitHub repository. NLauncher's built-in browser fetches content directly from this repo via the GitHub API.

- **Plugins** live in `Plugins/{plugin-id}/` as raw files (DLLs + `plugin.json`)
- **Themes** live in `Themes/{theme-id}.json` as single JSON files

---

## Submitting Plugins

1. Fork [Nyfaria/NLauncher-Plugins](https://github.com/Nyfaria/NLauncher-Plugins)
2. Create a folder: `Plugins/my-plugin/`
3. Copy your DLLs and `plugin.json` into that folder
4. Submit a pull request

Your `plugin.json` must include a `repository` field pointing to a public GitHub repo — plugins without this are rejected by the browser.

### Updating Your Plugin

1. Update the `version` in `plugin.json`
2. Replace the DLLs with the new build
3. Submit a pull request

NLauncher detects updates by comparing the installed version against the repo's `plugin.json` version.

---

## Submitting Themes

1. Fork [Nyfaria/NLauncher-Plugins](https://github.com/Nyfaria/NLauncher-Plugins)
2. Add your theme JSON to the `Themes/` folder: `Themes/my-cool-theme.json`
3. Submit a pull request

Or use the **Import** button in NLauncher's theme browser and share the JSON file directly.

---

## Repository Structure

```
NLauncher-Plugins/
├── Plugins/
│   ├── discord-rpc/
│   │   ├── plugin.json
│   │   ├── DiscordRPC.dll
│   │   └── DiscordRPC.deps.json
│   ├── another-plugin/
│   │   ├── plugin.json
│   │   └── AnotherPlugin.dll
│   └── ...
├── Themes/
│   ├── blue.json
│   ├── my-cool-theme.json
│   └── ...
└── README.md
```

- **`Plugins/`** — Each subfolder is one plugin, containing its DLLs and `plugin.json`.
- **`Themes/`** — Theme JSON files. Each file is a complete theme.

Both are discovered directly via the GitHub API — no catalog or index file needed.

---

## Security

- Repository maintainers review all submissions before merging
- Plugins must include a `repository` field with a public GitHub URL for code review
- Themes are plain JSON with no executable code — safe by design

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Plugin not appearing in browser | Ensure `Plugins/{id}/plugin.json` exists and has a `repository` field |
| Theme not appearing in browser | Ensure the JSON is valid and has `id`/`name` fields |
| Theme colors look wrong | Export the default theme and compare — make sure all color keys are present |
| "Requires restart" after install | Plugin installs require an NLauncher restart to load |
