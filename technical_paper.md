# Django Interview Revision Notes

A compact revision guide for Django interviews, focused on the topics in the review list plus the most important surrounding concepts.

---

## 1. Django Basics

### What is Django?

Django is a Python web framework used to build web applications quickly using reusable components.

It follows the **MVT (Model–View–Template)** architecture:

- **Model** → data and database structure
- **View** → application/business logic
- **Template** → presentation/UI
- **URLconf** → maps URLs to views

Typical request flow:

```text
Browser
   ↓
URL
   ↓
urls.py
   ↓
View
   ↓
Model / ORM
   ↓
Database
   ↓
View
   ↓
Template
   ↓
HTTP Response
```

### Project vs App

**Project**
- The complete Django website/application.
- Contains global configuration such as `settings.py`, `urls.py`, `wsgi.py`, `asgi.py`.

**App**
- A reusable feature/module inside the project.
- Examples: `polls`, `accounts`, `blog`, `orders`.

---

# 2. `settings.py`

`settings.py` contains project-wide configuration.

Important settings:

```python
SECRET_KEY = "..."

DEBUG = True

ALLOWED_HOSTS = []

INSTALLED_APPS = [...]

MIDDLEWARE = [...]

ROOT_URLCONF = "project.urls"

TEMPLATES = [...]

WSGI_APPLICATION = "project.wsgi.application"

DATABASES = {...}

STATIC_URL = "static/"
MEDIA_URL = "media/"
```

## SECRET_KEY

`SECRET_KEY` is a secret value used by Django for cryptographic signing.

It is used for things such as:

- sessions
- password-reset related tokens
- signed cookies
- other cryptographic operations

### Important

Never expose the production secret key publicly.

Do not commit a real production key to GitHub.

---

## `INSTALLED_APPS`

This contains Django applications enabled in the project.

A default Django project commonly contains:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```

### What do they do?

| App | Purpose |
|---|---|
| `admin` | Django admin interface |
| `auth` | Users, groups, permissions, authentication |
| `contenttypes` | Tracks model/content types |
| `sessions` | Session framework |
| `messages` | One-time messages between requests |
| `staticfiles` | Static file management |

There are **many more Django contrib apps and third-party apps**. `INSTALLED_APPS` can also contain your own applications:

```python
INSTALLED_APPS = [
    ...
    "polls",
]
```

---

# 3. Middleware

Middleware is a layer between the incoming HTTP request and the Django view, and also between the view and outgoing response.

Conceptually:

```text
Request
  ↓
Middleware 1
  ↓
Middleware 2
  ↓
Middleware 3
  ↓
View
  ↓
Middleware 3
  ↓
Middleware 2
  ↓
Middleware 1
  ↓
Response
```

Middleware can:

- inspect requests
- modify requests
- inspect responses
- modify responses
- reject requests
- add security protections
- manage sessions/authentication

## Default Middleware

Typical Django middleware includes:

### `SecurityMiddleware`

Provides security-related protections.

Examples include:

- HTTPS-related security settings
- HSTS
- secure redirects
- security headers

---

### `SessionMiddleware`

Enables session support.

Example:

```python
request.session["username"] = "Krishnav"
```

Later:

```python
username = request.session.get("username")
```

---

### `CommonMiddleware`

Provides common conveniences such as URL normalization and some request/response processing.

---

### `CsrfViewMiddleware`

Provides CSRF protection.

---

### `AuthenticationMiddleware`

Adds the authenticated user to the request:

```python
request.user
```

It depends on session middleware.

---

### `MessageMiddleware`

Enables Django's messages framework.

Example:

```python
messages.success(request, "Profile updated!")
```

---

### `XFrameOptionsMiddleware`

Helps protect against **clickjacking** by controlling whether the page can be loaded inside a frame.

---

# 4. Django Security

Django provides several built-in security protections, but developers still need to configure and use Django correctly.

Important areas:

- CSRF
- XSS
- Clickjacking
- SQL injection
- HTTPS
- secure cookies
- authentication
- password storage
- host-header validation

---

## CSRF

**CSRF = Cross-Site Request Forgery**

It occurs when an attacker tricks a user's browser into sending an unwanted request to a website where the user is already authenticated.

Example:

```text
User logs into bank.com
        ↓
