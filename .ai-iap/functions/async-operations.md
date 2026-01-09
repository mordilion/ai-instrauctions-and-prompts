---
title: Async Operations Patterns
category: Concurrency
difficulty: intermediate
languages:
  - typescript
  - python
  - java
  - csharp
  - php
  - kotlin
  - swift
  - dart
tags:
  - async
  - await
  - promises
  - concurrency
  - parallel
  - timeout
updated: 2026-01-09
---

# Async Operations Patterns

> **Purpose**: Handle asynchronous operations consistently
>
> **When to use**: API calls, database queries, file I/O, any operation that takes time

---

## TypeScript / JavaScript

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Native async/await** | Built-in | No install | Standard async operations |
| **Promises** | Built-in | No install | Promise-based operations |
| **p-queue** | `p-queue` | `npm install p-queue` | Rate limiting, concurrency control |
| **p-retry** | `p-retry` | `npm install p-retry` | Retry with exponential backoff |

### Native Async/Await

```typescript
// Basic async/await
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('User not found');
  return response.json();
}

// Sequential execution
async function fetchUserData(id: string) {
  const user = await fetchUser(id);
  const profile = await fetchProfile(user.profileId);
  const posts = await fetchPosts(user.id);
  return { user, profile, posts };
}

// Parallel execution
const [users, posts, comments] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
  fetchComments()
]);

// Race condition (first to complete wins)
const result = await Promise.race([
  fetchFromPrimary(),
  fetchFromBackup()
]);

// With timeout
function withTimeout<T>(promise: Promise<T>, ms: number): Promise<T> {
  return Promise.race([
    promise,
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
}

const user = await withTimeout(fetchUser(id), 5000);
```

### p-queue (Concurrency Control)

```typescript
// Install: npm install p-queue
import PQueue from 'p-queue';

const queue = new PQueue({ concurrency: 2 }); // Max 2 concurrent operations

// Add tasks to queue
const results = await Promise.all([
  queue.add(() => fetchUser('1')),
  queue.add(() => fetchUser('2')),
  queue.add(() => fetchUser('3')),
  queue.add(() => fetchUser('4'))
]);

// With priority
queue.add(() => fetchImportantData(), { priority: 1 });
queue.add(() => fetchNormalData(), { priority: 0 });
```

### p-retry (Retry Logic)

```typescript
// Install: npm install p-retry
import pRetry from 'p-retry';

const user = await pRetry(
  () => fetchUser(id),
  {
    retries: 3,
    onFailedAttempt: error => {
      console.log(`Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`);
    }
  }
);
```

---

## Python

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **asyncio** | Built-in Python | No install | Native async/await |
| **aiohttp** | `aiohttp` | `pip install aiohttp` | Async HTTP client |
| **asyncpg** | `asyncpg` | `pip install asyncpg` | Async PostgreSQL |
| **tenacity** | `tenacity` | `pip install tenacity` | Retry with backoff |

### Native asyncio

```python
import asyncio

# Basic async/await
async def fetch_user(user_id: str) -> User:
    async with aiohttp.ClientSession() as session:
        async with session.get(f'/api/users/{user_id}') as response:
            if response.status != 200:
                raise ValueError('User not found')
            return await response.json()

# Sequential
async def fetch_user_data(user_id: str):
    user = await fetch_user(user_id)
    profile = await fetch_profile(user.profile_id)
    posts = await fetch_posts(user.id)
    return {'user': user, 'profile': profile, 'posts': posts}

# Parallel execution
users, posts, comments = await asyncio.gather(
    fetch_users(),
    fetch_posts(),
    fetch_comments()
)

# With timeout
async def fetch_with_timeout(user_id: str, timeout: float = 5.0):
    try:
        return await asyncio.wait_for(fetch_user(user_id), timeout=timeout)
    except asyncio.TimeoutError:
        raise TimeoutError(f'Operation exceeded {timeout}s')

# Error handling in gather
results = await asyncio.gather(
    fetch_user('1'),
    fetch_user('2'),
    return_exceptions=True  # Don't fail on first error
)
```

### tenacity (Retry)

```python
# Install: pip install tenacity
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=10)
)
async def fetch_with_retry(url: str) -> dict:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            response.raise_for_status()
            return await response.json()
```

---

## Java

### 📦 Dependencies

| Approach | Library | Maven Dependency | Use Case |
|----------|---------|------------------|----------|
| **CompletableFuture** | Built-in Java | No install | Native async operations |
| **Virtual Threads** | Java 21+ | No install | Lightweight threads |
| **Reactor** | `io.projectreactor:reactor-core` | See below | Reactive programming |

### CompletableFuture

