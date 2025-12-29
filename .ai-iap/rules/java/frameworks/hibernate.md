# Hibernate ORM

> **Scope**: Apply these rules when using Hibernate in Java projects
> **Applies to**: Java files using Hibernate
> **Extends**: java/architecture.md, java/code-style.md

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use @Entity with proper mapping annotations
> **ALWAYS**: Use Criteria API or JPQL (NOT raw SQL)
> **ALWAYS**: Use transactions for write operations
> **ALWAYS**: Use fetch joins to avoid N+1 queries
> **ALWAYS**: Close SessionFactory on application shutdown
> 
> **NEVER**: Use raw SQL without parameterization (SQL injection)
> **NEVER**: Load collections in loops (N+1 problem)
> **NEVER**: Forget @Transactional on write methods
> **NEVER**: Use lazy loading without open session
> **NEVER**: Store Session in instance variables (not thread-safe)

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| Entity Mapping | Domain models | `@Entity`, `@Table`, `@Column` |
| Criteria API | Type-safe queries | `CriteriaBuilder`, type-safe |
| JPQL | String-based queries | HQL syntax |
| Fetch Joins | Avoid N+1 | `JOIN FETCH` |
| Named Queries | Reusable queries | `@NamedQuery` |

## Core Patterns

### Entity Mapping
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Post> posts = new ArrayList<>();
    
    @CreationTimestamp
    @Column(updatable = false)
    private LocalDateTime createdAt;
}
```

### Criteria API (Type-Safe)
```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> query = cb.createQuery(User.class);
Root<User> root = query.from(User.class);

query.select(root)
    .where(cb.equal(root.get("email"), email));

User user = session.createQuery(query).getSingleResult();
```

### JPQL with Fetch Join
```java
String jpql = "SELECT u FROM User u JOIN FETCH u.posts WHERE u.id = :id";
User user = session.createQuery(jpql, User.class)
    .setParameter("id", userId)
    .getSingleResult();
```

### Transaction Management
```java
Transaction tx = null;
try {
    tx = session.beginTransaction();
    
    User user = new User();
    user.setName("John");
    user.setEmail("john@example.com");
    
    session.persist(user);
    tx.commit();
} catch (Exception e) {
    if (tx != null) tx.rollback();
    throw e;
}
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **N+1 Queries** | Loop with lazy load | `JOIN FETCH` | Performance killer |
| **Raw SQL** | Unparameterized SQL | Criteria API or parameterized | SQL injection |
| **No Transaction** | Write without transaction | `@Transactional` or manual | Data inconsistency |
| **Lazy Load Outside Session** | Access lazy collection after close | Eager fetch or open session | LazyInitializationException |
| **Session as Field** | Store Session in class field | Method-local Session | Not thread-safe |

### Anti-Pattern: N+1 Queries (PERFORMANCE DISASTER)
```java
// ❌ WRONG - N+1 queries
List<User> users = session.createQuery("FROM User", User.class).list();
for (User user : users) {
    user.getPosts().size();  // Separate query for EACH user!
}

// ✅ CORRECT - Fetch join
String jpql = "SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.posts";
List<User> users = session.createQuery(jpql, User.class).list();
```

### Anti-Pattern: Raw SQL Without Parameters (SQL INJECTION)
```java
// ❌ WRONG - SQL injection vulnerability
String sql = "SELECT * FROM users WHERE email = '" + email + "'";
Query query = session.createNativeQuery(sql, User.class);

// ✅ CORRECT - Parameterized query
String jpql = "FROM User WHERE email = :email";
User user = session.createQuery(jpql, User.class)
    .setParameter("email", email)
    .getSingleResult();
```

## AI Self-Check (Verify BEFORE generating Hibernate code)

- [ ] Using @Entity with proper annotations?
- [ ] Criteria API or JPQL (NOT raw SQL)?
- [ ] Parameterized queries for all user input?
- [ ] Transactions for write operations?
- [ ] Fetch joins to avoid N+1?
- [ ] Cascade operations configured?
- [ ] No Session stored in fields?
- [ ] Lazy loading handled properly?
- [ ] SessionFactory properly configured?
- [ ] Following Hibernate best practices?

## Configuration

```xml
<!-- hibernate.cfg.xml -->
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
        <property name="hibernate.connection.url">jdbc:postgresql://localhost/mydb</property>
        <property name="hibernate.dialect">org.hibernate.dialect.PostgreSQLDialect</property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="hibernate.show_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

## Relationships

| Type | Annotation | Example |
|------|-----------|---------|
| One-to-Many | `@OneToMany(mappedBy="...")` | User → Posts |
| Many-to-One | `@ManyToOne` | Post → User |
| Many-to-Many | `@ManyToMany` | Users ↔ Roles |
| One-to-One | `@OneToOne` | User → Profile |

## Cascade Operations

```java
@OneToMany(
    mappedBy = "author",
    cascade = CascadeType.ALL,  // Propagate all operations
    orphanRemoval = true  // Delete orphaned entities
)
private List<Post> posts;
```

## Key Features

- **Automatic DDL**: Schema generation
- **Caching**: First/second level cache
- **Lazy Loading**: On-demand loading
- **Criteria API**: Type-safe queries
- **HQL/JPQL**: Object-oriented queries

## Key Annotations

- `@Entity`: Mark as entity
- `@Id`: Primary key
- `@GeneratedValue`: Auto-generate ID
- `@Column`: Column mapping
- `@ManyToOne/@OneToMany`: Relationships
- `@Transient`: Exclude from persistence
