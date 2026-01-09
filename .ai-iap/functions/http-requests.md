---
title: HTTP Request Patterns
category: Network Communication
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
  - http
  - api
  - rest
  - retry
  - timeout
  - networking
updated: 2026-01-09
---

# HTTP Request Patterns

> **Purpose**: Safe and reliable HTTP client requests with error handling and retries
>
> **When to use**: API calls, microservice communication, external service integration

---

## TypeScript / JavaScript

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **fetch** | Built-in (Node 18+) | No install | Native HTTP client |
| **axios** ⭐ | `axios` | `npm install axios` | Full-featured HTTP client |
| **got** | `got` | `npm install got` | Modern HTTP client |
| **node-fetch** | `node-fetch` | `npm install node-fetch` | fetch polyfill (Node <18) |

### Native fetch

```typescript
// Basic GET request
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    signal: AbortSignal.timeout(5000) // 5s timeout
  });
  
  if (!response.ok) {
    throw new HttpError(response.status, await response.text());
  }
  
  return response.json();
}

// POST request
async function createUser(data: UserCreate): Promise<User> {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    throw new HttpError(response.status, await response.text());
  }
  
  return response.json();
}
```

### axios (Recommended)

```typescript
// Install: npm install axios
import axios from 'axios';

// Create instance with defaults
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
  headers: { 'Content-Type': 'application/json' }
});

// Request interceptor
api.interceptors.request.use(
  (config) => {
    const token = getAuthToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
api.interceptors.response.use(
  (response) => response.data,
  async (error) => {
    if (error.response?.status === 401) {
      await refreshToken();
      return api.request(error.config);
    }
    return Promise.reject(error);
  }
);

// Usage
const user = await api.get(`/users/${id}`);
const newUser = await api.post('/users', userData);

// Retry logic
import axiosRetry from 'axios-retry';

axiosRetry(api, {
  retries: 3,
  retryDelay: axiosRetry.exponentialDelay,
  retryCondition: (error) => {
    return axiosRetry.isNetworkOrIdempotentRequestError(error) ||
           error.response?.status >= 500;
  }
});
```

---

## Python

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **httpx** ⭐ | `httpx` | `pip install httpx` | Modern async HTTP client |
| **requests** | `requests` | `pip install requests` | Simple synchronous client |
| **aiohttp** | `aiohttp` | `pip install aiohttp` | Async HTTP client |

### httpx (Recommended)

```python
# Install: pip install httpx
import httpx

# Async client
async def fetch_user(user_id: str) -> dict:
    async with httpx.AsyncClient(timeout=5.0) as client:
        response = await client.get(
            f'/api/users/{user_id}',
            headers={
                'Authorization': f'Bearer {token}',
                'Content-Type': 'application/json'
            }
        )
        response.raise_for_status()
        return response.json()

# POST request
async def create_user(data: dict) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.post(
            '/api/users',
            json=data,
            headers={'Authorization': f'Bearer {token}'}
        )
        response.raise_for_status()
        return response.json()

# Retry with tenacity
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=10)
)
async def fetch_with_retry(url: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        response.raise_for_status()
        return response.json()
```

### requests (Sync)

```python
# Install: pip install requests
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# Simple request
def fetch_user_sync(user_id: str) -> dict:
    response = requests.get(
        f'/api/users/{user_id}',
        headers={'Authorization': f'Bearer {token}'},
        timeout=5
    )
    response.raise_for_status()
    return response.json()

# Session with retry
session = requests.Session()
retry_strategy = Retry(
    total=3,
    backoff_factor=1,
    status_forcelist=[429, 500, 502, 503, 504]
)
adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("http://", adapter)
session.mount("https://", adapter)

response = session.get(url, timeout=5)
```

---

## Java

### 📦 Dependencies

| Approach | Library | Maven | Use Case |
|----------|---------|-------|----------|
| **HttpClient** | Built-in Java 11+ | No install | Native HTTP client |
| **RestTemplate** | Spring | `spring-web` | Spring synchronous |
| **WebClient** ⭐ | Spring WebFlux | `spring-webflux` | Spring reactive |
| **OkHttp** | `com.squareup.okhttp3` | See below | Full-featured client |

### HttpClient (Java 11+)