```java
// Basic async operation
CompletableFuture<User> fetchUser(String id) {
    return CompletableFuture.supplyAsync(() -> {
        Response response = httpClient.get("/api/users/" + id);
        if (!response.isSuccessful()) {
            throw new NotFoundException("User not found");
        }
        return response.body(User.class);
    });
}

// Sequential
CompletableFuture<Dashboard> fetchDashboard(String userId) {
    return fetchUser(userId)
        .thenCompose(user -> fetchProfile(user.getProfileId())
            .thenApply(profile -> new Dashboard(user, profile)));
}

// Parallel execution
CompletableFuture<List<User>> users = fetchUsers();
CompletableFuture<List<Post>> posts = fetchPosts();
CompletableFuture<List<Comment>> comments = fetchComments();

CompletableFuture.allOf(users, posts, comments)
    .thenApply(v -> new Dashboard(
        users.join(),
        posts.join(),
        comments.join()
    ));

// With timeout
user.orTimeout(5, TimeUnit.SECONDS)
    .exceptionally(ex -> {
        logger.error("Timeout fetching user", ex);
        return getDefaultUser();
    });
```

### Project Reactor (Spring WebFlux)

```xml
<!-- Maven: pom.xml -->
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
```

```java
import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

// Single result
Mono<User> fetchUser(String id) {
    return webClient.get()
        .uri("/api/users/{id}", id)
        .retrieve()
        .bodyToMono(User.class)
        .timeout(Duration.ofSeconds(5))
        .retry(3);
}

// Multiple results
Flux<User> fetchUsers() {
    return webClient.get()
        .uri("/api/users")
        .retrieve()
        .bodyToFlux(User.class);
}

// Parallel execution
Mono.zip(fetchUser("1"), fetchProfile("1"))
    .map(tuple -> new UserProfile(tuple.getT1(), tuple.getT2()));
```

---

## C# (.NET)

### 📦 Dependencies

| Approach | Library | NuGet | Use Case |
|----------|---------|-------|----------|
| **async/await** | Built-in .NET | No install | Native async operations |
| **Task Parallel Library** | Built-in .NET | No install | Parallel operations |
| **Polly** | `Polly` | `dotnet add package Polly` | Retry, circuit breaker |

### Native async/await

```csharp
// Basic async/await
async Task<User> FetchUserAsync(string id)
{
    var response = await _httpClient.GetAsync($"/api/users/{id}");
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsAsync<User>();
}

// Sequential
async Task<Dashboard> FetchDashboardAsync(string userId)
{
    var user = await FetchUserAsync(userId);
    var profile = await FetchProfileAsync(user.ProfileId);
    var posts = await FetchPostsAsync(user.Id);
    return new Dashboard(user, profile, posts);
}

// Parallel execution
var (users, posts, comments) = await (
    FetchUsersAsync(),
    FetchPostsAsync(),
    FetchCommentsAsync()
);

// Or with Task.WhenAll
var tasks = new[] {
    FetchUsersAsync(),
    FetchPostsAsync(),
    FetchCommentsAsync()
};
await Task.WhenAll(tasks);

// With timeout and cancellation
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
try
{
    var user = await FetchUserAsync(id, cts.Token);
    return user;
}
catch (OperationCanceledException)
{
    throw new TimeoutException("Request exceeded 5 seconds");
}
```

### Polly (Retry & Resilience)

```bash
# Install
dotnet add package Polly
```

```csharp
using Polly;

// Retry policy
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(
        3,
        retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
        onRetry: (exception, timeSpan, retryCount, context) =>
        {
            _logger.LogWarning($"Retry {retryCount} after {timeSpan.TotalSeconds}s");
        }
    );

var user = await retryPolicy.ExecuteAsync(() => FetchUserAsync(id));

// Timeout policy
var timeoutPolicy = Policy.TimeoutAsync<User>(5);
var user = await timeoutPolicy.ExecuteAsync(() => FetchUserAsync(id));

// Combined policies
var combinedPolicy = Policy.WrapAsync(retryPolicy, timeoutPolicy);
var user = await combinedPolicy.ExecuteAsync(() => FetchUserAsync(id));
```

---

## PHP

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Promises (Guzzle)** | `guzzlehttp/promises` | `composer require guzzlehttp/guzzle` | Promise-based HTTP |
| **ReactPHP** | `react/event-loop` | `composer require react/event-loop` | Event-driven async |
| **Amp** | `amphp/amp` | `composer require amphp/amp` | Async primitives |

### Guzzle Promises

