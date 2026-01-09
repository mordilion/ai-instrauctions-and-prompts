---
title: Database Query Patterns
category: Data Access
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
  - database
  - sql
  - orm
  - security
  - sql-injection
updated: 2026-01-09
---

# Database Query Patterns

> **Purpose**: Safe database queries that prevent SQL injection and ensure data integrity
>
> **When to use**: Any database interaction - SELECT, INSERT, UPDATE, DELETE

---

## TypeScript / JavaScript

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Prisma ORM** ⭐ | `@prisma/client` | `npm install prisma @prisma/client` | Type-safe, modern ORM |
| **TypeORM** | `typeorm` | `npm install typeorm` | Decorator-based ORM |
| **Knex.js** | `knex` | `npm install knex pg` | Query builder |
| **pg (PostgreSQL)** | `pg` | `npm install pg` | Raw PostgreSQL driver |

### Prisma ORM (Recommended)

```typescript
// Install: npm install prisma @prisma/client
// Setup: npx prisma init

import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

// Query
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: { posts: true }
});

// Insert
const newUser = await prisma.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
    posts: {
      create: [{ title: 'First Post', content: '...' }]
    }
  }
});

// Update
await prisma.user.update({
  where: { id: userId },
  data: { status: 'active' }
});

// Raw query with parameters
const users = await prisma.$queryRaw`
  SELECT * FROM users
  WHERE age > ${minAge} AND status = ${status}
`;
```

### TypeORM

```typescript
// Install: npm install typeorm reflect-metadata @types/node
// Database driver: npm install pg (PostgreSQL) or mysql2 (MySQL)

import { Repository } from 'typeorm';
import { User } from './entities/User';

// Query
const user = await userRepository.findOne({
  where: { id: userId },
  relations: ['posts']
});

// Insert
const newUser = userRepository.create({
  email: 'user@example.com',
  name: 'John Doe'
});
await userRepository.save(newUser);

// Query builder
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.age > :age', { age: minAge })
  .andWhere('user.status = :status', { status })
  .getMany();
```

### Knex.js (Query Builder)

```typescript
// Install: npm install knex pg

import knex from 'knex';
const db = knex({ client: 'pg', connection: process.env.DATABASE_URL });

// Query
const user = await db('users')
  .where({ id: userId })
  .first();

// Insert
const [id] = await db('users')
  .insert({
    email: 'user@example.com',
    name: 'John Doe'
  })
  .returning('id');

// Parameterized query
const users = await db('users')
  .where('age', '>', minAge)
  .andWhere('status', status);
```

### Plain PostgreSQL (pg)

```typescript
// Install: npm install pg @types/pg

import { Pool } from 'pg';
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// ✅ ALWAYS use parameterized queries
const query = 'SELECT * FROM users WHERE email = $1 AND status = $2';
const result = await pool.query(query, [email, status]);
const users = result.rows;

// Insert
const insertQuery = 'INSERT INTO users (email, name) VALUES ($1, $2) RETURNING id';
const { rows } = await pool.query(insertQuery, [email, name]);
const userId = rows[0].id;

// ❌ NEVER DO THIS (SQL Injection)
const badQuery = `SELECT * FROM users WHERE email = '${email}'`;
```

---

## Python

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **SQLAlchemy** ⭐ | `sqlalchemy` | `pip install sqlalchemy` | Full-featured ORM |
| **Django ORM** | Built-in Django | `pip install django` | Django projects |
| **asyncpg** | `asyncpg` | `pip install asyncpg` | Async PostgreSQL |
| **psycopg2** | `psycopg2-binary` | `pip install psycopg2-binary` | PostgreSQL driver |

### SQLAlchemy ORM (Recommended)

```python
# Install: pip install sqlalchemy

from sqlalchemy import create_engine, select
from sqlalchemy.orm import Session

engine = create_engine('postgresql://user:pass@localhost/db')

# Query
with Session(engine) as session:
    user = session.query(User).filter_by(id=user_id).first()
    
    # Or with select
    stmt = select(User).where(User.id == user_id)
    user = session.execute(stmt).scalar_one()

# Insert
with Session(engine) as session:
    new_user = User(email='user@example.com', name='John Doe')
    session.add(new_user)
    session.commit()

# Parameterized raw query
from sqlalchemy import text
stmt = text("SELECT * FROM users WHERE age > :age AND status = :status")
users = session.execute(stmt, {"age": min_age, "status": status}).all()
```

### Django ORM