```java
import java.net.http.*;

HttpClient client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(5))
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("/api/users/" + userId))
    .header("Authorization", "Bearer " + token)
    .header("Content-Type", "application/json")
    .GET()
    .build();

HttpResponse<String> response = client.send(
    request,
    HttpResponse.BodyHandlers.ofString()
);

if (response.statusCode() != 200) {
    throw new HttpException(response.statusCode(), response.body());
}

User user = objectMapper.readValue(response.body(), User.class);

// POST request
String requestBody = objectMapper.writeValueAsString(userData);

HttpRequest postRequest = HttpRequest.newBuilder()
    .uri(URI.create("/api/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString(requestBody))
    .build();
```

### WebClient (Spring WebFlux - Recommended)

```xml
<!-- Maven: pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
import org.springframework.web.reactive.function.client.WebClient;

@Service
public class UserService {
    private final WebClient webClient;
    
    public Mono<User> fetchUser(String id) {
        return webClient.get()
            .uri("/api/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(5))
            .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)));
    }
    
    public Mono<User> createUser(UserCreateDto dto) {
        return webClient.post()
            .uri("/api/users")
            .bodyValue(dto)
            .retrieve()
            .bodyToMono(User.class);
    }
}
```

---

## C# (.NET)

### 📦 Dependencies

| Approach | Library | NuGet | Use Case |
|----------|---------|-------|----------|
| **HttpClient** ⭐ | Built-in .NET | No install | Native HTTP client |
| **Refit** | `Refit` | `dotnet add package Refit` | Typed HTTP client |
| **RestSharp** | `RestSharp` | `dotnet add package RestSharp` | Simple REST client |

### HttpClient (Recommended)

```csharp
// Inject IHttpClientFactory
private readonly HttpClient _httpClient;

public async Task<User> FetchUserAsync(string id)
{
    var request = new HttpRequestMessage(
        HttpMethod.Get,
        $"/api/users/{id}"
    );
    request.Headers.Authorization = new AuthenticationHeaderValue(
        "Bearer",
        token
    );
    
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
    var response = await _httpClient.SendAsync(request, cts.Token);
    
    response.EnsureSuccessStatusCode();
    
    return await response.Content.ReadAsAsync<User>();
}

// POST request
public async Task<User> CreateUserAsync(UserCreateDto dto)
{
    var content = new StringContent(
        JsonSerializer.Serialize(dto),
        Encoding.UTF8,
        "application/json"
    );
    
    var response = await _httpClient.PostAsync("/api/users", content);
    response.EnsureSuccessStatusCode();
    
    return await response.Content.ReadAsAsync<User>();
}

// Retry with Polly
using Polly;

var retryPolicy = HttpPolicyExtensions
    .HandleTransientHttpError()
    .OrResult(msg => msg.StatusCode == HttpStatusCode.TooManyRequests)
    .WaitAndRetryAsync(
        3,
        retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))
    );

services.AddHttpClient("api")
    .AddPolicyHandler(retryPolicy);
```

### Refit (Typed HTTP)

```bash
# Install
dotnet add package Refit
```

```csharp
using Refit;

public interface IUserApi
{
    [Get("/users/{id}")]
    Task<User> GetUserAsync(string id);
    
    [Post("/users")]
    Task<User> CreateUserAsync([Body] UserCreateDto dto);
}

// Register in Startup.cs
services.AddRefitClient<IUserApi>()
    .ConfigureHttpClient(c => c.BaseAddress = new Uri("https://api.example.com"));

// Usage
var user = await _userApi.GetUserAsync("123");
```

---

## PHP

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Guzzle** ⭐ | `guzzlehttp/guzzle` | `composer require guzzlehttp/guzzle` | Full-featured client |
| **cURL** | Built-in PHP | No install | Native PHP HTTP |
| **Laravel HTTP** | Built-in Laravel | `composer require laravel/framework` | Laravel projects |

### Guzzle (Recommended)

