# Core Data Framework

> **Scope**: Apple's object graph and persistence framework
> **Applies to**: Swift files using Core Data
> **Extends**: swift/architecture.md, swift/code-style.md

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use NSPersistentContainer for setup
> **ALWAYS**: Save context after modifications
> **ALWAYS**: Use background contexts for heavy operations
> **ALWAYS**: Use NSFetchRequest with predicates and sort descriptors
> **ALWAYS**: Handle fetch errors gracefully
> 
> **NEVER**: Perform heavy operations on main context
> **NEVER**: Pass NSManagedObject between threads
> **NEVER**: Force unwrap Core Data operations
> **NEVER**: Skip context.hasChanges check before saving
> **NEVER**: Use string literals for entity/attribute names (use generated classes)

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| **NSPersistentContainer** | Modern setup (iOS 10+) | Simplified stack |
| **Background Context** | Heavy operations | `performBackgroundTask` |
| **NSFetchedResultsController** | TableView/CollectionView | Automatic updates |
| **Batch Operations** | Bulk updates/deletes | Performance |

## Core Patterns

### Stack Setup

```swift
class CoreDataStack {
    static let shared = CoreDataStack()
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "Model")
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Unable to load stores: \(error)")
            }
        }
        container.viewContext.automaticallyMergesChangesFromParent = true
        return container
    }()
    
    var context: NSManagedObjectContext {
        persistentContainer.viewContext
    }
    
    func saveContext() {
        let context = persistentContainer.viewContext
        guard context.hasChanges else { return }
        try? context.save()
    }
}
```

### CRUD Operations

```swift
// Create
func createUser(name: String, email: String) -> User? {
    let context = CoreDataStack.shared.context
    let user = User(context: context)
    user.id = UUID()
    user.name = name
    user.email = email
    try? context.save()
    return user
}

// Read (with predicate)
func fetchUsers(matching query: String) -> [User] {
    let context = CoreDataStack.shared.context
    let fetchRequest: NSFetchRequest<User> = User.fetchRequest()
    
    if !query.isEmpty {
        fetchRequest.predicate = NSPredicate(format: "name CONTAINS[cd] %@", query)
    }
    fetchRequest.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
    
    return (try? context.fetch(fetchRequest)) ?? []
}

// Update
func updateUser(_ user: User, name: String) -> Bool {
    user.name = name
    return (try? user.managedObjectContext?.save()) != nil
}

// Delete
func deleteUser(_ user: User) -> Bool {
    guard let context = user.managedObjectContext else { return false }
    context.delete(user)
    return (try? context.save()) != nil
}
```

### Background Context

```swift
func importData(_ data: [UserData]) {
    let container = CoreDataStack.shared.persistentContainer
    container.performBackgroundTask { context in
        for item in data {
            let user = User(context: context)
            user.name = item.name
            user.email = item.email
        }
        try? context.save()
    }
}
```

### NSFetchedResultsController (TableView)

```swift
class UserListViewController: UITableViewController, NSFetchedResultsControllerDelegate {
    lazy var fetchedResultsController: NSFetchedResultsController<User> = {
        let fetchRequest: NSFetchRequest<User> = User.fetchRequest()
        fetchRequest.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
        
        let controller = NSFetchedResultsController(
            fetchRequest: fetchRequest,
            managedObjectContext: CoreDataStack.shared.context,
            sectionNameKeyPath: nil,
            cacheName: nil
        )
        controller.delegate = self
        return controller
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        try? fetchedResultsController.performFetch()
    }
    
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        tableView.reloadData()
    }
}
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **Heavy Main Context** | Import 1000s on main | Background context | UI freeze |
| **Cross-Thread Objects** | Pass object to thread | Pass objectID | Crash risk |
| **Force Unwrap** | `try! context.save()` | `try? context.save()` | Crash on error |
| **No hasChanges Check** | Always save | Check `hasChanges` | Performance |
| **String Literals** | `"User"` entity name | Generated `User.entity()` | Type safety |

### Anti-Pattern: Heavy Main Context (UI FREEZE)

```swift
// ❌ WRONG: Heavy operation on main context
func importUsers(_ data: [UserData]) {
    let context = CoreDataStack.shared.context  // Main context!
    for item in data {  // Blocks UI
        let user = User(context: context)
        user.name = item.name
    }
    try? context.save()
}