```python
# Install: pip install django
# Requires Django project setup

from django.db import models

# Query
user = User.objects.get(id=user_id)
users = User.objects.filter(age__gt=min_age, status=status)

# Insert
user = User.objects.create(
    email='user@example.com',
    name='John Doe'
)

# Raw query with parameters
users = User.objects.raw(
    "SELECT * FROM users WHERE email = %s AND status = %s",
    [email, status]
)

# ❌ NEVER DO THIS
query = f"SELECT * FROM users WHERE email = '{email}'"
```

### asyncpg (Async PostgreSQL)

```python
# Install: pip install asyncpg

import asyncpg

# Connection
conn = await asyncpg.connect('postgresql://user:pass@localhost/db')

# Query with parameters
users = await conn.fetch(
    'SELECT * FROM users WHERE age > $1 AND status = $2',
    min_age, status
)

# Insert
user_id = await conn.fetchval(
    'INSERT INTO users (email, name) VALUES ($1, $2) RETURNING id',
    email, name
)

await conn.close()
```

### Plain psycopg2

```python
# Install: pip install psycopg2-binary

import psycopg2

conn = psycopg2.connect('postgresql://user:pass@localhost/db')
cursor = conn.cursor()

# ✅ ALWAYS use parameterized queries
cursor.execute(
    'SELECT * FROM users WHERE email = %s AND status = %s',
    (email, status)
)
users = cursor.fetchall()

cursor.close()
conn.close()
```

---

## Java

### 📦 Dependencies

| Approach | Library | Maven/Gradle | Use Case |
|----------|---------|--------------|----------|
| **Spring Data JPA** ⭐ | `spring-boot-starter-data-jpa` | See below | Spring Boot projects |
| **Hibernate** | `hibernate-core` | See below | Standalone ORM |
| **jOOQ** | `jooq` | See below | Type-safe SQL |
| **JDBC** | Built-in Java | No install | Raw database access |

### Spring Data JPA (Recommended)

```xml
<!-- Maven: pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByAgeGreaterThan(int age);
    
    @Query("SELECT u FROM User u WHERE u.status = :status")
    List<User> findByStatus(@Param("status") String status);
    
    // Native query with parameters
    @Query(value = "SELECT * FROM users WHERE age > ?1 AND status = ?2", 
           nativeQuery = true)
    List<User> findByAgeAndStatus(int age, String status);
}

// Usage
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("User not found"));
    }
}
```

### JPA/Hibernate

```xml
<!-- Maven -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.x.x</version>
</dependency>
```

```java
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Repository
public class UserRepository {
    @PersistenceContext
    private EntityManager em;
    
    public User findById(Long id) {
        return em.find(User.class, id);
    }
    
    public List<User> findByAge(int minAge) {
        return em.createQuery(
            "SELECT u FROM User u WHERE u.age > :minAge",
            User.class
        )
        .setParameter("minAge", minAge)
        .getResultList();
    }
}
```

### Plain JDBC (with PreparedStatement)

```java
// Driver included in modern Java, or add database-specific driver

import java.sql.*;

String url = "jdbc:postgresql://localhost:5432/db";
Connection conn = DriverManager.getConnection(url, "user", "pass");

// ✅ ALWAYS use PreparedStatement
String sql = "SELECT * FROM users WHERE email = ? AND status = ?";
try (PreparedStatement stmt = conn.prepareStatement(sql)) {
    stmt.setString(1, email);
    stmt.setString(2, status);
    
    ResultSet rs = stmt.executeQuery();
    while (rs.next()) {
        // Process results
    }
}

// ❌ NEVER DO THIS (SQL Injection)
String badQuery = "SELECT * FROM users WHERE email = '" + email + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(badQuery);
```

---

## C# (.NET)

### 📦 Dependencies

| Approach | Package | NuGet Command | Use Case |
|----------|---------|---------------|----------|
| **Entity Framework Core** ⭐ | `Microsoft.EntityFrameworkCore` | See below | Modern .NET ORM |
| **Dapper** | `Dapper` | `dotnet add package Dapper` | Micro ORM |
| **ADO.NET** | Built-in .NET | No install | Raw database access |

### Entity Framework Core (Recommended)

```bash
# Install
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
# Or for PostgreSQL:
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
}

// Query
var user = await _context.Users
    .Include(u => u.Posts)
    .FirstOrDefaultAsync(u => u.Id == userId);

var users = await _context.Users
    .Where(u => u.Age > minAge && u.Status == status)
    .ToListAsync();

// Insert
var newUser = new User {
    Email = "user@example.com",
    Name = "John Doe"
};
_context.Users.Add(newUser);
await _context.SaveChangesAsync();

// Raw SQL with parameters
var users = await _context.Users
    .FromSqlRaw(
        "SELECT * FROM Users WHERE Age > {0} AND Status = {1}",
        minAge,
        status
    )
    .ToListAsync();
```

