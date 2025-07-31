# Complete Django Tutorial with Project

## Table of Contents
1. [What is Django?](#what-is-django)
2. [Installation and Setup](#installation-and-setup)
3. [Django Project Structure](#django-project-structure)
4. [Django Apps and MVT Pattern](#django-apps-and-mvt-pattern)
5. [Models and Database](#models-and-database)
6. [Views and URLs](#views-and-urls)
7. [Templates](#templates)
8. [Forms](#forms)
9. [Admin Interface](#admin-interface)
10. [Authentication and Authorization](#authentication-and-authorization)
11. [Static Files and Media](#static-files-and-media)
12. [Complete Project: Blog Application](#complete-project-blog-application)
13. [REST API with Django REST Framework](#rest-api-with-django-rest-framework)
14. [Testing](#testing)
15. [Deployment](#deployment)

## What is Django?

Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. It follows the "batteries-included" philosophy, providing many built-in features.

**Key Features:**
- **MTV (Model-Template-View) Architecture**
- **Object-Relational Mapping (ORM)**
- **Automatic Admin Interface**
- **Built-in Authentication System**
- **URL Routing**
- **Template Engine**
- **Form Handling**
- **Security Features** (CSRF, XSS protection, etc.)
- **Internationalization Support**

**Django Philosophy:**
- **DRY (Don't Repeat Yourself)**
- **Convention over Configuration**
- **Loose Coupling**
- **Explicit is better than implicit**

## Installation and Setup

### Prerequisites
- Python 3.8 or higher
- pip (Python package installer)

### Step 1: Create Virtual Environment
```bash
# Create project directory
mkdir django-tutorial
cd django-tutorial

# Create virtual environment
python -m venv django-env

# Activate virtual environment
# On Windows:
django-env\Scripts\activate
# On macOS/Linux:
source django-env/bin/activate
```

### Step 2: Install Django
```bash
pip install Django
# For specific version: pip install Django==4.2.7
```

### Step 3: Verify Installation
```bash
python -m django --version
```

### Step 4: Create Django Project
```bash
django-admin startproject myproject
cd myproject
```

### Step 5: Run Development Server
```bash
python manage.py runserver
```
Visit `http://127.0.0.1:8000` to see the Django welcome page.

## Django Project Structure

### Initial Project Structure
```
myproject/
├── manage.py          # Command-line utility
├── myproject/         # Project package
│   ├── __init__.py
│   ├── settings.py    # Project settings
│   ├── urls.py        # URL configuration
│   ├── wsgi.py        # WSGI application
│   └── asgi.py        # ASGI application
└── db.sqlite3         # SQLite database (created after first migration)
```

### Key Files Explained

**manage.py:**
- Command-line interface for Django project
- Used for running server, migrations, creating apps, etc.

**settings.py:**
```python
# Key settings to understand
DEBUG = True  # Never True in production
ALLOWED_HOSTS = []  # Domains that can serve the app

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Your apps will go here
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

STATIC_URL = '/static/'
```

**urls.py:**
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    # Your URL patterns will go here
]
```

## Django Apps and MVT Pattern

### Creating a Django App
```bash
python manage.py startapp blog
```

### App Structure
```
blog/
├── __init__.py
├── admin.py           # Admin configuration
├── apps.py            # App configuration
├── models.py          # Database models
├── views.py           # View functions/classes
├── urls.py            # URL patterns (create this)
├── tests.py           # Test cases
├── migrations/        # Database migrations
│   └── __init__.py
├── templates/         # HTML templates (create this)
└── static/           # CSS, JS, images (create this)
```

### Register App in Settings
```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',  # Add your app here
]
```

### MVT Pattern Explained

**Model (M):** Data layer - defines database structure
**View (V):** Logic layer - processes requests and returns responses
**Template (T):** Presentation layer - HTML with Django template language

## Models and Database

### Defining Models

**blog/models.py:**
```python
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse
from django.utils import timezone

class Category(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name_plural = "Categories"
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def get_absolute_url(self):
        return reverse('blog:category_detail', kwargs={'pk': self.pk})

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    
    def __str__(self):
        return self.name

class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
    ]
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, blank=True)
    tags = models.ManyToManyField(Tag, blank=True)
    content = models.TextField()
    excerpt = models.TextField(max_length=500, blank=True)
    featured_image = models.ImageField(upload_to='posts/', blank=True, null=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published_at = models.DateTimeField(null=True, blank=True)
    views = models.PositiveIntegerField(default=0)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', 'created_at']),
        ]
    
    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        return reverse('blog:post_detail', kwargs={'slug': self.slug})
    
    def save(self, *args, **kwargs):
        if self.status == 'published' and not self.published_at:
            self.published_at = timezone.now()
        super().save(*args, **kwargs)

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    approved = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['created_at']
    
    def __str__(self):
        return f'Comment by {self.author.username} on {self.post.title}'
```

### Field Types Overview
```python
# Common Django model fields
CharField(max_length=100)           # Short text
TextField()                         # Long text
IntegerField()                      # Integer numbers
FloatField()                        # Decimal numbers
BooleanField(default=False)         # True/False
DateField()                         # Date only
DateTimeField(auto_now_add=True)    # Timestamp (creation)
DateTimeField(auto_now=True)        # Timestamp (last update)
EmailField()                        # Email validation
URLField()                          # URL validation
ImageField(upload_to='images/')     # Image uploads
FileField(upload_to='files/')       # File uploads
SlugField()                         # URL-friendly strings
JSONField()                         # JSON data (Django 3.1+)

# Relationship fields
ForeignKey(Model, on_delete=models.CASCADE)    # One-to-many
ManyToManyField(Model)                         # Many-to-many
OneToOneField(Model, on_delete=models.CASCADE) # One-to-one
```

### Database Operations

**Create and Run Migrations:**
```bash
# Create migration files
python manage.py makemigrations

# Apply migrations to database
python manage.py migrate

# Check migration status
python manage.py showmigrations

# Create specific migration
python manage.py makemigrations blog --name create_post_model
```

**Django Shell for Database Operations:**
```bash
python manage.py shell
```

```python
# In Django shell
from blog.models import Post, Category, Tag
from django.contrib.auth.models import User

# Create objects
category = Category.objects.create(name="Technology", description="Tech posts")
user = User.objects.get(username='admin')
post = Post.objects.create(
    title="My First Post",
    slug="my-first-post",
    author=user,
    category=category,
    content="This is my first post content.",
    status='published'
)

# Query objects
all_posts = Post.objects.all()
published_posts = Post.objects.filter(status='published')
recent_posts = Post.objects.filter(created_at__gte='2024-01-01')
post = Post.objects.get(slug='my-first-post')

# Update objects
post.title = "Updated Title"
post.save()

# Delete objects
post.delete()

# Complex queries
from django.db.models import Q, Count
posts_with_comments = Post.objects.annotate(comment_count=Count('comments'))
search_posts = Post.objects.filter(
    Q(title__icontains='django') | Q(content__icontains='django')
)
```

## Views and URLs

### Function-Based Views (FBV)

**blog/views.py:**
```python
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse, Http404
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from django.core.paginator import Paginator
from django.db.models import Q
from .models import Post, Category, Tag, Comment
from .forms import CommentForm

def post_list(request):
    posts = Post.objects.filter(status='published').select_related('author', 'category')
    
    # Search functionality
    search_query = request.GET.get('search')
    if search_query:
        posts = posts.filter(
            Q(title__icontains=search_query) | 
            Q(content__icontains=search_query)
        )
    
    # Category filter
    category_id = request.GET.get('category')
    if category_id:
        posts = posts.filter(category_id=category_id)
    
    # Pagination
    paginator = Paginator(posts, 5)  # 5 posts per page
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    categories = Category.objects.all()
    
    context = {
        'page_obj': page_obj,
        'categories': categories,
        'search_query': search_query,
    }
    return render(request, 'blog/post_list.html', context)

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug, status='published')
    
    # Increment view count
    post.views += 1
    post.save(update_fields=['views'])
    
    # Get comments
    comments = post.comments.filter(approved=True).select_related('author')
    
    # Handle comment form
    if request.method == 'POST':
        if request.user.is_authenticated:
            form = CommentForm(request.POST)
            if form.is_valid():
                comment = form.save(commit=False)
                comment.post = post
                comment.author = request.user
                comment.save()
                messages.success(request, 'Your comment has been submitted for approval.')
                return redirect('blog:post_detail', slug=slug)
        else:
            messages.error(request, 'Please log in to comment.')
            return redirect('login')
    else:
        form = CommentForm()
    
    # Related posts
    related_posts = Post.objects.filter(
        category=post.category,
        status='published'
    ).exclude(id=post.id)[:3]
    
    context = {
        'post': post,
        'comments': comments,
        'form': form,
        'related_posts': related_posts,
    }
    return render(request, 'blog/post_detail.html', context)

def category_detail(request, pk):
    category = get_object_or_404(Category, pk=pk)
    posts = Post.objects.filter(category=category, status='published')
    
    paginator = Paginator(posts, 5)
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    context = {
        'category': category,
        'page_obj': page_obj,
    }
    return render(request, 'blog/category_detail.html', context)

@login_required
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            form.save_m2m()  # Save many-to-many relationships
            messages.success(request, 'Post created successfully!')
            return redirect('blog:post_detail', slug=post.slug)
    else:
        form = PostForm()
    
    return render(request, 'blog/create_post.html', {'form': form})
```

### Class-Based Views (CBV)

**blog/views.py (additional CBVs):**
```python
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.urls import reverse_lazy

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 5
    
    def get_queryset(self):
        queryset = Post.objects.filter(status='published').select_related('author', 'category')
        
        search_query = self.request.GET.get('search')
        if search_query:
            queryset = queryset.filter(
                Q(title__icontains=search_query) | 
                Q(content__icontains=search_query)
            )
        
        category_id = self.request.GET.get('category')
        if category_id:
            queryset = queryset.filter(category_id=category_id)
        
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['categories'] = Category.objects.all()
        context['search_query'] = self.request.GET.get('search', '')
        return context

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    context_object_name = 'post'
    
    def get_queryset(self):
        return Post.objects.filter(status='published')
    
    def get_object(self):
        obj = super().get_object()
        obj.views += 1
        obj.save(update_fields=['views'])
        return obj
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['comments'] = self.object.comments.filter(approved=True).select_related('author')
        context['form'] = CommentForm()
        context['related_posts'] = Post.objects.filter(
            category=self.object.category,
            status='published'
        ).exclude(id=self.object.id)[:3]
        return context

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/create_post.html'
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        messages.success(self.request, 'Post created successfully!')
        return super().form_valid(form)

class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/update_post.html'
    
    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
    
    def form_valid(self, form):
        messages.success(self.request, 'Post updated successfully!')
        return super().form_valid(form)

class PostDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    model = Post
    template_name = 'blog/delete_post.html'
    success_url = reverse_lazy('blog:post_list')
    
    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
    
    def delete(self, request, *args, **kwargs):
        messages.success(self.request, 'Post deleted successfully!')
        return super().delete(request, *args, **kwargs)
```

### URL Configuration

**blog/urls.py:**
```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # Function-based views
    path('', views.post_list, name='post_list'),
    path('post/<slug:slug>/', views.post_detail, name='post_detail'),
    path('category/<int:pk>/', views.category_detail, name='category_detail'),
    path('create/', views.create_post, name='create_post'),
    
    # Class-based views (alternative)
    # path('', views.PostListView.as_view(), name='post_list'),
    # path('post/<slug:slug>/', views.PostDetailView.as_view(), name='post_detail'),
    # path('create/', views.PostCreateView.as_view(), name='create_post'),
    # path('post/<slug:slug>/update/', views.PostUpdateView.as_view(), name='post_update'),
    # path('post/<slug:slug>/delete/', views.PostDeleteView.as_view(), name='post_delete'),
]
```

**myproject/urls.py:**
```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
    path('accounts/', include('django.contrib.auth.urls')),
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Templates

### Template Configuration

**settings.py:**
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # Project-level templates
        'APP_DIRS': True,  # Look for templates in app directories
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

### Template Structure
```
templates/
├── base.html
├── blog/
│   ├── post_list.html
│   ├── post_detail.html
│   ├── category_detail.html
│   ├── create_post.html
│   └── update_post.html
└── registration/
    ├── login.html
    └── signup.html
```

### Base Template

**templates/base.html:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Blog{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.7.2/font/bootstrap-icons.css">
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="{% url 'blog:post_list' %}">
                <i class="bi bi-journal-text"></i> My Blog
            </a>
            
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="{% url 'blog:post_list' %}">Home</a>
                    </li>
                    {% if user.is_authenticated %}
                        <li class="nav-item">
                            <a class="nav-link" href="{% url 'blog:create_post' %}">Write Post</a>
                        </li>
                    {% endif %}
                </ul>
                
                <ul class="navbar-nav">
                    {% if user.is_authenticated %}
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" 
                               data-bs-toggle="dropdown">
                                <i class="bi bi-person-circle"></i> {{ user.username }}
                            </a>
                            <ul class="dropdown-menu">
                                <li><a class="dropdown-item" href="{% url 'admin:index' %}">Admin</a></li>
                                <li><hr class="dropdown-divider"></li>
                                <li><a class="dropdown-item" href="{% url 'logout' %}">Logout</a></li>
                            </ul>
                        </li>
                    {% else %}
                        <li class="nav-item">
                            <a class="nav-link" href="{% url 'login' %}">Login</a>
                        </li>
                    {% endif %}
                </ul>
            </div>
        </div>
    </nav>

    <main class="container mt-4">
        {% if messages %}
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        {% endif %}

        <div class="row">
            <div class="col-md-8">
                {% block content %}
                {% endblock %}
            </div>
            
            <div class="col-md-4">
                {% block sidebar %}
                    <div class="card">
                        <div class="card-header">
                            <h5>Search</h5>
                        </div>
                        <div class="card-body">
                            <form method="GET" action="{% url 'blog:post_list' %}">
                                <div class="input-group">
                                    <input type="text" class="form-control" name="search" 
                                           placeholder="Search posts..." value="{{ search_query }}">
                                    <button class="btn btn-outline-primary" type="submit">
                                        <i class="bi bi-search"></i>
                                    </button>
                                </div>
                            </form>
                        </div>
                    </div>
                    
                    <div class="card mt-3">
                        <div class="card-header">
                            <h5>Categories</h5>
                        </div>
                        <div class="card-body">
                            {% for category in categories %}
                                <a href="{% url 'blog:category_detail' category.pk %}" 
                                   class="badge bg-secondary text-decoration-none me-2 mb-2">
                                    {{ category.name }}
                                </a>
                            {% endfor %}
                        </div>
                    </div>
                {% endblock %}
            </div>
        </div>
    </main>

    <footer class="bg-light mt-5 py-4">
        <div class="container text-center">
            <p>&copy; 2024 My Blog. Built with Django.</p>
        </div>
    </footer>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>
```

### Post List Template

**templates/blog/post_list.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}
    {% if search_query %}Search Results for "{{ search_query }}"{% else %}Latest Posts{% endif %}
{% endblock %}

{% block content %}
    <h1>
        {% if search_query %}
            Search Results for "{{ search_query }}"
        {% else %}
            Latest Posts
        {% endif %}
    </h1>

    {% if page_obj %}
        {% for post in page_obj %}
            <article class="card mb-4">
                {% if post.featured_image %}
                    <img src="{{ post.featured_image.url }}" class="card-img-top" alt="{{ post.title }}">
                {% endif %}
                
                <div class="card-body">
                    <h2 class="card-title">
                        <a href="{{ post.get_absolute_url }}" class="text-decoration-none">
                            {{ post.title }}
                        </a>
                    </h2>
                    
                    <div class="text-muted mb-2">
                        <small>
                            <i class="bi bi-person"></i> By {{ post.author.get_full_name|default:post.author.username }}
                            <i class="bi bi-calendar ms-3"></i> {{ post.published_at|date:"M d, Y" }}
                            {% if post.category %}
                                <i class="bi bi-folder ms-3"></i> 
                                <a href="{% url 'blog:category_detail' post.category.pk %}">{{ post.category.name }}</a>
                            {% endif %}
                            <i class="bi bi-eye ms-3"></i> {{ post.views }} views
                        </small>
                    </div>
                    
                    <p class="card-text">
                        {% if post.excerpt %}
                            {{ post.excerpt }}
                        {% else %}
                            {{ post.content|truncatewords:30 }}
                        {% endif %}
                    </p>
                    
                    <div class="d-flex justify-content-between align-items-center">
                        <a href="{{ post.get_absolute_url }}" class="btn btn-primary">Read More</a>
                        
                        {% if post.tags.exists %}
                            <div>
                                {% for tag in post.tags.all %}
                                    <span class="badge bg-light text-dark">#{{ tag.name }}</span>
                                {% endfor %}
                            </div>
                        {% endif %}
                    </div>
                </div>
            </article>
        {% endfor %}

        <!-- Pagination -->
        {% if page_obj.has_other_pages %}
            <nav aria-label="Page navigation">
                <ul class="pagination justify-content-center">
                    {% if page_obj.has_previous %}
                        <li class="page-item">
                            <a class="page-link" href="?page=1{% if search_query %}&search={{ search_query }}{% endif %}">First</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.previous_page_number }}{% if search_query %}&search={{ search_query }}{% endif %}">Previous</a>
                        </li>
                    {% endif %}

                    <li class="page-item active">
                        <span class="page-link">
                            Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}
                        </span>
                    </li>

                    {% if page_obj.has_next %}
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.next_page_number }}{% if search_query %}&search={{ search_query }}{% endif %}">Next</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.paginator.num_pages }}{% if search_query %}&search={{ search_query }}{% endif %}">Last</a>
                        </li>
                    {% endif %}
                </ul>
            </nav>
        {% endif %}
    {% else %}
        <div class="alert alert-info">
            <h4>No posts found</h4>
            {% if search_query %}
                <p>No posts match your search query "{{ search_query }}".</p>
            {% else %}
                <p>No posts have been published yet.</p>
            {% endif %}
        </div>
    {% endif %}
{% endblock %}
```

### Template Tags and Filters

**Custom Template Tags (blog/templatetags/blog_tags.py):**
```python
from django import template
from django.db.models import Count
from ..models import Post, Category

register = template.Library()

@register.simple_tag
def total_posts():
    return Post.objects.filter(status='published').count()

@register.inclusion_tag('blog/latest_posts.html')
def show_latest_posts(count=5):
    latest_posts = Post.objects.filter(status='published')[:count]
    return {'latest_posts': latest_posts}

@register.filter
def markdown(value):
    """Convert markdown to HTML"""
    import markdown
    return markdown.markdown(value, extensions=['markdown.extensions.fenced_code'])

# Usage in templates:
# {% load blog_tags %}
# {% total_posts %} posts published
# {% show_latest_posts 3 %}
# {{ post.content|markdown|safe }}
```

## Forms

### Django Forms

**blog/forms.py:**
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
from .models import Post, Comment, Category, Tag

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'slug', 'category', 'tags', 'content', 'excerpt', 
                 'featured_image', 'status']
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Enter post title'
            }),
            'slug': forms.TextInput(attrs={
                'class': 'form-control',
                'help_text': 'URL-friendly version of title'
            }),
            'category': forms.Select(attrs={'class': 'form-select'}),
            'tags': forms.CheckboxSelectMultiple(),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 10,
                'placeholder': 'Write your post content here...'
            }),
            'excerpt': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 3,
                'placeholder': 'Brief description of the post'
            }),
            'featured_image': forms.FileInput(attrs={'class': 'form-control'}),
            'status': forms.Select(attrs={'class': 'form-select'}),
        }
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['category'].empty_label = "Select Category"
        self.fields['tags'].queryset = Tag.objects.all()
    
    def clean_slug(self):
        slug = self.cleaned_data['slug']
        if Post.objects.filter(slug=slug).exclude(pk=self.instance.pk).exists():
            raise forms.ValidationError("A post with this slug already exists.")
        return slug

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
        widgets = {
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4,
                'placeholder': 'Write your comment here...'
            }),
        }

class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Your Name'
        })
    )
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'your.email@example.com'
        })
    )
    subject = forms.CharField(
        max_length=200,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Subject'
        })
    )
    message = forms.CharField(
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
            'placeholder': 'Your message...'
        })
    )
    
    def send_email(self):
        # Implementation for sending email
        pass