```php
<?php
// Install: composer require guzzlehttp/guzzle
use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();

// Parallel requests
$promises = [
    'users' => $client->getAsync('/api/users'),
    'posts' => $client->getAsync('/api/posts'),
    'comments' => $client->getAsync('/api/comments'),
];

// Wait for all
$results = Promise\Utils::unwrap($promises);
$users = json_decode($results['users']->getBody(), true);

// Or with then()
$promise = $client->getAsync('/api/users/' . $id);
$promise->then(
    function ($response) {
        return json_decode($response->getBody(), true);
    },
    function ($exception) {
        throw new NotFoundException('User not found');
    }
);

// With timeout
$response = $client->request('GET', '/api/users/' . $id, [
    'timeout' => 5.0,
    'connect_timeout' => 2.0
]);
```

### Amp (Async PHP)

```php
<?php
// Install: composer require amphp/amp amphp/http-client
use Amp\Loop;
use Amp\Promise;

function fetchUser(string $id): Promise {
    return Amp\call(function () use ($id) {
        $httpClient = HttpClientBuilder::buildDefault();
        $request = new Request("/api/users/{$id}");
        $response = yield $httpClient->request($request);
        return yield $response->getBody()->buffer();
    });
}

// Parallel execution
Loop::run(function () {
    $results = yield [
        fetchUsers(),
        fetchPosts(),
        fetchComments()
    ];
    
    [$users, $posts, $comments] = $results;
});
```

---

## Kotlin

### 📦 Dependencies

| Approach | Library | Gradle | Use Case |
|----------|---------|--------|----------|
| **Coroutines** ⭐ | `org.jetbrains.kotlinx:kotlinx-coroutines-core` | See below | Native Kotlin async |
| **Flow** | Built-in with coroutines | See below | Reactive streams |

### Coroutines (Recommended)

```kotlin
// build.gradle.kts
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
}
```

```kotlin
import kotlinx.coroutines.*

// Basic suspend function
suspend fun fetchUser(id: String): User {
    return withContext(Dispatchers.IO) {
        val response = httpClient.get("/api/users/$id")
        if (!response.status.isSuccess()) {
            throw NotFoundException("User not found")
        }
        response.body()
    }
}

// Sequential
suspend fun fetchDashboard(userId: String): Dashboard {
    val user = fetchUser(userId)
    val profile = fetchProfile(user.profileId)
    val posts = fetchPosts(user.id)
    return Dashboard(user, profile, posts)
}

// Parallel execution
val (users, posts, comments) = coroutineScope {
    val usersDeferred = async { fetchUsers() }
    val postsDeferred = async { fetchPosts() }
    val commentsDeferred = async { fetchComments() }
    
    Triple(
        usersDeferred.await(),
        postsDeferred.await(),
        commentsDeferred.await()
    )
}

// With timeout
withTimeout(5000) {
    fetchUser(id)
}

// With retry
suspend fun <T> retry(
    times: Int = 3,
    initialDelay: Long = 100,
    maxDelay: Long = 1000,
    factor: Double = 2.0,
    block: suspend () -> T
): T {
    var currentDelay = initialDelay
    repeat(times - 1) {
        try {
            return block()
        } catch (e: Exception) {
            delay(currentDelay)
            currentDelay = (currentDelay * factor).toLong().coerceAtMost(maxDelay)
        }
    }
    return block() // Last attempt
}

val user = retry { fetchUser(id) }
```

---

## Swift

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **async/await** | Built-in iOS 15+ | No install | Native Swift concurrency |
| **Combine** | Built-in iOS 13+ | No install | Reactive programming |
| **PromiseKit** | `PromiseKit` | Swift Package Manager | Promise-based async |

### Native async/await (iOS 15+)

```swift
// Basic async function
func fetchUser(id: String) async throws -> User {
    let url = URL(string: "/api/users/\(id)")!
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw AppError.notFound(resource: "User")
    }
    
    return try JSONDecoder().decode(User.self, from: data)
}

// Sequential
func fetchDashboard(userId: String) async throws -> Dashboard {
    let user = try await fetchUser(id: userId)
    let profile = try await fetchProfile(id: user.profileId)
    let posts = try await fetchPosts(userId: userId)
    return Dashboard(user: user, profile: profile, posts: posts)
}

// Parallel execution
async let users = fetchUsers()
async let posts = fetchPosts()
async let comments = fetchComments()

let dashboard = try await Dashboard(
    users: users,
    posts: posts,
    comments: comments
)

// With timeout
func withTimeout<T>(
    seconds: TimeInterval,
    operation: @escaping () async throws -> T
) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        group.addTask {
            try await operation()
        }
        group.addTask {
            try await Task.sleep(nanoseconds: UInt64(seconds * 1_000_000_000))
            throw URLError(.timedOut)
        }
        
        let result = try await group.next()!
        group.cancelAll()
        return result
    }
}

let user = try await withTimeout(seconds: 5) {
    try await fetchUser(id: id)
}
```

### Combine (Reactive)