Attacker's website tricks browser
        ↓
POST request to bank.com
        ↓
Browser automatically sends authentication cookies
```

Django protects POST forms using a CSRF token.

Template:

```html
<form method="post">
    {% csrf_token %}
    ...
</form>
```

For AJAX requests, the CSRF token also needs to be handled appropriately.

### Interview point

CSRF protection verifies that a state-changing request came from a trusted origin/context rather than simply relying on the user's session cookie.

---

# 5. XSS

**XSS = Cross-Site Scripting**

An attacker injects malicious JavaScript into a page that is viewed by another user.

Example of dangerous input:

```html
<script>alert("Hacked")</script>
```

Django templates automatically escape many values:

```django
{{ username }}
```

This is safer than deliberately marking untrusted content safe.

Avoid blindly using:

```django
{{ value|safe }}
```

### Interview point

CSRF and XSS are different:

- **CSRF** → tricks the browser into making an unwanted request.
- **XSS** → injects executable content into a page.

---

# 6. Clickjacking

Clickjacking tricks a user into clicking something different from what they believe they are clicking.

A malicious site can attempt to embed another website in an invisible or disguised `<iframe>`.

Django's:

```python
XFrameOptionsMiddleware
```

helps protect against this.

Related settings include:

```python
X_FRAME_OPTIONS = "DENY"
```

---

# 7. SQL Injection

SQL injection happens when untrusted input is directly inserted into SQL.

Bad pattern:

```python
query = "SELECT * FROM users WHERE name = '" + name + "'"
```

Django ORM normally parameterizes queries and greatly reduces this risk.

Example:

```python
User.objects.filter(username=name)
```

### Important

Using the ORM does not mean every possible SQL-related mistake disappears. Be careful with raw SQL and unsafe string construction.

---

# 8. Other Important Security Concepts

## HTTPS

Encrypts communication between client and server.

In production, use HTTPS and configure secure settings appropriately.

Important settings include:

```python
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

These should be understood before enabling them because deployment environments can differ.

---

## Secure Cookies

Important cookie settings include:

```python
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
```

`HttpOnly` helps prevent JavaScript from directly reading the cookie.

---

## Passwords

Never store plain-text passwords.

Django's authentication system uses password hashing.

Use:

```python
user.set_password("password")
```

rather than manually storing a password string.

---

# 9. WSGI

**WSGI = Web Server Gateway Interface**

It is a standard interface between Python web applications/frameworks and web servers.

Django's:

```text
wsgi.py
```

provides the WSGI application.

Conceptually:

```text
Web Server
    ↓
WSGI
    ↓
Django
    ↓
View
```

Example:

```python
application = get_wsgi_application()
```

### WSGI vs ASGI

**WSGI**
- Traditional Python web interface
- Primarily synchronous

**ASGI**
- Modern interface
- Supports asynchronous applications
- Useful for WebSockets and async Django features

Django projects commonly contain both:

```text
wsgi.py
asgi.py
```

---

# 10. Models

A Django model is a Python class representing data that Django maps to a database table.

Example:

```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")
```

Conceptually:

```text
Python Model
     ↓
Django ORM
     ↓
Database Table
```

---

# 11. Python Module vs Python Class

This is an important Python concept.

## Module

A Python file is a module.

Example:

```text
models.py
```

You can import it:

```python
import models
```

## Class

A class is a blueprint for creating objects.

Example:

```python
class Question:
    pass
```

So:

```text
models.py → module
Question   → class
```

A module can contain many classes, functions, variables, etc.

---

# 12. Django Model Fields

A field represents a database column.

Common fields:

```python
CharField
TextField
IntegerField
FloatField
DecimalField
BooleanField
DateField
DateTimeField
EmailField
URLField
UUIDField
FileField
ImageField
JSONField
```

Example:

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    available = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

---

# 13. Important Field Options

## `null`

Controls whether the database column can contain SQL `NULL`.

```python
age = models.IntegerField(null=True)
```

## `blank`

Controls whether the field is allowed to be empty during validation/forms.

```python
name = models.CharField(max_length=100, blank=True)
```

### Key difference

```text
null  → database
blank → validation/forms
```

For string fields, `blank=True` is commonly preferred over `null=True`.

---

## `default`

```python
active = models.BooleanField(default=True)
```