class SignUpForm(UserCreationForm):
    email = forms.EmailField(required=True)
    first_name = forms.CharField(max_length=30, required=True)
    last_name = forms.CharField(max_length=30, required=True)
    
    class Meta:
        model = User
        fields = ('username', 'first_name', 'last_name', 'email', 'password1', 'password2')
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        for field in self.fields.values():
            field.widget.attrs['class'] = 'form-control'
    
    def save(self, commit=True):
        user = super().save(commit=False)
        user.email = self.cleaned_data['email']
        user.first_name = self.cleaned_data['first_name']
        user.last_name = self.cleaned_data['last_name']
        if commit:
            user.save()
        return user
```

### Form Templates

**templates/blog/create_post.html:**
```html
{% extends 'base.html' %}
{% load static %}

{% block title %}Create New Post{% endblock %}

{% block content %}
    <h1>Create New Post</h1>
    
    <form method="post" enctype="multipart/form-data" class="needs-validation" novalidate>
        {% csrf_token %}
        
        <div class="mb-3">
            <label for="{{ form.title.id_for_label }}" class="form-label">Title</label>
            {{ form.title }}
            {% if form.title.errors %}
                <div class="invalid-feedback d-block">{{ form.title.errors.0 }}</div>
            {% endif %}
        </div>
        
        <div class="mb-3">
            <label for="{{ form.slug.id_for_label }}" class="form-label">Slug</label>
            {{ form.slug }}
            <div class="form-text">URL-friendly version of the title</div>
            {% if form.slug.errors %}
                <div class="invalid-feedback d-block">{{ form.slug.errors.0 }}</div>
            {% endif %}
        </div>
        
        <div class="row">
            <div class="col-md-6 mb-3">
                <label for="{{ form.category.id_for_label }}" class="form-label">Category</label>
                {{ form.category }}
                {% if form.category.errors %}
                    <div class="invalid-feedback d-block">{{ form.category.errors.0 }}</div>
                {% endif %}
            </div>
            
            <div class="col-md-6 mb-3">
                <label for="{{ form.status.id_for_label }}" class="form-label">Status</label>
                {{ form.status }}
                {% if form.status.errors %}
                    <div class="invalid-feedback d-block">{{ form.status.errors.0 }}</div>
                {% endif %}
            </div>
        </div>
        
        <div class="mb-3">
            <label class="form-label">Tags</label>
            <div class="row">
                {% for tag in form.tags %}
                    <div class="col-md-3 col-sm-4 col-6 mb-2">
                        <div class="form-check">
                            {{ tag.tag }}
                            <label class="form-check-label" for="{{ tag.id_for_label }}">
                                {{ tag.choice_label }}
                            </label>
                        </div>
                    </div>
                {% endfor %}
            </div>
        </div>
        
        <div class="mb-3">
            <label for="{{ form.featured_image.id_for_label }}" class="form-label">Featured Image</label>
            {{ form.featured_image }}
            {% if form.featured_image.errors %}
                <div class="invalid-feedback d-block">{{ form.featured_image.errors.0 }}</div>
            {% endif %}
        </div>
        
        <div class="mb-3">
            <label for="{{ form.excerpt.id_for_label }}" class="form-label">Excerpt</label>
            {{ form.excerpt }}
            <div class="form-text">Brief description shown in post listings</div>
            {% if form.excerpt.errors %}
                <div class="invalid-feedback d-block">{{ form.excerpt.errors.0 }}</div>
            {% endif %}
        </div>
        
        <div class="mb-3">
            <label for="{{ form.content.id_for_label }}" class="form-label">Content</label>
            {{ form.content }}
            {% if form.content.errors %}
                <div class="invalid-feedback d-block">{{ form.content.errors.0 }}</div>
            {% endif %}
        </div>
        
        <div class="mb-3">
            <button type="submit" class="btn btn-primary">Create Post</button>
            <a href="{% url 'blog:post_list' %}" class="btn btn-secondary">Cancel</a>
        </div>
    </form>
{% endblock %}