```swift
// Built into iOS 13+
import Combine

var cancellables = Set<AnyCancellable>()

// Single request
URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .decode(type: User.self, decoder: JSONDecoder())
    .receive(on: DispatchQueue.main)
    .sink(
        receiveCompletion: { completion in
            if case .failure(let error) = completion {
                print("Error: \(error)")
            }
        },
        receiveValue: { user in
            print("User: \(user)")
        }
    )
    .store(in: &cancellables)

// Parallel execution
let usersPublisher = fetchUsers()
let postsPublisher = fetchPosts()

Publishers.Zip(usersPublisher, postsPublisher)
    .sink { users, posts in
        print("Got \(users.count) users and \(posts.count) posts")
    }
    .store(in: &cancellables)
```

---

## Dart (Flutter)

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Native async/await** | Built-in Dart | No install | Standard async operations |
| **Streams** | Built-in Dart | No install | Reactive data streams |
| **dio** | `dio` | `flutter pub add dio` | Advanced HTTP client with interceptors |

### Native async/await

```dart
// Basic async function
Future<User> fetchUser(String id) async {
  final response = await http.get(Uri.parse('/api/users/$id'));
  
  if (response.statusCode != 200) {
    throw NotFoundException('User not found');
  }
  
  return User.fromJson(jsonDecode(response.body));
}

// Sequential
Future<Dashboard> fetchDashboard(String userId) async {
  final user = await fetchUser(userId);
  final profile = await fetchProfile(user.profileId);
  final posts = await fetchPosts(user.id);
  return Dashboard(user: user, profile: profile, posts: posts);
}

// Parallel execution
final results = await Future.wait([
  fetchUsers(),
  fetchPosts(),
  fetchComments(),
]);

final users = results[0] as List<User>;
final posts = results[1] as List<Post>;
final comments = results[2] as List<Comment>;

// With timeout
try {
  final user = await fetchUser(id).timeout(
    Duration(seconds: 5),
    onTimeout: () => throw TimeoutException('Request exceeded 5s'),
  );
  return user;
} catch (e) {
  logger.error('Failed to fetch user: $e');
  rethrow;
}

// Retry logic
Future<T> retry<T>(
  Future<T> Function() fn, {
  int maxAttempts = 3,
  Duration delay = const Duration(seconds: 1),
}) async {
  int attempt = 0;
  while (true) {
    try {
      return await fn();
    } catch (e) {
      attempt++;
      if (attempt >= maxAttempts) rethrow;
      await Future.delayed(delay * attempt);
    }
  }
}

final user = await retry(() => fetchUser(id));
```

### Streams

```dart
// Stream handling
Stream<User> watchUsers() async* {
  await for (final user in database.watchUsers()) {
    yield user;
  }
}

// Transform stream
stream
  .where((user) => user.age > 18)
  .map((user) => user.name)
  .listen((name) => print(name));

// Combine streams
StreamZip([usersStream, postsStream])
  .listen((List results) {
    final users = results[0];
    final posts = results[1];
  });
```

---

## Best Practices

✅ **DO**:
- Always handle errors in async operations
- Use timeout to prevent hanging indefinitely
- Run independent operations in parallel (not sequential)
- Cancel operations when no longer needed (use cancellation tokens)
- Use structured concurrency (scoped async operations)
- Log async operation failures with context
- Test timeout and retry logic

❌ **DON'T**:
- Block the main/UI thread with synchronous operations
- Forget to await promises/futures
- Run operations sequentially when parallel works
- Ignore cancellation tokens
- Mix async and sync patterns inconsistently
- Create unbounded concurrent operations (use queues)
- Ignore backpressure in streams

---

## Common Patterns

### Retry with Exponential Backoff

```typescript
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Debounce

```typescript
function debounce<T extends (...args: any[]) => any>(
  fn: T,
  ms: number
): (...args: Parameters<T>) => Promise<ReturnType<T>> {
  let timeoutId: NodeJS.Timeout;
  return (...args) => {
    return new Promise((resolve) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => resolve(fn(...args)), ms);
    });
  };
}

const debouncedSearch = debounce(search, 300);
```

### Async Queue

```typescript
class AsyncQueue {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(private concurrency = 1) {}
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.concurrency) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

const queue = new AsyncQueue(2);
queue.add(() => fetchUser('1'));
queue.add(() => fetchUser('2'));
```

---

## Performance Comparison

| Pattern | Use Case | Performance | Complexity |
|---------|----------|-------------|------------|
| Sequential | Operations depend on each other | Slowest | Simple |
| Parallel (Promise.all) | Independent operations | Fastest | Medium |
| Race | First result wins | Fast | Medium |
| Queue | Rate limiting needed | Controlled | Complex |
| Retry | Transient failures | Variable | Medium |