// ✅ CORRECT: Background context
func importUsers(_ data: [UserData]) {
    CoreDataStack.shared.persistentContainer.performBackgroundTask { context in
        for item in data {
            let user = User(context: context)
            user.name = item.name
        }
        try? context.save()
    }
}
```

### Anti-Pattern: Cross-Thread Objects (CRASH RISK)

```swift
// ❌ WRONG: Pass object between threads
DispatchQueue.global().async {
    user.name = "Updated"  // Crash! Wrong thread
}

// ✅ CORRECT: Pass objectID
let objectID = user.objectID
DispatchQueue.global().async {
    let context = CoreDataStack.shared.persistentContainer.newBackgroundContext()
    if let user = try? context.existingObject(with: objectID) as? User {
        user.name = "Updated"
        try? context.save()
    }
}
```

## AI Self-Check (Verify BEFORE generating Core Data code)

- [ ] Using NSPersistentContainer?
- [ ] Saving context after modifications?
- [ ] Background context for heavy operations?
- [ ] NSFetchRequest with predicates?
- [ ] Handling fetch errors?
- [ ] Not passing objects between threads?
- [ ] Checking hasChanges before save?
- [ ] Using generated entity classes?
- [ ] No force unwrapping?
- [ ] Proper sort descriptors?

## Key Features

| Feature | Purpose | Keywords |
|---------|---------|----------|
| **NSPersistentContainer** | Stack setup | Modern, simplified |
| **NSFetchRequest** | Querying | Predicates, sort descriptors |
| **NSPredicate** | Filtering | `format:` with arguments |
| **Background Context** | Heavy operations | `performBackgroundTask` |
| **NSFetchedResultsController** | TableView integration | Automatic updates |
| **Batch Operations** | Bulk updates/deletes | `NSBatchUpdateRequest` |

## Batch Operations

```swift
// Batch update
func markAllPostsAsRead() {
    let batchUpdate = NSBatchUpdateRequest(entityName: "Post")
    batchUpdate.propertiesToUpdate = ["isRead": true]
    batchUpdate.resultType = .updatedObjectIDsResultType
    
    try? CoreDataStack.shared.context.execute(batchUpdate)
}

// Batch delete
func deleteOldPosts() {
    let fetchRequest: NSFetchRequest<NSFetchRequestResult> = Post.fetchRequest()
    fetchRequest.predicate = NSPredicate(format: "createdAt < %@", oldDate as NSDate)
    
    let batchDelete = NSBatchDeleteRequest(fetchRequest: fetchRequest)
    try? CoreDataStack.shared.context.execute(batchDelete)
}
```

## Predicates

```swift
// Simple comparison
NSPredicate(format: "age > %d", 18)

// String matching
NSPredicate(format: "name CONTAINS[cd] %@", searchTerm)  // Case/diacritic insensitive

// Date range
NSPredicate(format: "createdAt >= %@ AND createdAt <= %@", startDate as NSDate, endDate as NSDate)

// Relationship
NSPredicate(format: "posts.@count > %d", 10)
```

## Best Practices

**MUST**:
- NSPersistentContainer
- Background context for heavy ops
- Check hasChanges before save
- Handle fetch errors
- Use generated classes

**SHOULD**:
- NSFetchedResultsController for lists
- Batch operations for bulk changes
- Predicates for filtering
- objectID for cross-thread access

**AVOID**:
- Heavy ops on main context
- Passing objects between threads
- Force unwrapping
- String literals for entities
- Saving without hasChanges check