Provides a default value.

---

## `unique`

```python
email = models.EmailField(unique=True)
```

Prevents duplicate values at the database/schema level.

---

# 14. Relationships

## ForeignKey

Represents many-to-one relationships.

Example:

```python
class Question(models.Model):
    ...

class Choice(models.Model):
    question = models.ForeignKey(
        Question,
        on_delete=models.CASCADE
    )
```

Meaning:

```text
Question
   ↓
many Choices
```

---

## OneToOneField

One object is related to exactly one object.

Example:

```python
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
```

---

## ManyToManyField

Many objects can be related to many other objects.

Example:

```python
class Student(models.Model):
    courses = models.ManyToManyField(Course)
```

---

# 15. `on_delete=models.CASCADE`

`on_delete` tells Django what should happen to related objects when the referenced object is deleted.

```python
question = models.ForeignKey(
    Question,
    on_delete=models.CASCADE
)
```

`CASCADE` means:

> If the referenced Question is deleted, delete related Choice objects as well.

Example:

```text
Question 1
 ├── Choice A
 ├── Choice B
 └── Choice C

Delete Question 1
        ↓
Choices A/B/C are deleted
```

Other commonly encountered options:

```python
models.CASCADE
models.PROTECT
models.SET_NULL
models.SET_DEFAULT
models.SET(...)
models.RESTRICT
```

### Difference

- `CASCADE` → delete related objects
- `PROTECT` → prevent deletion if related objects exist
- `SET_NULL` → set relation to `NULL`
- `SET_DEFAULT` → use the field's default
- `RESTRICT` → restrict deletion in relevant cases

---

# 16. Validators

Validators are functions that check whether a value satisfies a condition.

Example:

```python
from django.core.validators import MinValueValidator

age = models.IntegerField(
    validators=[MinValueValidator(18)]
)
```

Django includes validators such as:

```text
MinValueValidator
MaxValueValidator
MinLengthValidator
MaxLengthValidator
EmailValidator
URLValidator
RegexValidator
DecimalValidator
```

Example:

```python
from django.core.validators import MinLengthValidator

username = models.CharField(
    max_length=50,
    validators=[MinLengthValidator(3)]
)
```

### Important distinction

A validator performs validation. It is not the same thing as a database constraint.

---

# 17. Django ORM

ORM = **Object-Relational Mapper**

It allows you to interact with a database using Python objects instead of writing SQL for every operation.

Example:

```python
Question.objects.all()
```

instead of:

```sql
SELECT * FROM question;
```

---

# 18. Django Shell

Start the shell:

```bash
python manage.py shell
```

Import your model:

```python
from polls.models import Question
```

Query:

```python
Question.objects.all()
```

Get one object:

```python
Question.objects.get(pk=1)
```

Filter:

```python
Question.objects.filter(question_text__startswith="What")
```

Exclude:

```python
Question.objects.exclude(pub_date__year=2026)
```

Order:

```python
Question.objects.order_by("-pub_date")
```

---

# 19. QuerySet

A QuerySet represents a collection of database queries/results.

Example:

```python
questions = Question.objects.filter(pub_date__year=2026)
```

QuerySets are generally **lazy**.

That means Django does not necessarily execute the database query immediately.

Example:

```python
qs = Question.objects.filter(pub_date__year=2026)
```

The query is evaluated when its results are actually needed.

Common evaluation triggers include:

```python
list(qs)
for question in qs:
    ...
qs[0]
len(qs)
```

---

# 20. CRUD with ORM

## Create

```python
Question.objects.create(
    question_text="Your question?",
    pub_date=timezone.now()
)
```

Or:

```python
question = Question(
    question_text="Your question?",
    pub_date=timezone.now()
)

question.save()
```

## Read

```python
Question.objects.all()
Question.objects.get(pk=1)
Question.objects.filter(...)
```

## Update

```python
question = Question.objects.get(pk=1)
question.question_text = "Updated?"
question.save()
```

Or:

```python
Question.objects.filter(pk=1).update(
    question_text="Updated?"
)
```

## Delete

```python
question.delete()
```

---

# 21. Turning ORM into SQL

In the Django shell:

```python
qs = Question.objects.filter(
    question_text__startswith="What"
)

print(qs.query)
```

This shows the SQL generated by the QuerySet.