{% block extra_js %}
<script>
    // Auto-generate slug from title
    document.getElementById('id_title').addEventListener('input', function() {
        const title = this.value;
        const slug = title.toLowerCase()
            .replace(/[^a-z0-9\s-]/g, '')
            .replace(/\s+/g, '-')
            .replace(/-+/g, '-')
            .trim('-');
        document.getElementById('id_slug').value = slug;
    });
</script>
{% endblock %}
```

## Admin Interface

### Admin Configuration

**blog/admin.py:**
```python
from django.contrib import admin
from django.utils.html import format_html
from .models import Post, Category, Tag, Comment

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'description', 'post_count', 'created_at']
    search_fields = ['name', 'description']
    prepopulated_fields = {'slug': ('name',)} if hasattr(Category, 'slug') else {}
    
    def post_count(self, obj):
        return obj.post_set.count()
    post_count.short_description = 'Posts'

@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display = ['name', 'post_count']
    search_fields = ['name']
    
    def post_count(self, obj):
        return obj.post_set.count()
    post_count.short_description = 'Posts'

class CommentInline(admin.TabularInline):
    model = Comment
    extra = 0
    readonly_fields = ['author', 'created_at']
    fields = ['author', 'content', 'approved', 'created_at']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'category', 'status', 'views', 'created_at', 'featured_image_preview']
    list_filter = ['status', 'created_at', 'category', 'tags']
    search_fields = ['title', 'content', 'author__username']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    filter_horizontal = ['tags']
    inlines = [CommentInline]
    
    fieldsets = (
        ('Post Information', {
            'fields': ('title', 'slug', 'author', 'category', 'tags')
        }),
        ('Content', {
            'fields': ('content', 'excerpt', 'featured_image')
        }),
        ('Publication', {
            'fields': ('status', 'published_at'),
            'classes': ('collapse',)
        }),
        ('Statistics', {
            'fields': ('views',),
            'classes': ('collapse',)
        }),
    )
    
    readonly_fields = ['views']
    
    def featured_image_preview(self, obj):
        if obj.featured_image:
            return format_html(
                '<img src="{}" style="width: 50px; height: 50px; object-fit: cover;" />',
                obj.featured_image.url
            )
        return "No Image"
    featured_image_preview.short_description = 'Preview'
    
    def save_model(self, request, obj, form, change):
        if not change:  # If creating new post
            obj.author = request.user
        super().save_model(request, obj, form, change)
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if request.user.is_superuser:
            return qs
        return qs.filter(author=request.user)

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['post', 'author', 'content_preview', 'approved', 'created_at']
    list_filter = ['approved', 'created_at']
    search_fields = ['content', 'author__username', 'post__title']
    actions = ['approve_comments', 'disapprove_comments']
    
    def content_preview(self, obj):
        return obj.content[:50] + "..." if len(obj.content) > 50 else obj.content
    content_preview.short_description = 'Content'
    
    def approve_comments(self, request, queryset):
        updated = queryset.update(approved=True)
        self.message_user(request, f'{updated} comments approved.')
    approve_comments.short_description = 'Approve selected comments'
    
    def disapprove_comments(self, request, queryset):
        updated = queryset.update(approved=False)
        self.message_user(request, f'{updated} comments disapproved.')
    disapprove_comments.short_description = 'Disapprove selected comments'

