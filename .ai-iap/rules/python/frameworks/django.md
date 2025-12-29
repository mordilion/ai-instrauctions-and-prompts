# Django Framework

> **Scope**: Apply these rules when working with Django 4+ applications
> **Applies to**: Python files in Django projects
> **Extends**: python/architecture.md, python/code-style.md
> **Precedence**: Framework rules OVERRIDE Python rules for Django-specific patterns

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use Django ORM querysets (type-safe, SQL injection protection)
> **ALWAYS**: Use Django forms or serializers for validation
> **ALWAYS**: Use class-based views (CBVs) or function-based views (FBVs) consistently
> **ALWAYS**: Use Django's built-in authentication system
> **ALWAYS**: Use migrations for all schema changes
> 
> **NEVER**: Use raw SQL without parameterization (SQL injection risk)
> **NEVER**: Skip CSRF protection (security vulnerability)
> **NEVER**: Store sensitive data in settings.py (use environment variables)
> **NEVER**: Use syncdb (deprecated, use migrations)
> **NEVER**: Query in templates (N+1 problem)

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| Class-Based Views (CBV) | CRUD operations, reusability | `ListView`, `CreateView`, `UpdateView` |
| Function-Based Views (FBV) | Custom logic, simple views | `@require_http_methods`, decorators |
| Django REST Framework | APIs | `APIView`, `ViewSet`, `Serializer` |
| ORM QuerySets | Database queries | `filter()`, `exclude()`, `select_related()` |
| Forms/ModelForms | Data validation | `Form`, `ModelForm`, `is_valid()` |

## Core Patterns

### Model Definition
```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [models.Index(fields=['-created_at'])]
    
    def __str__(self):
        return self.title
```

### Class-Based View
```python
from django.views.generic import ListView, CreateView
from django.contrib.auth.mixins import LoginRequiredMixin

class PostListView(ListView):
    model = Post
    template_name = 'posts/list.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        return Post.objects.select_related('author').filter(published=True)

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'posts/create.html'
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)
```

### Function-Based View
```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required

@login_required
def post_detail(request, pk):
    post = get_object_or_404(Post.objects.select_related('author'), pk=pk)
    return render(request, 'posts/detail.html', {'post': post})

@login_required
def post_create(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'posts/create.html', {'form': form})
```

### Django REST Framework (API)
```python
from rest_framework import viewsets, serializers
from rest_framework.permissions import IsAuthenticated

class PostSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(source='author.username', read_only=True)
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author_name', 'created_at']
        read_only_fields = ['author_name', 'created_at']

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.select_related('author')
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticated]
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **Raw SQL** | `cursor.execute(f"... {user_input}")` | Use ORM or parameterized queries | SQL injection vulnerability |
| **N+1 Queries** | `for post in posts: post.author.name` | `select_related('author')` | Performance disaster |
| **No CSRF** | Disable CSRF protection | Keep CSRF enabled, use `{% csrf_token %}` | Security vulnerability |
| **Settings in Code** | `SECRET_KEY = 'hardcoded'` | Use `os.environ.get()` | Secret exposure |
| **Template Queries** | `{% for user in User.objects.all %}` | Pass data via context | N+1, breaks separation |

### Anti-Pattern: Raw SQL Without Params (SQL INJECTION)
```python
# ❌ WRONG - SQL injection vulnerability
def search_users(query):
    cursor.execute(f"SELECT * FROM users WHERE name = '{query}'")  # DANGEROUS!
    return cursor.fetchall()

# ✅ CORRECT - Use ORM
def search_users(query):
    return User.objects.filter(name__icontains=query)

# ✅ CORRECT - Parameterized if raw SQL needed
def search_users(query):
    return User.objects.raw("SELECT * FROM users WHERE name = %s", [query])
```

### Anti-Pattern: N+1 Queries (PERFORMANCE KILLER)
```python
# ❌ WRONG - N+1 queries
posts = Post.objects.all()
for post in posts:
    print(post.author.name)  # Separate query for EACH post!

# ✅ CORRECT - select_related (1 query with JOIN)
posts = Post.objects.select_related('author').all()
for post in posts:
    print(post.author.name)  # No additional query

# ✅ CORRECT - prefetch_related (2 queries, for many-to-many)
posts = Post.objects.prefetch_related('tags').all()
```

## AI Self-Check (Verify BEFORE generating Django code)

- [ ] Using Django ORM (NOT raw SQL without params)?
- [ ] Using select_related/prefetch_related to avoid N+1?
- [ ] Forms/serializers for validation?
- [ ] CSRF protection enabled?
- [ ] Secrets in environment variables (NOT settings.py)?
- [ ] Using migrations for schema changes?
- [ ] Authentication via Django's system?
- [ ] No queries in templates?
- [ ] Type hints for function parameters?
- [ ] Following Django naming conventions?

## URL Patterns

```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.PostListView.as_view(), name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('post/create/', views.post_create, name='post_create'),
]
```

## Migrations

```bash
# Create migration after model changes
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Show migration SQL
python manage.py sqlmigrate app_name 0001
```

## QuerySet Optimization

| Method | Purpose | Use Case |
|--------|---------|----------|
| select_related() | JOIN for ForeignKey/OneToOne | Reduce queries |
| prefetch_related() | Separate query for ManyToMany | Reduce queries |
| only() | Fetch specific fields | Reduce data transfer |
| defer() | Exclude specific fields | Reduce data transfer |
| values() | Return dict instead of model | Performance |

## Key Features

- **ORM**: Type-safe database queries
- **Admin**: Auto-generated admin interface
- **Auth**: Built-in user authentication
- **Forms**: Data validation and rendering
- **Middleware**: Request/response processing
- **Signals**: Event-driven actions

## Key Libraries

- **Django REST Framework**: API development
- **django-crispy-forms**: Form rendering
- **django-filter**: QuerySet filtering
- **Celery**: Async task queue
