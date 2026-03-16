# NLauncher Theme Development Guide

How to create custom themes for NLauncher. For publishing to the repository, see [PLUGIN_PUBLISHING](PLUGIN_PUBLISHING.md).

## Getting Started: Export the Default Theme

The easiest way to create a theme is to start from the default:

1. In NLauncher, go to **Settings → Themes**
2. Select the **Default** theme
3. Click **Export** to save the full JSON
4. Edit the colors to your liking
5. Change the `id`, `name`, `author`, and `description` fields

## Theme JSON Structure

```json
{
  "id": "my-cool-theme",
  "name": "My Cool Theme",
  "author": "Your Name",
  "version": "1.0.0",
  "description": "A sleek dark blue theme for NLauncher",
  "colors": {
    "windowBackground": "#0d1117",
    "panelBackground": "#161b22",
    "panelBackgroundAlt": "#1c2128",
    "panelBorder": "#30363d",
    "accentPrimary": "#58a6ff",
    "accentPrimaryHover": "#79c0ff",
    "accentSecondary": "#388bfd",
    "textPrimary": "#f0f6fc",
    "textSecondary": "#8b949e",
    "buttonPrimaryBackground": "#58a6ff",
    "buttonPrimaryBackgroundHover": "#79c0ff",
    "buttonPrimaryForeground": "#ffffff"
  }
}
```

## Theme Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | ✅ | Unique identifier (lowercase, hyphens). Must match the filename without `.json` |
| `name` | ✅ | Display name shown in the theme browser |
| `author` | ✅ | Your name |
| `version` | ✅ | Semver version string (bump this when updating) |
| `description` | ❌ | Short description of the theme |
| `colors` | ✅ | Object containing all color overrides |

## Color Categories

Themes can customize every color in the launcher. The main categories are:

| Category | Examples |
|----------|----------|
| **Window** | `windowBackground`, `windowOverlay` |
| **Panel** | `panelBackground`, `panelBackgroundAlt`, `panelBorder` |
| **Card** | `cardBackground`, `cardBackgroundHover`, `cardBorder` |
| **Accent** | `accentPrimary`, `accentPrimaryHover`, `accentSecondary`, `accentTertiary` |
| **Text** | `textPrimary`, `textSecondary`, `textMuted`, `textLink`, `textHeader` |
| **Buttons** | `buttonPrimaryBackground`, `buttonDangerBackground`, `buttonPlayBackground`, etc. |
| **Inputs** | `textBoxBackground`, `textBoxBorder`, `textBoxBorderFocus`, `dropDownBackground`, etc. |
| **Controls** | `checkBoxBackground`, `radioButtonBackground`, `sliderTrackFill`, etc. |
| **Lists** | `listBackground`, `listItemBackgroundHover`, `listItemBackgroundSelected` |
| **Context Menu** | `contextMenuBackground`, `contextMenuBorder`, `contextMenuItemBackgroundHover` |
| **Status** | `success`, `warning`, `danger`, `info` + background/text variants |
| **Icons** | `iconPrimary`, `iconSecondary`, `iconAccent`, etc. |
| **Borders** | `borderPrimary`, `borderSecondary`, `borderMuted`, `borderAccent` |
| **Logo** | `logoTopFace`, `logoLeftFace`, `logoRightFace`, `logoBackground` |

All colors use hex format: `#RRGGBB` or `#AARRGGBB` (with alpha).

## Testing Your Theme Locally

1. Copy your `.json` file to the themes folder:
   - **Windows:** `%APPDATA%\.nlauncher\themes\`
   - **Linux:** `~/.nlauncher/themes/`
   - **macOS:** `~/.nlauncher/themes/`
2. In NLauncher, go to **Settings → Themes** and click **Refresh**
3. Select your theme and verify all colors look good

## Next Steps

Ready to share your theme? See the [Publishing Guide](README.md) for submitting to the repository.

