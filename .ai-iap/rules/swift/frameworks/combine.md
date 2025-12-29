# Combine Framework

> **Scope**: Apple's reactive framework for processing values over time
> **Applies to**: Swift files using Combine
> **Extends**: swift/architecture.md, swift/code-style.md

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Store cancellables properly (Set<AnyCancellable> or AnyCancellable)
> **ALWAYS**: Use `.eraseToAnyPublisher()` for public APIs
> **ALWAYS**: Handle completion and errors with `sink(receiveCompletion:receiveValue:)`
> **ALWAYS**: Use `.receive(on: DispatchQueue.main)` before UI updates
> **ALWAYS**: Use weak self in closures to prevent retain cycles
> 
> **NEVER**: Ignore cancellables (causes memory leaks and unexpected behavior)
> **NEVER**: Update UI on background threads
> **NEVER**: Use `.sink` without storing the cancellable
> **NEVER**: Force unwrap publisher values
> **NEVER**: Create retain cycles with self in closures

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| **PassthroughSubject** | Manual value emission | Stateless, events |
| **CurrentValueSubject** | Stateful values | Initial value, state |
| **@Published** | Property observation | SwiftUI, ObservableObject |
| **Future** | Single async value | Promise-based, one-shot |

## Core Patterns

### Publishers

```swift
// Just (single value)
let just = Just(42)

// PassthroughSubject (manual emissions)
let subject = PassthroughSubject<String, Never>()
subject.send("Hello")

// CurrentValueSubject (stateful)
let state = CurrentValueSubject<Int, Never>(0)
print(state.value)  // Current value access

// @Published (property wrapper)
class ViewModel: ObservableObject {
    @Published var username = ""
    @Published var isLoading = false
}
```

### Operators

```swift
// map, filter, compactMap
[1, 2, 3].publisher
    .map { $0 * 2 }
    .filter { $0 > 4 }
    .compactMap { $0 }
    .sink { print($0) }
    .store(in: &cancellables)

// flatMap (async chaining)
userIDs.publisher
    .flatMap { id in fetchUser(id: id) }
    .sink(receiveCompletion: { _ in }, receiveValue: { print($0) })
    .store(in: &cancellables)

// zip, combineLatest
publisher1.zip(publisher2)
    .sink { val1, val2 in print("\(val1), \(val2)") }
    .store(in: &cancellables)
```

### Subscribers & Cancellables

```swift
class UserViewModel {
    private var cancellables = Set<AnyCancellable>()
    
    func fetchUsers() {
        userService.getUsers()
            .receive(on: DispatchQueue.main)  // UI updates on main thread
            .sink(
                receiveCompletion: { [weak self] completion in
                    switch completion {
                    case .finished: self?.isLoading = false
                    case .failure(let error): self?.handleError(error)
                    }
                },
                receiveValue: { [weak self] users in
                    self?.users = users
                }
            )
            .store(in: &cancellables)
    }
}
```

### Error Handling

```swift
func fetchData() -> AnyPublisher<Data, Error> {
    URLSession.shared.dataTaskPublisher(for: url)
        .map(\.data)
        .retry(3)
        .catch { error in Just(Data()).setFailureType(to: Error.self) }
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()
}
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **No Cancellable Storage** | `publisher.sink {}` | `.store(in: &cancellables)` | Memory leak, unexpected behavior |
| **Background UI Updates** | `.sink { self.label.text = ... }` | `.receive(on: .main)` | Crash risk |
| **Retain Cycles** | `self.update()` in closure | `[weak self]` | Memory leak |
| **No Error Handling** | `.sink { value in ... }` | `.sink(receiveCompletion:)` | Silent failures |

### Anti-Pattern: Lost Cancellable (MEMORY LEAK)

```swift
// ❌ WRONG: Cancellable not stored
func loadData() {
    dataService.fetch()
        .sink { data in
            self.process(data)
        }
    // Cancellable immediately cancelled!
}

// ✅ CORRECT: Store cancellable
private var cancellables = Set<AnyCancellable>()

func loadData() {
    dataService.fetch()
        .sink { [weak self] data in
            self?.process(data)
        }
        .store(in: &cancellables)
}
```

### Anti-Pattern: Background UI Update (CRASH RISK)

```swift
// ❌ WRONG: UI update on background thread
dataService.fetch()
    .sink { [weak self] data in
        self?.label.text = data  // May crash!
    }
    .store(in: &cancellables)

// ✅ CORRECT: UI update on main thread
dataService.fetch()
    .receive(on: DispatchQueue.main)
    .sink { [weak self] data in
        self?.label.text = data
    }
    .store(in: &cancellables)
```

## AI Self-Check (Verify BEFORE generating Combine code)

- [ ] Cancellables stored in Set<AnyCancellable>?
- [ ] Using [weak self] in closures?
- [ ] .receive(on: .main) before UI updates?
- [ ] Handling both completion and value?
- [ ] Using .eraseToAnyPublisher() for public APIs?
- [ ] Error handling with catch or retry?
- [ ] No force unwrapping?
- [ ] Proper operator chaining?
- [ ] No retain cycles?
- [ ] Cancellables cleaned up (stored)?

## Key Operators

| Operator | Purpose | Example Use |
|----------|---------|-------------|
| **map** | Transform values | `.map { $0 * 2 }` |
| **filter** | Select values | `.filter { $0 > 10 }` |
| **flatMap** | Async chaining | `.flatMap { fetchUser($0) }` |
| **zip** | Combine publishers | `.zip(publisher2)` |
| **combineLatest** | Latest from both | `.combineLatest(other)` |
| **debounce** | Rate limiting | `.debounce(for: .seconds(0.5))` |
| **removeDuplicates** | Distinct values | `.removeDuplicates()` |
| **catch** | Error recovery | `.catch { _ in Just(default) }` |

## SwiftUI Integration

```swift
class UserViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private var cancellables = Set<AnyCancellable>()
    
    func loadUsers() {
        isLoading = true
        
        userService.getUsers()
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.isLoading = false
                    if case .failure(let error) = completion {
                        self?.errorMessage = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] users in
                    self?.users = users
                }
            )
            .store(in: &cancellables)
    }
}
```

## Best Practices

**MUST**:
- Store cancellables
- [weak self] in closures
- .receive(on: .main) for UI
- Handle completions and errors
- .eraseToAnyPublisher() for public APIs

**SHOULD**:
- Use @Published for properties
- Debounce user input
- Retry on failures
- Use proper error types

**AVOID**:
- Lost cancellables
- Background UI updates
- Retain cycles
- Ignoring errors
- Force unwrapping