### Dapper (Micro ORM)

```bash
# Install
dotnet add package Dapper
dotnet add package Microsoft.Data.SqlClient
```

```csharp
using Dapper;
using Microsoft.Data.SqlClient;

using (var connection = new SqlConnection(connectionString))
{
    // Query
    var user = await connection.QueryFirstOrDefaultAsync<User>(
        "SELECT * FROM Users WHERE Email = @Email",
        new { Email = email }
    );
    
    // Multiple results
    var users = await connection.QueryAsync<User>(
        "SELECT * FROM Users WHERE Age > @MinAge AND Status = @Status",
        new { MinAge = minAge, Status = status }
    );
    
    // Insert
    var sql = "INSERT INTO Users (Email, Name) VALUES (@Email, @Name); SELECT CAST(SCOPE_IDENTITY() as int)";
    var id = await connection.ExecuteScalarAsync<int>(sql, 
        new { Email = email, Name = name });
}
```

### ADO.NET (Plain)

```csharp
using System.Data.SqlClient;

using (var connection = new SqlConnection(connectionString))
using (var command = new SqlCommand())
{
    command.Connection = connection;
    
    // ✅ ALWAYS use parameters
    command.CommandText = "SELECT * FROM Users WHERE Email = @Email AND Status = @Status";
    command.Parameters.AddWithValue("@Email", email);
    command.Parameters.AddWithValue("@Status", status);
    
    await connection.OpenAsync();
    using var reader = await command.ExecuteReaderAsync();
    
    while (await reader.ReadAsync())
    {
        // Process results
    }
}

// ❌ NEVER DO THIS
var badQuery = $"SELECT * FROM Users WHERE Email = '{email}'";
```

---

## PHP

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Laravel Eloquent** ⭐ | Built-in Laravel | `composer require laravel/framework` | Laravel projects |
| **Doctrine ORM** | `doctrine/orm` | `composer require doctrine/orm` | Symfony, standalone |
| **PDO** | Built-in PHP | No install | Raw database access |

### Laravel Eloquent (Recommended)

```bash
# Install Laravel
composer create-project laravel/laravel myapp
```

```php
<?php
// Model: app/Models/User.php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $fillable = ['email', 'name'];
}

// Query
$user = User::where('email', $email)->first();
$users = User::where('age', '>', $minAge)
    ->where('status', $status)
    ->get();

// Insert
$user = User::create([
    'email' => 'user@example.com',
    'name' => 'John Doe',
]);

// Raw query with bindings
$users = DB::select(
    'SELECT * FROM users WHERE age > ? AND status = ?',
    [$minAge, $status]
);

// Query builder (without model)
$users = DB::table('users')
    ->where('age', '>', $minAge)
    ->where('status', $status)
    ->get();
```

### Doctrine ORM

```bash
# Install
composer require doctrine/orm
composer require doctrine/dbal
```

```php
<?php
use Doctrine\ORM\EntityManager;

// Query
$user = $entityManager->find(User::class, $userId);

$users = $entityManager->createQuery(
    'SELECT u FROM App\Entity\User u WHERE u.age > :age AND u.status = :status'
)
->setParameter('age', $minAge)
->setParameter('status', $status)
->getResult();

// Insert
$user = new User();
$user->setEmail('user@example.com');
$user->setName('John Doe');

$entityManager->persist($user);
$entityManager->flush();

// DQL (Doctrine Query Language)
$qb = $entityManager->createQueryBuilder();
$users = $qb->select('u')
    ->from(User::class, 'u')
    ->where('u.age > :age')
    ->setParameter('age', $minAge)
    ->getQuery()
    ->getResult();
```

### Plain PDO

```php
<?php
// ✅ ALWAYS use PDO with prepared statements

$pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');

// Query
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email AND status = :status');
$stmt->execute(['email' => $email, 'status' => $status]);
$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

// Insert
$stmt = $pdo->prepare('INSERT INTO users (email, name) VALUES (:email, :name)');
$stmt->execute([
    'email' => $email,
    'name' => $name
]);
$userId = $pdo->lastInsertId();

// ❌ NEVER DO THIS (SQL Injection)
$badQuery = "SELECT * FROM users WHERE email = '$email'";
$result = $pdo->query($badQuery);
```

---

## Kotlin

### 📦 Dependencies

