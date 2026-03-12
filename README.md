# NLauncher Plugin Development Guide

Complete guide for creating plugins for NLauncher using the Plugin SDK.

## Prerequisites

- .NET 8.0 SDK
- IDE (Visual Studio, Rider, or VS Code)
- NLauncher installed (for testing)

## Quick Start

### 1. Create a New Project

```bash
dotnet new classlib -n MyPlugin -f net8.0
cd MyPlugin
dotnet add package NLauncher.PluginSDK
```

### 2. Create the Plugin Class

Create `MyPlugin.cs`:

```csharp
using NLauncher.PluginSDK.Interfaces;
using NLauncher.PluginSDK.Models;

namespace MyPlugin;

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
        
        // Register your UI, subscribe to events, etc.
        
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

Create `plugin.json` in your project root:

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "description": "Does something awesome",
  "version": "1.0.0",
  "author": "Your Name",
  "repository": "https://github.com/YourName/MyPlugin",
  "minLauncherVersion": "0.2.0",
  "entryPoint": "MyPlugin.dll",
  "mainClass": "MyPlugin.MyPlugin",
  "permissions": [],
  "dependencies": []
}
```

> ⚠️ **Required:** The `repository` field must point to your public GitHub repo for security review.

### 4. Configure Build Output

Add to your `.csproj`:

```xml
<ItemGroup>
  <None Update="plugin.json">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

### 5. Build and Install

```bash
dotnet build -c Release
```

Copy the output to test:
```powershell
Copy-Item -Path "bin/Release/net8.0/*" -Destination "$env:APPDATA/.nlauncher/plugins/my-plugin/" -Recurse
```

Restart NLauncher to load your plugin.

---

## Plugin Host Services

The `IPluginHost` provides access to launcher functionality:

### Logger
```csharp
_host.Logger.Info("Information message");
_host.Logger.Warn("Warning message");
_host.Logger.Error("Error message");
```

### Events
```csharp
// Instance selected
_host.Events.InstanceSelected.Subscribe(args => {
    _host.Logger.Info($"Selected: {args.Instance.Name}");
});

// Game launched
_host.Events.GameLaunched.Subscribe(args => {
    _host.Logger.Info($"Launched: {args.Instance.Name}");
});

// Game closed
_host.Events.GameClosed.Subscribe(args => {
    _host.Logger.Info($"Closed: {args.Instance.Name}");
});

// Theme changed
_host.Events.ThemeChanged.Subscribe(args => {
    // Update UI colors from args.Theme
});
```

### Instance Service
```csharp
// Get currently selected instance
var instance = await _host.InstanceService.GetSelectedInstanceAsync();

// Get all instances
var instances = await _host.InstanceService.GetAllInstancesAsync();
```

### Dialog Service
```csharp
// Show message
await _host.DialogService.ShowMessageAsync("Title", "Message");

// Show confirmation
bool confirmed = await _host.DialogService.ShowConfirmAsync("Title", "Are you sure?");
```

### Theme
```csharp
var theme = _host.CurrentTheme;
if (theme != null)
{
    var bgColor = theme.BackgroundColor;
    var textColor = theme.TextColor;
    var accentColor = theme.AccentColor;
}
```

---

## Registering UI

### Add a Tab to Instance Panel

```csharp
public Task LoadAsync(IPluginHost host)
{
    _host = host;
    
    _host.RegisterInstanceTab(new TabRegistration
    {
        Id = "my-tab",
        Title = "My Tab",
        Order = 100,  // Lower = appears first
        ViewFactory = () => new MyTabView(),
        ViewModelFactory = () => new MyTabViewModel(_host)
    });
    
    State = PluginState.Loaded;
    return Task.CompletedTask;
}
```

### Add a Main Tab

```csharp
_host.RegisterMainTab(new TabRegistration
{
    Id = "my-main-tab",
    Title = "My Feature",
    Order = 50,
    ViewFactory = () => new MyMainView(),
    ViewModelFactory = () => new MyMainViewModel(_host)
});
```

### Add Settings Section

```csharp
_host.RegisterSettingsSection(new SettingsSectionRegistration
{
    Id = "my-settings",
    Title = "My Plugin Settings",
    Order = 100,
    ViewFactory = () => new MySettingsView(),
    ViewModelFactory = () => new MySettingsViewModel(_host)
});
```

---

## Creating Views (Avalonia)

Plugins use Avalonia for UI. You can create views in code or XAML.

### Code-based View

```csharp
using Avalonia.Controls;
using Avalonia.Layout;