```php
<?php
// Install: composer require guzzlehttp/guzzle
use GuzzleHttp\Client;

$client = new Client([
    'base_uri' => 'https://api.example.com',
    'timeout' => 5.0,
    'headers' => [
        'Authorization' => 'Bearer ' . $token,
        'Content-Type' => 'application/json'
    ]
]);

// GET request
try {
    $response = $client->get('/api/users/' . $userId);
    $user = json_decode($response->getBody(), true);
} catch (RequestException $e) {
    if ($e->hasResponse()) {
        $statusCode = $e->getResponse()->getStatusCode();
        throw new HttpException($statusCode, $e->getMessage());
    }
    throw $e;
}

// POST request
$response = $client->post('/api/users', [
    'json' => [
        'email' => 'user@example.com',
        'name' => 'John Doe'
    ]
]);

// Retry middleware
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware;

$stack = HandlerStack::create();
$stack->push(Middleware::retry(
    function ($retries, $request, $response, $exception) {
        if ($retries >= 3) return false;
        if ($response && $response->getStatusCode() >= 500) return true;
        return $exception instanceof ConnectException;
    },
    function ($retries) {
        return 1000 * pow(2, $retries); // Exponential backoff
    }
));

$client = new Client(['handler' => $stack]);
```

### Laravel HTTP Client

```php
<?php
// Built into Laravel
use Illuminate\Support\Facades\Http;

// GET request
$response = Http::withToken($token)
    ->timeout(5)
    ->retry(3, 1000)
    ->get('/api/users/' . $userId);

$user = $response->json();

// POST request
$response = Http::post('/api/users', [
    'email' => 'user@example.com',
    'name' => 'John Doe'
]);

// With error handling
$response = Http::withToken($token)
    ->get('/api/users/' . $userId);

if ($response->successful()) {
    $user = $response->json();
} elseif ($response->failed()) {
    throw new HttpException($response->status(), $response->body());
}
```

---

## Kotlin

### 📦 Dependencies

| Approach | Library | Gradle | Use Case |
|----------|---------|--------|----------|
| **OkHttp** | `com.squareup.okhttp3:okhttp` | See below | HTTP client |
| **Retrofit** ⭐ | `com.squareup.retrofit2:retrofit` | See below | Type-safe REST client |
| **Ktor Client** | `io.ktor:ktor-client-core` | See below | Multiplatform HTTP client |

### Retrofit (Recommended)

```kotlin
// build.gradle.kts
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    implementation("com.squareup.okhttp3:okhttp:4.11.0")
}
```

```kotlin
interface UserApi {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: String): User
    
    @POST("users")
    suspend fun createUser(@Body user: UserCreateDto): User
}

// Setup
val okHttpClient = OkHttpClient.Builder()
    .connectTimeout(5, TimeUnit.SECONDS)
    .addInterceptor { chain ->
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        chain.proceed(request)
    }
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com")
    .client(okHttpClient)
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val api = retrofit.create(UserApi::class.java)

// Usage
suspend fun fetchUser(id: String): User {
    return withContext(Dispatchers.IO) {
        try {
            api.getUser(id)
        } catch (e: IOException) {
            throw NetworkException("Network error", e)
        }
    }
}
```

### Ktor Client

```kotlin
// build.gradle.kts
dependencies {
    implementation("io.ktor:ktor-client-core:2.3.0")
    implementation("io.ktor:ktor-client-cio:2.3.0")
    implementation("io.ktor:ktor-client-content-negotiation:2.3.0")
    implementation("io.ktor:ktor-serialization-gson:2.3.0")
}
```

```kotlin
import io.ktor.client.*
import io.ktor.client.request.*

val client = HttpClient {
    install(HttpTimeout) {
        requestTimeoutMillis = 5000
    }
    install(HttpRequestRetry) {
        retryOnServerErrors(maxRetries = 3)
        exponentialDelay()
    }
}

suspend fun fetchUser(id: String): User {
    return client.get("/api/users/$id") {
        header("Authorization", "Bearer $token")
    }.body()
}
```

---

## Swift

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **URLSession** | Built-in iOS | No install | Native HTTP client |
| **Alamofire** ⭐ | `Alamofire` | Swift Package Manager | Full-featured client |

### URLSession (Native)

```swift
// GET request
func fetchUser(id: String) async throws -> User {
    var request = URLRequest(url: URL(string: "/api/users/\(id)")!)
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    request.timeoutInterval = 5
    
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw HttpError.invalidResponse
    }
    
    return try JSONDecoder().decode(User.self, from: data)
}

// POST request
func createUser(dto: UserCreateDto) async throws -> User {
    var request = URLRequest(url: URL(string: "/api/users")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(dto)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(User.self, from: data)
}
```