# Customize Admin Site
admin.site.site_header = "My Blog Administration"
admin.site.site_title = "My Blog Admin"
admin.site.index_title = "Welcome to My Blog Administration"
```

### Creating Superuser
```bash
python manage.py createsuperuser
```

## Authentication and Authorization

### Built-in Authentication Views

**myproject/urls.py:**
```python
from django.contrib import admin
from django.urls import path, include
from django.contrib.auth import views as auth_views
from . import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
    
    # Authentication URLs
    path('accounts/login/', auth_views.LoginView.as_view(), name='login'),
    path('accounts/logout/', auth_views.LogoutView.as_view(), name='logout'),
    path('accounts/signup/', views.SignUpView.as_view(), name='signup'),
    
    # Password reset URLs
    path('accounts/password_reset/', auth_views.PasswordResetView.as_view(), name='password_reset'),
    path('accounts/password_reset/done/', auth_views.PasswordResetDoneView.as_view(), name='password_reset_done'),
    path('accounts/reset/<uidb64>/<token>/', auth_views.PasswordResetConfirmView.as_view(), name='password_reset_confirm'),
    path('accounts/reset/done/', auth_views.PasswordResetCompleteView.as_view(), name='password_reset_complete'),
]
```

### Custom User Views

**myproject/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth import login
from django.views.generic import CreateView
from django.urls import reverse_lazy
from blog.forms import SignUpForm

class SignUpView(CreateView):
    form_class = SignUpForm
    template_name = 'registration/signup.html'
    success_url = reverse_lazy('login')
    
    def form_valid(self, form):
        response = super().form_valid(form)
        login(self.request, self.object)
        return redirect('blog:post_list')
```

