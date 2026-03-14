# NLauncher Plugin Development Guide

How to create plugins for NLauncher. For publishing and distribution, see [PLUGIN_PUBLISHING.md](PLUGIN_PUBLISHING.md).

## Quick Start

### 1. Initialize a New Plugin Project

```bash
nlauncher-plugin init MyAwesomePlugin
```

This creates:
- `MyAwesomePlugin/MyAwesomePlugin.csproj` - Project file
- `MyAwesomePlugin/plugin.json` - Plugin manifest
- `MyAwesomePlugin/MyAwesomePluginPlugin.cs` - Main plugin class

Or manually create a Class Library targeting .NET 8.0 and add the SDK:

```bash
dotnet new classlib -n MyAwesomePlugin -f net8.0
cd MyAwesomePlugin
dotnet add package NLauncher.PluginSDK
```

### 2. Implement IPlugin

```csharp
using NLauncher.PluginSDK.Interfaces;
using NLauncher.PluginSDK.Models;

public class MyPlugin : IPlugin
{
    private IPluginHost? _host;

    public PluginInfo Info => new()
    {
        Id = "my-plugin",
        Name = "My Plugin",
        Description = "Does something awesome",
        Version = "1.0.0",
        Author = "Your Name"
    };

    public PluginState State { get; private set; } = PluginState.Unloaded;

    public Task LoadAsync(IPluginHost host)
    {
        _host = host;
        _host.Logger.Info("My plugin loaded!");
        State = PluginState.Loaded;
        return Task.CompletedTask;
    }

    public Task UnloadAsync()
    {
        State = PluginState.Unloaded;
        return Task.CompletedTask;
    }

    public Task EnableAsync()
    {
        State = PluginState.Enabled;
        return Task.CompletedTask;
    }

    public Task DisableAsync()
    {
        State = PluginState.Disabled;
        return Task.CompletedTask;
    }

    public void Dispose() { }
}
```

### 3. Create plugin.json

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "Does something awesome",
  "author": "Your Name",
  "repository": "https://github.com/YourName/MyPlugin",
  "minLauncherVersion": "0.2.0",
  "entryPoint": "MyPlugin.dll",
  "mainClass": "MyPlugin.MyPlugin"
}
```

> **Note:** The `repository` field is **required**. All plugins must provide a public GitHub repository URL for security review.

### 4. Build and Test Locally

```bash
cd MyAwesomePlugin
dotnet build
```

Copy the build output to your plugins folder to test:

| Platform | Path |
|----------|------|
| Windows  | `%APPDATA%\.nlauncher\plugins\MyAwesomePlugin\` |
| Linux    | `~/.nlauncher/plugins/MyAwesomePlugin/` |
| macOS    | `~/.nlauncher/plugins/MyAwesomePlugin/` |

## Plugin Manifest Reference

| Field | Required | Description |
|-------|----------|-------------|
| `id` | ✅ | Unique plugin identifier (lowercase, hyphens) |
| `name` | ✅ | Display name |
| `version` | ✅ | Semver version string |
| `description` | ✅ | Short description |
| `author` | ✅ | Author name |
| `repository` | ✅ | Public GitHub repository URL |
| `minLauncherVersion` | ✅ | Minimum NLauncher version required |
| `entryPoint` | ✅ | Main DLL filename |
| `mainClass` | ✅ | Fully qualified class name implementing `IPlugin` |
| `permissions` | ❌ | Array of permission strings |
| `dependencies` | ❌ | Array of plugin dependency objects |

## IPlugin Lifecycle

```
LoadAsync() → EnableAsync() → [running] → DisableAsync() → UnloadAsync() → Dispose()
```

| Method | When Called |
|--------|------------|
| `LoadAsync(IPluginHost)` | Plugin is loaded, receive host services |
| `EnableAsync()` | Plugin is activated |
| `DisableAsync()` | Plugin is deactivated (can be re-enabled) |
| `UnloadAsync()` | Plugin is being removed |
| `Dispose()` | Final cleanup |

## IPluginHost Services

The `IPluginHost` provided in `LoadAsync` gives access to:

| Service | Description |
|---------|-------------|
| `Logger` | Log messages (`Debug`, `Info`, `Warn`, `Error`) |
| `Events` | Subscribe to launcher events |
| `InstanceService` | Access Minecraft instances |
| `DialogService` | Show dialogs, file pickers, notifications |
| `FileSystem` | Safe file operations |
| `HttpClient` | Pre-configured HTTP client |
| `CurrentTheme` | Current UI theme colors |
| `LauncherVersion` | NLauncher version string |
| `PluginDataDirectory` | Per-plugin persistent storage path |
| `IsDebugMode` | Whether launcher is in debug mode |

## UI Registration

### Instance Tab
```csharp
_host.RegisterInstanceTab(new TabRegistration
{
    Id = "my-tab",
    Title = "My Tab",
    ViewFactory = () => new MyTabView(),
    ViewModelFactory = () => new MyTabViewModel(_host)
});
```

### Main Tab
```csharp
_host.RegisterMainTab(new TabRegistration
{
    Id = "my-main-tab",
    Title = "My Main Tab",
    ViewFactory = () => new MyMainView(),
    ViewModelFactory = () => new MyMainViewModel()
});
```

### Settings Section
```csharp
_host.RegisterSettingsSection(new SettingsSectionRegistration
{
    Id = "my-settings",
    Title = "My Plugin Settings",
    ViewFactory = () => new MySettingsView()
});
```

### Context Menu
```csharp
_host.RegisterContextMenu(new ContextMenuRegistration
{
    Id = "my-context-action",
    Title = "Do Something",
    Location = ContextMenuLocation.Instance,
    Action = async (context) => { /* handle click */ }
});
```

### Toolbar Button
```csharp
_host.RegisterToolbarButton(new ToolbarRegistration
{
    Id = "my-toolbar-btn",
    Tooltip = "My Action",
    Icon = new TextBlock { Text = "🔧" },
    Location = ToolbarLocation.MainToolbar,
    Action = async () => { /* handle click */ }
});
```

## Events

Subscribe to launcher events via `IPluginHost.Events`:

```csharp
// Instance events
_host.Events.InstanceSelected.Subscribe(args =>
{
    _host.Logger.Info($"Selected: {args.Instance.Name}");
});