public class MyTabView : UserControl
{
    public MyTabView()
    {
        Content = new StackPanel
        {
            Spacing = 10,
            Children =
            {
                new TextBlock { Text = "Hello from my plugin!" },
                new Button { Content = "Click Me" }
            }
        };
    }
}
```

### Using Theme Colors

```csharp
public class MyTabViewModel
{
    private readonly IPluginHost _host;
    
    public MyTabViewModel(IPluginHost host)
    {
        _host = host;
        
        // Subscribe to theme changes
        _host.Events.ThemeChanged.Subscribe(args => {
            // Update your UI colors
        });
    }
    
    public string BackgroundColor => _host.CurrentTheme?.BackgroundColor ?? "#1a0a20";
    public string TextColor => _host.CurrentTheme?.TextColor ?? "#ffffff";
}
```

---

## File Storage

Store plugin data in the designated directory:

```csharp
var dataPath = _host.PluginDataDirectory;
var configFile = Path.Combine(dataPath, "config.json");

// Save
await File.WriteAllTextAsync(configFile, json);

// Load
if (File.Exists(configFile))
{
    var json = await File.ReadAllTextAsync(configFile);
}
```

---

## Publishing Your Plugin

### 1. Push to GitHub

Your plugin source code must be in a public GitHub repository.

### 2. Build Release

```bash
dotnet build -c Release
```

### 3. Submit to Plugin Repository

1. Fork https://github.com/Nyfaria/NLauncher-Plugins
2. Create a folder with your plugin ID
3. Copy your built files:
   - `YourPlugin.dll`
   - `plugin.json`
   - Any dependencies
4. Submit a Pull Request

The PR will be automatically scanned for security issues.

---

## Best Practices

1. **Use reactive properties** for UI bindings
2. **Subscribe to theme changes** to keep your UI consistent
3. **Handle errors gracefully** - don't crash the launcher
4. **Clean up resources** in `UnloadAsync()` and `Dispose()`
5. **Log important events** using `_host.Logger`
6. **Test thoroughly** before publishing

---

## Example Plugins

- **TreePackEditor** - Full-featured editor plugin with multiple views
  - Location: `Plugins/TreePackEditor/`
  - Demonstrates: Tab registration, file editing, theme support

---

## API Reference

### IPlugin Interface

| Method | Description |
|--------|-------------|
| `LoadAsync(IPluginHost)` | Called when plugin is loaded |
| `UnloadAsync()` | Called when plugin is unloaded |
| `EnableAsync()` | Called when plugin is enabled |
| `DisableAsync()` | Called when plugin is disabled |
| `Dispose()` | Clean up resources |

### IPluginHost Properties

| Property | Type | Description |
|----------|------|-------------|
| `Logger` | `IPluginLogger` | Logging service |
| `Events` | `IPluginEvents` | Event subscriptions |
| `InstanceService` | `IPluginInstanceService` | Instance access |
| `DialogService` | `IPluginDialogService` | Show dialogs |
| `FileSystem` | `IPluginFileSystem` | File operations |
| `CurrentTheme` | `PluginThemeInfo?` | Current theme colors |
| `PluginDataDirectory` | `string` | Plugin storage path |
| `LauncherVersion` | `string` | NLauncher version |
| `IsDebugMode` | `bool` | Debug mode flag |

---

## Getting Help

- **Discord:** https://discord.gg/WbNYM68Bkt
- **SDK Package:** https://www.nuget.org/packages/NLauncher.PluginSDK
- **Plugin Repository:** https://github.com/Nyfaria/NLauncher-Plugins