### Authentication Templates

**templates/registration/login.html:**
```html
{% extends 'base.html' %}

{% block title %}Login{% endblock %}

{% block content %}
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card">
                <div class="card-header">
                    <h3 class="text-center">Login</h3>
                </div>
                <div class="card-body">
                    <form method="post">
                        {% csrf_token %}
                        
                        {% if form.errors %}
                            <div class="alert alert-danger">
                                {{ form.errors }}
                            </div>
                        {% endif %}
                        
                        <div class="mb-3">
                            <label for="{{ form.username.id_for_label }}" class="form-label">Username</label>
                            <input type="text" class="form-control" name="username" required>
                        </div>
                        
                        <div class="mb-3">
                            <label for="{{ form.password.id_for_label }}" class="form-label">Password</label>
                            <input type="password" class="form-control" name="password" required>
                        </div>
                        
                        <div class="d-grid">
                            <button type="submit" class="btn btn-primary">Login</button>
                        </div>
                        
                        <div class="text-center mt-3">
                            <p>Don't have an account? <a href="{% url 'signup' %}">Sign up here</a></p>
                            <p><a href="{% url 'password_reset' %}">Forgot your password?</a></p>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
{% endblock %}
```

### User Profile and Permissions

**blog/models.py (additional):**
```python
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(max_length=500, blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)
    website = models.URLField(blank=True)
    twitter = models.CharField(max_length=100, blank=True)
    
    def __str__(self):
        return f"{self.user.username}'s Profile"

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

## Static Files and Media

### Settings Configuration

**settings.py:**
```python
import os