For example, conceptually:

```sql
SELECT ...
FROM ...
WHERE question_text LIKE 'What%';
```

### Important

`QuerySet` → Django constructs SQL → database executes SQL.

---

# 22. Aggregation

Aggregation produces a single summarized result from a group of records.

Common aggregate functions:

```text
Count
Sum
Avg
Min
Max
```

Example:

```python
from django.db.models import Count

Question.objects.aggregate(
    total=Count("id")
)
```

Result:

```python
{"total": 10}
```

### Think:

**Aggregation → one overall result**

Example:

> What is the average price of all products?

---

# 23. Annotation

Annotation adds a calculated value to **each object** in a QuerySet.

Example:

```python
from django.db.models import Count

authors = Author.objects.annotate(
    book_count=Count("book")
)
```

Then:

```python
for author in authors:
    print(author.name, author.book_count)
```

### Think:

**Annotation → calculated value attached to every returned object**

---

## Aggregation vs Annotation

```text
aggregate()
    ↓
one summarized result

annotate()
    ↓
calculated value for each row/object
```

Example:

```python
Author.objects.aggregate(total=Count("book"))
```

→ total books across the query.

```python
Author.objects.annotate(book_count=Count("book"))
```

→ each author gets a `book_count`.

---

# 24. `F` and `Q` Objects

These are useful ORM interview topics.

## `F`

Allows you to reference another database field.

Example:

```python
from django.db.models import F

Product.objects.filter(
    stock__lt=F("reorder_level")
)
```

Useful for database-side comparisons and updates.

---

## `Q`

Used for complex conditions such as OR.

```python
from django.db.models import Q

Question.objects.filter(
    Q(question_text__startswith="What") |
    Q(question_text__startswith="Why")
)
```

---

# 25. `select_related` vs `prefetch_related`

Important for performance interviews.

Suppose:

```python
choices = Choice.objects.all()
```

and later each choice accesses:

```python
choice.question
```

This can result in many database queries.

## `select_related`

Used mainly for:

- ForeignKey
- OneToOneField

Uses SQL JOIN.

```python
Choice.objects.select_related("question")
```

## `prefetch_related`

Used mainly for:

- ManyToManyField
- reverse ForeignKey relationships

It performs additional queries and combines the results in Python.

```python
Question.objects.prefetch_related("choice_set")
```

### Easy memory trick

```text
select_related  → JOIN → single-valued relationship
prefetch_related → separate query → collection/multi-valued relationship
```

---

# 26. Migrations

A migration records changes to Django's database schema.

Suppose you change:

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
```

and add:

```python
price = models.DecimalField(
    max_digits=10,
    decimal_places=2
)
```

Run:

```bash
python manage.py makemigrations
```

This creates a migration file.

Then:

```bash
python manage.py migrate
```

applies it to the database.

### Flow

```text
models.py
   ↓
makemigrations
   ↓
migration file
   ↓
migrate
   ↓
database schema changes
```

---

## Why are migration files needed?

They provide a versioned history of database schema changes.

They allow teams to:

- track schema changes in Git
- reproduce database structure
- apply changes consistently
- move changes between environments

Migration files are code describing how to change the schema.

---

# 27. SQL Transactions

A transaction is a group of database operations treated as one unit of work.

The classic properties are **ACID**:

### Atomicity
All operations succeed or none do.

### Consistency
The database moves from one valid state to another valid state.

### Isolation
Concurrent transactions should not incorrectly interfere with each other.

### Durability
Once committed, changes survive failures.

Example:

```text
Transfer ₹500

Account A: -₹500
Account B: +₹500
```

Both operations should happen together.

You should not end up with:

```text
A: -₹500
B: no change
```

---

# 28. Django `transaction.atomic()`

Django provides:

```python
from django.db import transaction
```

Then:

```python
with transaction.atomic():
    account_a.balance -= 500
    account_a.save()

    account_b.balance += 500
    account_b.save()
```

If an exception occurs inside the block, Django rolls back the transaction.

Conceptually:

```text
BEGIN
   ↓
Operation 1
   ↓
Operation 2
   ↓
Error?
 ├── Yes → ROLLBACK
 └── No  → COMMIT
