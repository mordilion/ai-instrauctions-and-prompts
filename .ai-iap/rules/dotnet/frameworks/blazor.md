# Blazor Framework

> **Scope**: Apply these rules when working with Blazor WebAssembly or Blazor Server applications.

## Overview

Blazor enables building interactive web UIs using C# instead of JavaScript. Blazor WebAssembly runs in browser via WebAssembly; Blazor Server runs on server with SignalR connection.

**Key Capabilities**:
- **C# for Web**: Write frontend in C#/.NET
- **Component-Based**: Reusable Razor components
- **Two Hosting Models**: WebAssembly (client) or Server (SignalR)
- **Full .NET**: Access entire .NET ecosystem
- **Hot Reload**: Fast development cycle

## Pattern Selection

### Hosting Model
**Use Blazor WebAssembly when**:
- Need offline capability
- Want client-side execution
- Can accept larger download size

**Use Blazor Server when**:
- Need small download size
- Want server-side execution
- Need direct database access

### State Management
**Use Component State when**:
- UI-only state (form input)
- Local to component

**Use Cascading Parameters when**:
- Shared UI state (theme, auth)
- Deep component trees

**Use Services when**:
- Application state
- Shared across pages

## Core Patterns

| Pattern | Example |
|---------|---------|
| **Component Parameters** | `[Parameter] public User User { get; set; }` |
| **Event Callbacks** | `[Parameter] public EventCallback<int> OnDelete { get; set; }` |
| **State Management** | `@inject IUserService UserService` (NOT static state) |
| **Event Handling** | `<button @onclick="HandleClickAsync">` (async Task) |
| **Forms** | `<EditForm Model="@model" OnValidSubmit="HandleSubmit">` |
| **Virtualization** | `<Virtualize Items="@users" Context="user">` |
| **JS Interop** | `await JSRuntime.InvokeAsync<T>("method", args)` |

## Hosting Models

| Aspect | Server | WebAssembly |
|--------|--------|-------------|
| **State** | Scoped per circuit | Singleton per tab |
| **DB Access** | Direct | Via API |
| **Offline** | No | Yes (PWA) |

## Best Practices

**MUST**: [Parameter], EventCallback, async Task, DI services, IAsyncDisposable  
**SHOULD**: EditForm, CommunityToolkit.Mvvm, ShouldRender, Virtualize  
**AVOID**: Static state, async void, forgetting StateHasChanged