_host.Events.InstanceLaunched.Subscribe(args =>
{
    // Game started playing
});

_host.Events.InstanceClosed.Subscribe(args =>
{
    // Game exited
});

_host.Events.InstanceCreated.Subscribe(args => { });
_host.Events.InstanceDeleted.Subscribe(args => { });

// Mod events
_host.Events.ModInstalled.Subscribe(args => { });
_host.Events.ModUninstalled.Subscribe(args => { });
_host.Events.ModUpdated.Subscribe(args => { });
_host.Events.ModToggled.Subscribe(args => { });

// Theme events
_host.Events.ThemeChanged.Subscribe(args =>
{
    // Update your UI colors to match the new theme
    var theme = args.Theme;
});
```

## Stateful Plugins

Implement `IStatefulPlugin` to persist state across launcher restarts:

```csharp
public class MyStatefulPlugin : IStatefulPlugin
{
    // ...IPlugin members...

    public Task<Dictionary<string, object>?> GetStateAsync()
    {
        return Task.FromResult<Dictionary<string, object>?>(new Dictionary<string, object>
        {
            ["lastUsedProfile"] = "default",
            ["windowSize"] = 800
        });
    }

    public Task RestoreStateAsync(Dictionary<string, object> state)
    {
        if (state.TryGetValue("lastUsedProfile", out var profile))
        {
            // Restore your state
        }
        return Task.CompletedTask;
    }
}
```

## Dialogs

Use `IPluginDialogService` via the host:

```csharp
// Simple notification
await _host.DialogService.ShowNotificationAsync("Hello!", NotificationType.Info);

// Confirmation
bool confirmed = await _host.DialogService.ShowConfirmationAsync("Delete?", "Are you sure?");

// Text input
string? name = await _host.DialogService.ShowInputDialogAsync("Name", "Enter your name:");

// File/folder pickers
string? file = await _host.DialogService.ShowOpenFileDialogAsync("Select file", new[] { "*.json" });
string? folder = await _host.DialogService.ShowFolderDialogAsync("Select folder");

// Custom dialog
await _host.DialogService.ShowDialogAsync(myCustomControl, "Custom Dialog", width: 600, height: 400);
```

## Example Plugin

See the full example at [`Plugins/ExamplePlugin/`](Plugins/ExamplePlugin/) in the NLauncher source.

## Next Steps

Ready to share your plugin? See the [Publishing Guide](PLUGIN_PUBLISHING.md) for packaging and submitting to the repository.