| Approach | Library | Gradle | Use Case |
|----------|---------|--------|----------|
| **Exposed** ⭐ | `org.jetbrains.exposed` | See below | Kotlin SQL DSL |
| **Room** | `androidx.room` | See below | Android local DB |
| **Spring Data JPA** | `spring-boot-starter-data-jpa` | See below | Spring Boot |

### Exposed (SQL DSL)

```kotlin
// build.gradle.kts
dependencies {
    implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    implementation("org.jetbrains.exposed:exposed-dao:0.40.1")
    implementation("org.jetbrains.exposed:exposed-jdbc:0.40.1")
    implementation("org.postgresql:postgresql:42.5.0")
}
```

```kotlin
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.transaction

object Users : Table() {
    val id = integer("id").autoIncrement()
    val email = varchar("email", 255)
    val name = varchar("name", 100)
    override val primaryKey = PrimaryKey(id)
}

// Query
val user = transaction {
    Users.select { Users.id eq userId }.singleOrNull()
}

val users = transaction {
    Users.select {
        (Users.age greater minAge) and (Users.status eq status)
    }.toList()
}

// Insert
val userId = transaction {
    Users.insert {
        it[email] = "user@example.com"
        it[name] = "John Doe"
    } get Users.id
}
```

### Room (Android)

```kotlin
// build.gradle
dependencies {
    implementation("androidx.room:room-runtime:2.5.0")
    kapt("androidx.room:room-compiler:2.5.0")
    implementation("androidx.room:room-ktx:2.5.0")
}
```

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserById(userId: Int): User?
    
    @Query("SELECT * FROM users WHERE age > :minAge AND status = :status")
    suspend fun getUsersByAgeAndStatus(minAge: Int, status: String): List<User>
    
    @Insert
    suspend fun insert(user: User): Long
    
    @Update
    suspend fun update(user: User)
}

// Usage
val user = userDao.getUserById(123)
val users = userDao.getUsersByAgeAndStatus(18, "active")
```

---

## Swift

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **CoreData** ⭐ | Built-in iOS/macOS | No install | Apple platform local DB |
| **SQLite.swift** | `SQLite.swift` | Swift Package Manager | SQLite wrapper |
| **GRDB** | `GRDB.swift` | Swift Package Manager | SQLite toolkit |

### CoreData (Apple Platforms)

```swift
// No installation needed - built into iOS/macOS

import CoreData

// Query
let fetchRequest: NSFetchRequest<User> = User.fetchRequest()
fetchRequest.predicate = NSPredicate(
    format: "email == %@ AND status == %@",
    email, status
)

do {
    let users = try context.fetch(fetchRequest)
} catch {
    print("Fetch failed: \(error)")
}

// Insert
let newUser = User(context: context)
newUser.email = "user@example.com"
newUser.name = "John Doe"

do {
    try context.save()
} catch {
    print("Save failed: \(error)")
}
```

### SQLite.swift

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/stephencelis/SQLite.swift.git", from: "0.14.1")
]
```

```swift
import SQLite

let db = try Connection("path/to/db.sqlite3")
let users = Table("users")
let id = Expression<Int64>("id")
let email = Expression<String>("email")
let age = Expression<Int>("age")

// Query with parameters
for user in try db.prepare(users.filter(age > minAge && status == statusValue)) {
    print("User: \(user[email])")
}

// Insert
let insert = users.insert(
    email <- "user@example.com",
    name <- "John Doe"
)
let rowId = try db.run(insert)

// Raw query with bindings (safe)
let stmt = try db.prepare("SELECT * FROM users WHERE email = ? AND status = ?")
for row in try stmt.bind(email, status) {
    // Process row
}
```

---

## Dart (Flutter)

### 📦 Dependencies

| Approach | Library | Installation | Use Case |
|----------|---------|--------------|----------|
| **Drift** ⭐ | `drift` | See below | Type-safe SQLite ORM |
| **Floor** | `floor` | See below | Room-inspired ORM |
| **sqflite** | `sqflite` | `flutter pub add sqflite` | Raw SQLite |

### Drift (Moor - Recommended)

```yaml
# pubspec.yaml
dependencies:
  drift: ^2.x.x
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.0.0
  path: ^1.8.0

dev_dependencies:
  drift_dev: ^2.x.x
  build_runner: ^2.0.0
```

```dart
import 'package:drift/drift.dart';

@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  // Query
  Future<User?> getUserById(int id) {
    return (select(users)..where((u) => u.id.equals(id))).getSingleOrNull();
  }
  
  Future<List<User>> getUsersOlderThan(int minAge) {
    return (select(users)
      ..where((u) => u.age.isBiggerThanValue(minAge))
      ..where((u) => u.status.equals('active')))
    .get();
  }
  
  // Insert
  Future<int> insertUser(UsersCompanion user) {
    return into(users).insert(user);
  }
}
```

