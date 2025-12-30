# GetX Framework

> **Scope**: State management, routing, and DI for Flutter  
> **Applies to**: Dart files using GetX in Flutter
> **Extends**: dart/architecture.md, dart/code-style.md

## CRITICAL REQUIREMENTS

> **ALWAYS**: Use GetxController (NOT StatefulWidget)
> **ALWAYS**: Use .obs for reactive variables
> **ALWAYS**: Use bindings for DI
> **ALWAYS**: Use Get.find() to access controllers
> **ALWAYS**: Dispose in onClose()
> 
> **NEVER**: Instantiate controllers directly
> **NEVER**: Use StatefulWidget with GetX
> **NEVER**: Skip onClose() cleanup
> **NEVER**: Overuse Get.find()
> **NEVER**: Create global state unless needed

## Core Patterns

### Controller

```dart
class UserController extends GetxController {
  final UserRepository _repository;
  UserController(this _repository);
  
  final users = <User>[].obs;
  final isLoading = false.obs;
  final error = Rxn<String>();
  
  @override
  void onInit() {
    super.onInit();
    loadUsers();
  }
  
  Future<void> loadUsers() async {
    try {
      isLoading.value = true;
      error.value = null;
      users.value = await _repository.getUsers();
    } catch (e) {
      error.value = e.toString();
    } finally {
      isLoading.value = false;
    }
  }
  
  @override
  void onClose() {
    // Cleanup
    super.onClose();
  }
}
```

### View (Obx)

```dart
class UserListView extends StatelessWidget {
  final controller = Get.find<UserController>();
  
  @override
  Widget build(BuildContext context) {
    return Obx(() {
      if (controller.isLoading.value) {
        return CircularProgressIndicator();
      }
      return ListView.builder(
        itemCount: controller.users.length,
        itemBuilder: (context, index) => UserTile(controller.users[index]),
      );
    });
  }
}

// Or use GetView
class UserListView extends GetView<UserController> {
  @override
  Widget build(BuildContext context) {
    return Obx(() => ListView.builder(
      itemCount: controller.users.length,
      itemBuilder: (context, index) => UserTile(controller.users[index]),
    ));
  }
}
```

### Bindings

```dart
class UserBinding extends Bindings {
  @override
  void dependencies() {
    Get.lazyPut(() => UserRepository());
    Get.lazyPut(() => UserController(Get.find()));
  }
}

// Usage
GetPage(
  name: '/users',
  page: () => UserListView(),
  binding: UserBinding(),
);
```

### Navigation

```dart
// Navigate
Get.to(() => UserDetailView());

// Named route
Get.toNamed('/user/details', arguments: userId);

// Replace
Get.off(() => HomeView());

// Clear stack
Get.offAll(() => LoginView());

// Back
Get.back();
```

### Dialogs & Snackbars

```dart
// Snackbar
Get.snackbar('Success', 'User created');

// Dialog
Get.defaultDialog(
  title: 'Confirm',
  middleText: 'Delete user?',
  onConfirm: () => controller.deleteUser(),
);

// Bottom sheet
Get.bottomSheet(Container(child: Text('Options')));
```

## Common AI Mistakes

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| **Direct Instantiation** | `UserController()` | `Get.find<UserController>()` |
| **StatefulWidget** | With GetX | Use GetxController |
| **No onClose** | Missing cleanup | Implement onClose() |
| **Overuse Get.find** | Everywhere | Use GetView |

## AI Self-Check

- [ ] Using GetxController?
- [ ] .obs for reactive vars?
- [ ] Bindings for DI?
- [ ] Get.find() for access?
- [ ] onClose() cleanup?
- [ ] No direct instantiation?
- [ ] No StatefulWidget?
- [ ] GetView where appropriate?

## Key Features

| Feature | Purpose |
|---------|---------|
| GetxController | State management |
| .obs | Reactive variables |
| Obx | UI rebuild |
| Bindings | DI |
| Get.to() | Navigation |

## Best Practices

**MUST**: GetxController, .obs, bindings, onClose(), Get.find()
**SHOULD**: GetView, lazy loading, named routes
**AVOID**: Direct instantiation, StatefulWidget, overuse Get.find(), global state