### Alamofire (Recommended)

```swift
// Install via Swift Package Manager
import Alamofire

// GET request
AF.request("/api/users/\(id)", headers: [
    "Authorization": "Bearer \(token)"
])
.validate()
.responseDecodable(of: User.self) { response in
    switch response.result {
    case .success(let user):
        print(user)
    case .failure(let error):
        print("Error: \(error)")
    }
}

// POST request
AF.request("/api/users",
           method: .post,
           parameters: dto,
           encoder: JSONParameterEncoder.default)
    .validate()
    .responseDecodable(of: User.self) { response in
        // Handle response
    }

// Retry configuration
let interceptor = RetryPolicy(retryLimit: 3)
AF.request(url, interceptor: interceptor)
```

---

## Dart (Flutter)

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **http** | `http` | `flutter pub add http` | Simple HTTP client |
| **dio** ⭐ | `dio` | `flutter pub add dio` | Advanced HTTP client |

### Native http

```dart
// Install: flutter pub add http
import 'package:http/http.dart' as http;

// GET request
Future<User> fetchUser(String id) async {
  final response = await http.get(
    Uri.parse('/api/users/$id'),
    headers: {
      'Authorization': 'Bearer $token',
      'Content-Type': 'application/json',
    },
  ).timeout(Duration(seconds: 5));
  
  if (response.statusCode != 200) {
    throw HttpException(response.statusCode, response.body);
  }
  
  return User.fromJson(jsonDecode(response.body));
}

// POST request
Future<User> createUser(UserCreateDto dto) async {
  final response = await http.post(
    Uri.parse('/api/users'),
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode(dto.toJson()),
  );
  
  if (response.statusCode != 201) {
    throw HttpException(response.statusCode, response.body);
  }
  
  return User.fromJson(jsonDecode(response.body));
}
```

### dio (Recommended)

```dart
// Install: flutter pub add dio
import 'package:dio/dio.dart';

final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',
  connectTimeout: Duration(seconds: 5),
  headers: {'Authorization': 'Bearer $token'},
));

// Interceptor for retry
dio.interceptors.add(RetryInterceptor(
  dio: dio,
  logPrint: print,
  retries: 3,
  retryDelays: [
    Duration(seconds: 1),
    Duration(seconds: 2),
    Duration(seconds: 4),
  ],
));

// GET request
Future<User> fetchUser(String id) async {
  try {
    final response = await dio.get('/users/$id');
    return User.fromJson(response.data);
  } on DioError catch (e) {
    if (e.response?.statusCode == 404) {
      throw NotFoundException('User not found');
    }
    rethrow;
  }
}

// POST request
Future<User> createUser(UserCreateDto dto) async {
  final response = await dio.post('/users', data: dto.toJson());
  return User.fromJson(response.data);
}
```

---

## Best Practices

✅ **DO**:
- Set timeouts (connect + read)
- Retry transient errors (500s, network)
- Use exponential backoff
- Log request/response details
- Handle rate limiting (429)
- Validate SSL certificates
- Use connection pooling
- Cancel requests when not needed

❌ **DON'T**:
- Ignore timeouts (infinite wait)
- Retry non-idempotent requests blindly
- Expose auth tokens in logs
- Trust server certificates without validation
- Create new clients per request
- Retry immediately without backoff
- Ignore response status codes

---

## Common Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process response |
| 201 | Created | Process new resource |
| 400 | Bad Request | Fix request, don't retry |
| 401 | Unauthorized | Refresh token, retry |
| 403 | Forbidden | Don't retry, show error |
| 404 | Not Found | Don't retry, handle missing |
| 429 | Rate Limited | Retry after delay |
| 500 | Server Error | Retry with backoff |
| 502/503 | Service Unavailable | Retry with backoff |
| 504 | Gateway Timeout | Retry with backoff |

---

## Security Checklist

- [ ] HTTPS only (no HTTP)
- [ ] Validate SSL certificates
- [ ] Auth tokens in headers (not URL)
- [ ] Sensitive data encrypted
- [ ] Timeout set on all requests
- [ ] Input validated before sending
- [ ] Responses validated before using
- [ ] Tokens not logged
- [ ] API keys not in code (use env)