# Static files (CSS, JavaScript, Images)
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'  # For production
STATICFILES_DIRS = [
    BASE_DIR / 'static',  # Project-level static files
]

# Media files (User uploads)
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Static files finders
STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]
```

### Static Files Structure
```
static/
├── css/
│   ├── style.css
│   └── admin.css
├── js/
│   ├── main.js
│   └── form-validation.js
└── images/
    ├── logo.png
    └── default-avatar.png
```

**static/css/style.css:**
```css
/* Custom styles for the blog */
.post-meta {
    color: #6c757d;
    font-size: 0.9em;
}

.post-content {
    line-height: 1.6;
}

.post-content h1, .post-content h2, .post-content h3 {
    margin-top: 2rem;
    margin-bottom: 1rem;
}

.post-content blockquote {
    border-left: 4px solid #007bff;
    padding-left: 1rem;
    margin: 1rem 0;
    background-color: #f8f9fa;
    padding: 1rem;
}

.post-content pre {
    background-color: #f8f9fa;
    padding: 1rem;
    border-radius: 0.25rem;
    overflow-x: auto;
}

.comment-item {
    border-left: 3px solid #007bff;
    padding-left: 1rem;
    margin-bottom: 1rem;
}

.tag-cloud .badge {
    margin: 0.2rem;
    font-size: 0.9em;
}

.social-share {
    margin: 2rem 0;
}

.social-share a {
    margin-right: 1rem;
    text-decoration: none;
}

footer {
    margin-top: 3rem;
    padding: 2rem 0;
    background-color: #f8f9fa;
    border-top: 1px solid #dee2e6;
}
```

## Complete Project: Blog Application

### Project Setup Commands
```bash
# Create project
django-admin startproject myblog
cd myblog

# Create blog app
python manage.py startapp blog

# Install additional packages
pip install Pillow  # For image handling
pip install django-crispy-forms  # For better forms
pip install django-taggit  # For tagging system

# Create directories
mkdir media
mkdir static
mkdir templates
mkdir blog/templates
mkdir blog/templates/blog
mkdir blog/static
mkdir blog/static/blog
```

### Enhanced Settings

**settings.py:**
```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'your-secret-key-here'  # Change in production
DEBUG = True
ALLOWED_HOSTS = []

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third-party apps
    'crispy_forms',
    'crispy_bootstrap5',
    'taggit',
    
    # Local apps
    'blog',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'myblog.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'myblog.wsgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Crispy Forms
CRISPY_ALLOWED_TEMPLATE_PACKS = "bootstrap5"
CRISPY_TEMPLATE_PACK = "bootstrap5"

# Login/Logout redirects
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/'

# Email settings (for password reset)
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'  # Development only
```

### Run the Complete Project
```bash
# Install requirements
pip install -r requirements.txt

# Make migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Collect static files (for production)
python manage.py collectstatic

# Run server
python manage.py runserver
```

### Requirements File

**requirements.txt:**
```
Django==4.2.7
Pillow==10.0.1
django-crispy-forms==2.1
crispy-bootstrap5==0.7
django-taggit==4.0.0
```

## REST API with Django REST Framework

### Installation and Setup
```bash
pip install djangorestframework
pip install django-cors-headers  # For CORS support
```

**settings.py additions:**
```python
INSTALLED_APPS = [
    # ... other apps
    'rest_framework',
    'corsheaders',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # ... other middleware
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20
}

CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",  # React development server
    "http://127.0.0.1:3000",
]
```

### API Serializers

**blog/serializers.py:**
```python
from rest_framework import serializers
from .models import Post, Category, Tag, Comment

class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = ['id', 'name', 'description']

class TagSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tag
        fields = ['id', 'name']

class CommentSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)
    
    class Meta:
        model = Comment
        fields = ['id', 'author', 'content', 'created_at', 'approved']
        read_only_fields = ['author', 'created_at', 'approved']

class PostSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)
    category = CategorySerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    comments = CommentSerializer(many=True, read_only=True)
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'slug', 'author', 'category', 'tags', 
                 'content', 'excerpt', 'featured_image', 'status', 
                 'created_at', 'updated_at', 'published_at', 'views', 'comments']
        read_only_fields = ['author', 'views', 'created_at', 'updated_at']

class PostListSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)
    category = CategorySerializer(read_only=True)
    comment_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'slug', 'author', 'category', 'excerpt', 
                 'featured_image', 'published_at', 'views', 'comment_count']
    
    def get_comment_count(self, obj):
        return obj.comments.filter(approved=True).count()
```

### API Views

**blog/api_views.py:**
```python
from rest_framework import generics, permissions, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from django.shortcuts import get_object_or_404
from .models import Post, Category, Tag, Comment
from .serializers import (PostSerializer, PostListSerializer, 
                         CategorySerializer, TagSerializer, CommentSerializer)

class PostListAPIView(generics.ListCreateAPIView):
    serializer_class = PostListSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_queryset(self):
        queryset = Post.objects.filter(status='published')
        category = self.request.query_params.get('category')
        tag = self.request.query_params.get('tag')
        search = self.request.query_params.get('search')
        
        if category:
            queryset = queryset.filter(category__name__icontains=category)
        if tag:
            queryset = queryset.filter(tags__name__icontains=tag)
        if search:
            queryset = queryset.filter(
                models.Q(title__icontains=search) | 
                models.Q(content__icontains=search)
            )
        
        return queryset.select_related('author', 'category').prefetch_related('tags')
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

class PostDetailAPIView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Post.objects.filter(status='published')
    serializer_class = PostSerializer
    lookup_field = 'slug'
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_object(self):
        obj = super().get_object()
        obj.views += 1
        obj.save(update_fields=['views'])
        return obj

class CategoryListAPIView(generics.ListAPIView):
    queryset = Category.objects.all()
    serializer_class = CategorySerializer

class TagListAPIView(generics.ListAPIView):
    queryset = Tag.objects.all()
    serializer_class = TagSerializer

@api_view(['POST'])
@permission_classes([permissions.IsAuthenticated])
def add_comment(request, slug):
    post = get_object_or_404(Post, slug=slug, status='published')
    serializer = CommentSerializer(data=request.data)
    
    if serializer.is_valid():
        serializer.save(author=request.user, post=post)
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### API URLs

**blog/api_urls.py:**
```python
from django.urls import path
from . import api_views

app_name = 'api'

urlpatterns = [
    path('posts/', api_views.PostListAPIView.as_view(), name='post-list'),
    path('posts/<slug:slug>/', api_views.PostDetailAPIView.as_view(), name='post-detail'),
    path('posts/<slug:slug>/comments/', api_views.add_comment, name='add-comment'),
    path('categories/', api_views.CategoryListAPIView.as_view(), name='category-list'),
    path('tags/', api_views.TagListAPIView.as_view(), name='tag-list'),
]
```

**Main urls.py:**
```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('blog.api_urls')),
    path('', include('blog.urls')),
]
```

## Testing

### Django Testing Framework

**blog/tests.py:**
```python
from django.test import TestCase, Client
from django.contrib.auth.models import User
from django.urls import reverse
from django.core.files.uploadedfile import SimpleUploadedFile
from .models import Post, Category, Tag, Comment

class PostModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.category = Category.objects.create(
            name='Test Category',
            description='Test description'
        )
        self.post = Post.objects.create(
            title='Test Post',
            slug='test-post',
            author=self.user,
            category=self.category,
            content='Test content',
            status='published'
        )
    
    def test_post_creation(self):
        self.assertEqual(self.post.title, 'Test Post')
        self.assertEqual(self.post.slug, 'test-post')
        self.assertEqual(self.post.author, self.user)
        self.assertEqual(self.post.status, 'published')
    
    def test_post_str_method(self):
        self.assertEqual(str(self.post), 'Test Post')
    
    def test_get_absolute_url(self):
        self.assertEqual(self.post.get_absolute_url(), '/post/test-post/')

class PostViewsTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.category = Category.objects.create(name='Test Category')
        self.post = Post.objects.create(
            title='Test Post',
            slug='test-post',
            author=self.user,
            category=self.category,
            content='Test content',
            status='published'
        )
    
    def test_post_list_view(self):
        response = self.client.get(reverse('blog:post_list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Test Post')
        self.assertContains(response, self.post.content)
    
    def test_post_detail_view(self):
        response = self.client.get(
            reverse('blog:post_detail', kwargs={'slug': 'test-post'})
        )
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Test Post')
        self.assertContains(response, 'Test content')
    
    def test_post_detail_view_increment_views(self):
        initial_views = self.post.views
        self.client.get(
            reverse('blog:post_detail', kwargs={'slug': 'test-post'})
        )
        self.post.refresh_from_db()
        self.assertEqual(self.post.views, initial_views + 1)
    
    def test_create_post_requires_login(self):
        response = self.client.get(reverse('blog:create_post'))
        self.assertRedirects(response, '/accounts/login/?next=/create/')
    
    def test_create_post_authenticated(self):
        self.client.login(username='testuser', password='testpass123')
        response = self.client.get(reverse('blog:create_post'))
        self.assertEqual(response.status_code, 200)

class PostFormTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.category = Category.objects.create(name='Test Category')
    
    def test_create_post_form_valid(self):
        self.client.login(username='testuser', password='testpass123')
        data = {
            'title': 'New Test Post',
            'slug': 'new-test-post',
            'category': self.category.id,
            'content': 'New test content',
            'status': 'published'
        }
        response = self.client.post(reverse('blog:create_post'), data)
        self.assertEqual(response.status_code, 302)  # Redirect after successful creation
        self.assertTrue(Post.objects.filter(title='New Test Post').exists())

class APITest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.category = Category.objects.create(name='Test Category')
        self.post = Post.objects.create(
            title='API Test Post',
            slug='api-test-post',
            author=self.user,
            category=self.category,
            content='API test content',
            status='published'
        )
    
    def test_post_list_api(self):
        response = self.client.get('/api/posts/')
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'API Test Post')
    
    def test_post_detail_api(self):
        response = self.client.get(f'/api/posts/{self.post.slug}/')
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'API Test Post')

# Run tests with: python manage.py test
# Run specific test: python manage.py test blog.tests.PostModelTest
# Run with coverage: coverage run --source='.' manage.py test && coverage report
```

