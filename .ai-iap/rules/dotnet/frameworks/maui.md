# .NET MAUI Framework

> **Scope**: .NET Multi-platform App UI for cross-platform mobile/desktop apps
> **Applies to**: C# files using .NET MAUI
> **Extends**: dotnet/architecture.md, dotnet/code-style.md

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use MVVM pattern (ViewModels separate from Views)
> **ALWAYS**: Use CommunityToolkit.Mvvm for boilerplate reduction
> **ALWAYS**: Use dependency injection for services
> **ALWAYS**: Use Shell for navigation
> **ALWAYS**: Handle platform differences gracefully
> 
> **NEVER**: Put logic in code-behind (use ViewModels)
> **NEVER**: Block UI thread with synchronous operations
> **NEVER**: Use static state
> **NEVER**: Skip platform-specific testing
> **NEVER**: Create memory leaks (unsubscribe events)

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| **MVVM** | All apps | ViewModels, data binding |
| **Shell** | Navigation | Routes, hierarchical |
| **DI** | Service injection | `MauiProgram.cs` |
| **Platform-Specific** | Native features | `#if ANDROID`, conditional compilation |

## Core Patterns

### MVVM (ViewModel)

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

public partial class UserViewModel : ObservableObject
{
    private readonly IUserService _userService;
    
    [ObservableProperty]
    private string _username = "";
    
    [ObservableProperty]
    [NotifyCanExecuteChangedFor(nameof(SaveCommand))]
    private bool _isValid;
    
    public UserViewModel(IUserService userService)
    {
        _userService = userService;
    }
    
    [RelayCommand(CanExecute = nameof(CanSave))]
    private async Task SaveAsync()
    {
        await _userService.SaveAsync(Username);
    }
    
    private bool CanSave() => IsValid && !string.IsNullOrWhiteSpace(Username);
}
```

### View (XAML + Binding)

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             x:Class="MyApp.Views.UserPage"
             x:DataType="vm:UserViewModel">
    <StackLayout Padding="20">
        <Entry Text="{Binding Username}"
               Placeholder="Enter username" />
        
        <Button Text="Save"
                Command="{Binding SaveCommand}"
                IsEnabled="{Binding IsValid}" />
        
        <ActivityIndicator IsRunning="{Binding IsBusy}"
                          IsVisible="{Binding IsBusy}" />
    </StackLayout>
</ContentPage>
```

### Dependency Injection

```csharp
// MauiProgram.cs
public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
            .ConfigureFonts(fonts => {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            });
        
        // Services
        builder.Services.AddSingleton<IUserService, UserService>();
        builder.Services.AddTransient<UserViewModel>();
        builder.Services.AddTransient<UserPage>();
        
        return builder.Build();
    }
}
```

### Navigation (Shell)

```xml
<!-- AppShell.xaml -->
<Shell xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
       x:Class="MyApp.AppShell">
    <TabBar>
        <Tab Title="Home" Icon="home.png">
            <ShellContent ContentTemplate="{DataTemplate views:HomePage}" />
        </Tab>
        <Tab Title="Profile" Icon="profile.png">
            <ShellContent ContentTemplate="{DataTemplate views:ProfilePage}" />
        </Tab>
    </TabBar>
</Shell>
```

```csharp
// Navigate in code
await Shell.Current.GoToAsync("//profile");
await Shell.Current.GoToAsync($"details?id={userId}");
```

### Platform-Specific Code

```csharp
// Conditional compilation
public partial class FileService
{
#if ANDROID
    public string GetStoragePath() => Android.OS.Environment.ExternalStorageDirectory.AbsolutePath;
#elif IOS
    public string GetStoragePath() => Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
#elif WINDOWS
    public string GetStoragePath() => Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
#endif
}

// Platform-specific folders
// Platforms/Android/MainActivity.cs
// Platforms/iOS/AppDelegate.cs
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **Code-Behind Logic** | Business logic in `.xaml.cs` | ViewModel | Testability, MVVM |
| **No DI** | `new UserService()` | Constructor injection | Testability |
| **Blocking UI** | Synchronous I/O | async/await | Responsiveness |
| **Static State** | `static User CurrentUser` | DI scoped services | Multi-platform issues |

### Anti-Pattern: Logic in Code-Behind (MVVM VIOLATION)

```csharp
// ❌ WRONG: Logic in code-behind
public partial class UserPage : ContentPage
{
    public UserPage()
    {
        InitializeComponent();
    }
    
    private async void OnSaveClicked(object sender, EventArgs e)
    {
        var user = new User { Name = UsernameEntry.Text };
        await _userService.SaveAsync(user);  // Business logic!
    }
}

// ✅ CORRECT: ViewModel handles logic
public partial class UserViewModel : ObservableObject
{
    [RelayCommand]
    private async Task SaveAsync()
    {
        var user = new User { Name = Username };
        await _userService.SaveAsync(user);
    }
}
```

## AI Self-Check (Verify BEFORE generating MAUI code)

- [ ] Using MVVM pattern?
- [ ] ViewModels with CommunityToolkit.Mvvm?
- [ ] Dependency injection configured?
- [ ] Data binding (not code-behind)?
- [ ] Shell navigation?
- [ ] async/await for I/O?
- [ ] Platform-specific code isolated?
- [ ] ObservableProperty for bindable properties?
- [ ] RelayCommand for commands?
- [ ] No static state?

## Key Features

| Feature | Purpose | Keywords |
|---------|---------|----------|
| **MVVM** | Separation of concerns | ViewModels, data binding |
| **CommunityToolkit.Mvvm** | Boilerplate reduction | `[ObservableProperty]`, `[RelayCommand]` |
| **Shell** | Navigation | Routes, tabs |
| **DI** | Dependency injection | `MauiProgram.cs` |
| **Platform-Specific** | Native features | `Platforms/` folder |
| **Hot Reload** | Fast development | XAML/C# hot reload |

## Best Practices

**MUST**:
- MVVM pattern
- CommunityToolkit.Mvvm
- Dependency injection
- Shell navigation
- Platform-specific testing

**SHOULD**:
- Data binding (no code-behind)
- async/await
- ObservableProperty/RelayCommand
- Platform folders
- Converters for UI transformations

**AVOID**:
- Logic in code-behind
- Blocking operations
- Static state
- Manual property notification
- Platform-specific code in shared layer