```

### Important

`atomic()` is about **transactional behavior**.

It does not automatically solve every concurrency problem. Depending on the use case, database locking/isolation techniques may also be required.

---

# 29. Views

A view receives a request and returns a response.

Example:

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello")
```

A view can:

- read request data
- query models
- perform business logic
- render templates
- redirect users
- return JSON

---

# 30. Function-Based vs Class-Based Views

## Function-Based View

```python
def index(request):
    return render(request, "polls/index.html")
```

Simple and explicit.

## Class-Based View

```python
from django.views import View

class IndexView(View):
    def get(self, request):
        return render(request, "polls/index.html")
```

Useful when behavior can be reused or organized through inheritance/mixins.

Know both at an interview level.

---

# 31. URLs

URL configuration maps URLs to views.

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

Named URLs allow templates/views to refer to routes without hardcoding URLs.

```python
reverse("index")
```

Template:

```django
{% url "index" %}
```

---

# 32. Templates

Django templates generate HTML.

Example:

```django
<h1>{{ question.question_text }}</h1>
```

Common template features:

```django
{% if user.is_authenticated %}
    ...
{% endif %}
```

```django
{% for question in questions %}
    {{ question.question_text }}
{% endfor %}
```

Templates should primarily handle presentation, not complex business logic.

---

# 33. Forms

Django Forms help with:

- input validation
- HTML form generation
- cleaned data
- error handling

Example:

```python
from django import forms

class ContactForm(forms.Form):
    email = forms.EmailField()
    message = forms.CharField()
```

Important concepts:

```text
is_valid()
cleaned_data
errors
```

---

# 34. Authentication and Authorization

### Authentication

**Who are you?**

Examples:

- login
- logout
- password verification

### Authorization

**What are you allowed to do?**

Examples:

- permissions
- groups
- staff status
- object access rules

Django provides a built-in authentication framework.

Important objects/concepts:

```text
User
Group
Permission
Session
```

---

# 35. Static vs Media Files

## Static files

Files belonging to the application:

```text
CSS
JavaScript
logos
application images
```

Typical setting:

```python
STATIC_URL = "static/"
```

## Media files

User-uploaded content:

```text
profile photos
documents
uploaded images
```

Typical settings:

```python
MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "media"
```

---

# 36. Django Admin

Django provides an admin interface for managing model data.

Register a model:

```python
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

Create an admin user:

```bash
python manage.py createsuperuser
```

---

# 37. Management Commands

`manage.py` provides project utilities.

Important commands:

```bash
python manage.py runserver
python manage.py startapp polls
python manage.py makemigrations
python manage.py migrate
python manage.py shell
python manage.py createsuperuser
python manage.py test
python manage.py check
```

---

# 38. Django Request/Response Cycle

Understand this clearly for interviews.

```text
Client
  ↓
Web Server
  ↓
WSGI / ASGI
  ↓
Django
  ↓
Middleware
  ↓
URL Resolver
  ↓
View
  ↓
ORM
  ↓
Database
  ↓
View
  ↓
Template
  ↓
Response Middleware
  ↓
Client
```

---

# 39. Common Interview Differences

## `get()` vs `filter()`

```python
Question.objects.get(pk=1)
```

Returns one object.

It raises an exception if no object exists or multiple objects match.

```python
Question.objects.filter(pk=1)
```

Returns a QuerySet, potentially containing zero, one, or many objects.

---

## `save()` vs `update()`

```python
obj.save()
```

Saves a model instance.

```python
QuerySet.update(...)
```

Performs a database-level update for matching rows without loading each object into Python.

---

## `null=True` vs `blank=True`

```text
null=True  → database can store NULL
blank=True → validation can accept empty value
```

---

## `CASCADE` vs `PROTECT`

```text
CASCADE → delete dependent objects
PROTECT → prevent deletion
```

---

## Authentication vs Authorization

```text
Authentication → Who are you?
Authorization  → What can you do?
```

---

## WSGI vs ASGI

```text
WSGI → traditional synchronous Python web interface
ASGI → supports asynchronous applications and protocols
```

---

## Aggregation vs Annotation

```text
aggregate() → summary result
annotate()  → calculated value attached to each object
```

---

## `select_related` vs `prefetch_related`

```text
select_related  → SQL JOIN → ForeignKey/OneToOne
prefetch_related → separate query → ManyToMany/reverse relations
```

---