### Test Database Configuration

**settings.py (test settings):**
```python
import sys

if 'test' in sys.argv:
    DATABASES['default'] = {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:'
    }
    
    # Disable migrations during testing
    class DisableMigrations:
        def __contains__(self, item):
            return True
        def __getitem__(self, item):
            return None
    
    MIGRATION_MODULES = DisableMigrations()
```

## Deployment

### Preparing for Production

**settings_production.py:**
```python
import os
from .settings import *

# Security Settings
DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Static files
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.StaticFilesStorage'

# Media files
DEFAULT_FILE_STORAGE = 'django.core.files.storage.FileSystemStorage'

# Security
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_SECONDS = 31536000
SECURE_REDIRECT_EXEMPT = []
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# Email
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_USE_TLS = True
EMAIL_PORT = 587
EMAIL_HOST_USER = os.environ.get('EMAIL_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_PASS')

# Cache
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': '/var/log/django/error.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
    },
}
```

### Docker Deployment

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . .

# Collect static files
RUN python manage.py collectstatic --noinput

# Run migrations and start server
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myblog.wsgi:application"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DEBUG=False
      - DB_NAME=blogdb
      - DB_USER=bloguser
      - DB_PASSWORD=blogpass
      - DB_HOST=db
    depends_on:
      - db
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=blogdb
      - POSTGRES_USER=bloguser
      - POSTGRES_PASSWORD=blogpass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

### Heroku Deployment

**Procfile:**
```
release: python manage.py migrate
web: gunicorn myblog.wsgi --log-file -
```

**runtime.txt:**
```
python-3.11.0
```

**Heroku deployment commands:**
```bash
# Install Heroku CLI and login
heroku login

# Create Heroku app
heroku create your-blog-app

# Set environment variables
heroku config:set SECRET_KEY="your-secret-key"
heroku config:set DEBUG=False

# Add PostgreSQL addon
heroku addons:create heroku-postgresql:mini

# Deploy
git add .
git commit -m "Deploy to Heroku"
git push heroku main

# Run migrations
heroku run python manage.py migrate

# Create superuser
heroku run python manage.py createsuperuser
```

### Performance Optimization

**Caching:**
```python
# views.py
from django.views.decorators.cache import cache_page
from django.core.cache import cache

@cache_page(60 * 15)  # Cache for 15 minutes
def post_list(request):
    # View logic here
    pass

# Template caching
{% load cache %}
{% cache 500 sidebar %}
    <!-- sidebar content -->
{% endcache %}

# Low-level caching
def get_popular_posts():
    posts = cache.get('popular_posts')
    if posts is None:
        posts = Post.objects.filter(status='published').order_by('-views')[:5]
        cache.set('popular_posts', posts, 60 * 60)  # Cache for 1 hour
    return posts
```

**Database Optimization:**
```python
# Use select_related for foreign keys
posts = Post.objects.select_related('author', 'category').all()

# Use prefetch_related for many-to-many
posts = Post.objects.prefetch_related('tags').all()

# Database indexes
class Post(models.Model):
    # ... fields ...
    
    class Meta:
        indexes = [
            models.Index(fields=['status', 'created_at']),
            models.Index(fields=['author', 'status']),
        ]
```

## Django Best Practices

### 1. Project Structure
```
myproject/
├── manage.py
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   └── production.txt
├── myproject/
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── blog/
│   ├── users/
│   └── core/
├── static/
├── media/
├── templates/
└── locale/
```

### 2. Security Best Practices
```python
# Use environment variables for sensitive data
import os
from decouple import config

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)

# Always validate user input
from django.core.exceptions import ValidationError

def clean_slug(self):
    slug = self.cleaned_data['slug']
    if not slug.replace('-', '').replace('_', '').isalnum():
        raise ValidationError('Slug can only contain letters, numbers, hyphens, and underscores.')
    return slug

# Use Django's built-in security features
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
```

### 3. Code Organization
```python
# Use class-based views for complex logic
from django.views.generic import ListView
from django.contrib.auth.mixins import LoginRequiredMixin

class PostListView(LoginRequiredMixin, ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        return Post.objects.filter(
            status='published',
            author=self.request.user
        ).select_related('category')

# Use managers for common queries
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')

class Post(models.Model):
    # ... fields ...
    
    objects = models.Manager()  # Default manager
    published = PublishedManager()  # Custom manager
```

### 4. Testing Strategy
```python
# Use factories for test data
import factory
from django.contrib.auth.models import User

class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User
    
    username = factory.Sequence(lambda n: f"user{n}")
    email = factory.LazyAttribute(lambda obj: f"{obj.username}@example.com")
    first_name = factory.Faker('first_name')
    last_name = factory.Faker('last_name')

class PostFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Post
    
    title = factory.Faker('sentence', nb_words=4)
    slug = factory.LazyAttribute(lambda obj: obj.title.lower().replace(' ', '-'))
    author = factory.SubFactory(UserFactory)
    content = factory.Faker('text', max_nb_chars=1000)
    status = 'published'

# Use in tests
class PostTestCase(TestCase):
    def test_post_creation(self):
        post = PostFactory()
        self.assertTrue(isinstance(post, Post))
```

This comprehensive Django tutorial covers everything from basic concepts to building a complete blog application with modern features like REST API, testing, and deployment strategies. Django's "batteries-included" philosophy makes it an excellent choice for rapid development of robust web applications.