### Floor (Room-like)

```yaml
# pubspec.yaml
dependencies:
  floor: ^1.4.0

dev_dependencies:
  floor_generator: ^1.4.0
  build_runner: ^2.0.0
```

```dart
@dao
abstract class UserDao {
  @Query('SELECT * FROM User WHERE id = :id')
  Future<User?> findUserById(int id);

  @Query('SELECT * FROM User WHERE age > :minAge AND status = :status')
  Future<List<User>> findUsersByAgeAndStatus(int minAge, String status);

  @insert
  Future<void> insertUser(User user);
}
```

### Plain sqflite

```dart
// Install: flutter pub add sqflite
import 'package:sqflite/sqflite.dart';

final db = await openDatabase('users.db');

// Query with parameters
final List<Map<String, dynamic>> maps = await db.query(
  'users',
  where: 'email = ? AND status = ?',
  whereArgs: [email, status],
);

final users = maps.map((map) => User.fromMap(map)).toList();

// Insert
final id = await db.insert(
  'users',
  {
    'email': 'user@example.com',
    'name': 'John Doe',
  },
  conflictAlgorithm: ConflictAlgorithm.replace,
);

// ❌ NEVER DO THIS (SQL Injection)
final badQuery = "SELECT * FROM users WHERE email = '$email'";
await db.rawQuery(badQuery);
```

---

## Best Practices

✅ **DO**:
- Use parameterized queries ALWAYS (prevents SQL injection)
- Use ORM for type safety and productivity
- Close connections properly
- Use connection pooling in production
- Handle errors gracefully
- Use transactions for multi-step operations
- Index frequently queried columns

❌ **DON'T**:
- Concatenate user input into queries
- Trust client-provided SQL
- Leave connections open
- Use SELECT * in production (specify columns)
- Ignore connection limits
- Query in loops (N+1 problem)
- Expose database schema in error messages

---

## Security Checklist

- [ ] All queries use parameters/bindings (no string concatenation)
- [ ] ORM escapes input automatically
- [ ] Least privilege database user (not root/admin)
- [ ] Prepared statements for all raw queries
- [ ] Input validated before queries (see input-validation.md)
- [ ] Error messages don't expose schema details
- [ ] Connection strings secured (environment variables, not in code)
- [ ] SSL/TLS enabled for remote databases

---

## Performance Tips

```typescript
// ❌ N+1 Problem - Queries in loop
for (const user of users) {
  const posts = await fetchPosts(user.id); // BAD: N queries!
}

// ✅ Eager Loading - Single query with JOIN
const users = await prisma.user.findMany({
  include: { posts: true }
});

// ✅ Batch Loading - 2 queries total
const userIds = users.map(u => u.id);
const posts = await prisma.post.findMany({
  where: { userId: { in: userIds } }
});
```

---

## Transaction Example

```typescript
// TypeScript (Prisma)
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  await tx.account.create({ data: { userId: user.id } });
  return user;
});

// Python (SQLAlchemy)
with session.begin():
    user = User(**user_data)
    session.add(user)
    account = Account(user_id=user.id)
    session.add(account)

// Java (Spring)
@Transactional
public User createUserWithAccount(UserData data) {
    User user = userRepository.save(new User(data));
    accountRepository.save(new Account(user.getId()));
    return user;
}

// C# (EF Core)
using var transaction = await _context.Database.BeginTransactionAsync();
try {
    var user = new User { ... };
    _context.Users.Add(user);
    await _context.SaveChangesAsync();
    
    var account = new Account { UserId = user.Id };
    _context.Accounts.Add(account);
    await _context.SaveChangesAsync();
    
    await transaction.CommitAsync();
} catch {
    await transaction.RollbackAsync();
    throw;
}

// PHP (Laravel)
DB::transaction(function () use ($userData) {
    $user = User::create($userData);
    Account::create(['user_id' => $user->id]);
});
```

---

## Quick Decision Guide

```
Need to query a database?
    ↓
Using a framework? ──YES──→ Use framework ORM (Laravel, Django, Spring)
    ↓ NO
Need type safety? ──YES──→ Use modern ORM (Prisma, SQLAlchemy, EF Core)
    ↓ NO
Need performance? ──YES──→ Use query builder or micro ORM (Knex, Dapper)
    ↓ NO
Simple queries? ──YES──→ Use plain driver with prepared statements
    ↓
ALWAYS use parameterized queries (never string concatenation)
```
