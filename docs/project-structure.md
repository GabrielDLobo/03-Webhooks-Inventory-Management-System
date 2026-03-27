# Project Structure

## Directory Tree

```
03-Webhooks-Inventory-Management-System/
├── .env                          # Environment variables (not versioned)
├── .gitignore                    # Git ignore rules
├── db.sqlite3                    # SQLite database file
├── manage.py                     # Django management script
├── README.md                     # Project readme
├── requirements.txt              # Python dependencies
├── docs/                         # Documentation folder
│   ├── index.md
│   ├── getting-started.md
│   ├── project-structure.md
│   ├── guidelines.md
│   ├── api-endpoints.md
│   ├── system-modeling.md
│   ├── authentication-security.md
│   ├── development.md
│   ├── deploy.md
│   ├── contributing.md
│   └── release-notes.md
├── app/                          # Django project configuration
│   ├── __init__.py
│   ├── asgi.py                   # ASGI configuration
│   ├── settings.py               # Django settings
│   ├── urls.py                   # Root URL configuration
│   └── wsgi.py                   # WSGI configuration
├── services/                     # External service integrations
│   ├── __init__.py
│   └── callmebot.py              # CallMeBot WhatsApp service
└── webhooks/                     # Main application module
    ├── __init__.py
    ├── admin.py                  # Django admin configuration
    ├── apps.py                   # App configuration
    ├── messages.py               # Notification message templates
    ├── models.py                 # Database models
    ├── templates/                # HTML templates
    │   └── outflow.html          # Email template for outflows
    ├── tests.py                  # Unit tests
    └── views.py                  # Request handlers/views
```

---

## Root Directory Files

### `.env`
Environment configuration file containing sensitive variables:
- CallMeBot API credentials
- Email SMTP settings
- Django secret key and debug settings

**Note:** This file should never be committed to version control.

### `.gitignore`
Specifies intentionally untracked files that Git should ignore:
- Python cache files (`__pycache__/`, `*.pyc`)
- Virtual environment (`venv/`)
- Environment files (`.env`)
- Database files (`db.sqlite3`)
- IDE settings

### `db.sqlite3`
SQLite database file created after running migrations. Contains:
- Webhook event records
- Django admin user data
- Session data

### `manage.py`
Django's command-line utility for administrative tasks:
```bash
python manage.py runserver      # Start development server
python manage.py migrate        # Apply database migrations
python manage.py createsuperuser # Create admin user
python manage.py test           # Run tests
```

### `requirements.txt`
Lists all Python package dependencies:

| Package | Version | Purpose |
|---------|---------|---------|
| Django | 5.2.5 | Web framework |
| djangorestframework | 3.16.1 | REST API framework |
| python-decouple | 3.8 | Environment variable management |
| requests | 2.32.4 | HTTP client for API calls |
| asgiref | 3.9.1 | ASGI support |
| cached-property | 2.0.1 | Property caching |
| certifi | 2025.8.3 | SSL certificates |
| charset-normalizer | 3.4.3 | Charset detection |
| idna | 3.10 | International domain names |
| sqlparse | 0.5.3 | SQL parser |
| standardjson | 0.3.1 | JSON serialization |
| tzdata | 2025.2 | Timezone data |
| urllib3 | 2.5.0 | HTTP library |
| uuid | 1.30 | UUID generation |
| webhooks | 0.4.2 | Webhook utilities |
| wrapt | 1.17.3 | Decorators and wrappers |

### `README.md`
Project overview and quick start guide.

---

## `app/` Directory

Django project configuration package.

### `app/__init__.py`
Empty file marking this directory as a Python package.

### `app/settings.py`
Django settings and configuration:

**Key Configurations:**
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'webhooks',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Environment variables via python-decouple
CALLMEBOT_API_URL = config('CALLMEBOT_API_URL', default='')
CALLMEBOT_PHONE_NUMBER = config('CALLMEBOT_PHONE_NUMBER', default='')
CALLMEBOT_API_KEY = config('CALLMEBOT_API_KEY', default='')
EMAIL_HOST = config('EMAIL_HOST', default='smtp.gmail.com')
EMAIL_PORT = config('EMAIL_PORT', default=587)
EMAIL_HOST_USER = config('EMAIL_HOST_USER', default='')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD', default='')
EMAIL_ADMIN_RECEIVER = config('EMAIL_ADMIN_RECEIVER', default='')
```

### `app/urls.py`
Root URL configuration:

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/V1/webhooks/order/', WebhookOrderView.as_view(), name='webhook-order'),
]
```

**URL Patterns:**
| URL | View | Description |
|-----|------|-------------|
| `/admin/` | Django Admin | Administrative interface |
| `/api/V1/webhooks/order/` | WebhookOrderView | Webhook endpoint for orders |

### `app/wsgi.py`
WSGI configuration for production deployment:
```python
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'app.settings')
application = get_wsgi_application()
```

### `app/asgi.py`
ASGI configuration for async deployment:
```python
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'app.settings')
application = get_asgi_application()
```

---

## `services/` Directory

External service integrations and business logic.

### `services/__init__.py`
Empty file marking this directory as a Python package.

### `services/callmebot.py`
CallMeBot WhatsApp API integration service:

**Class:** `CallMeBot`

**Private Attributes:**
- `__base_url`: API endpoint URL
- `__phone_number`: Destination phone number
- `__api_key`: API authentication key

**Methods:**
| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `send_message()` | `message: str` | `str` | Sends WhatsApp message |

**Usage Example:**
```python
from services.callmebot import CallMeBot

bot = CallMeBot()
bot.send_message("Hello, World!")
```

**⚠️ Known Issues:**
The current implementation has typos that need fixing:
- `respose` should be `response`
- `urf` should be `url`
- `response.text` references undefined variable

---

## `webhooks/` Directory

Main application module containing webhook handling logic.

### `webhooks/__init__.py`
Empty file marking this directory as a Python package.

### `webhooks/models.py`
Database models:

**Model:** `Webhook`

| Field | Type | Attributes | Description |
|-------|------|------------|-------------|
| `id` | UUIDField | primary_key, editable=False | Unique identifier |
| `event_type` | CharField(50) | blank=True, null=True | Type of webhook event |
| `event` | TextField | blank=True, null=True | Full JSON event payload |
| `updated_at` | DateTimeField | auto_now=True | Last update timestamp |
| `created_at` | DateTimeField | auto_now_add=True | Creation timestamp |

**Meta Options:**
```python
class Meta:
    ordering = ['-created_at']  # Newest first
```

### `webhooks/views.py`
Request handlers and business logic:

**Class:** `WebhookOrderView`

**Inherits:** `rest_framework.views.APIView`

**Methods:**
| Method | HTTP | Description |
|--------|------|-------------|
| `post()` | POST | Process incoming webhook |

**Processing Flow:**
1. Persist webhook event to database
2. Calculate total value: `selling_price × quantity`
3. Calculate profit: `total_value - (cost_price × quantity)`
4. Send WhatsApp notification via CallMeBot
5. Send email notification via SMTP
6. Return response with calculated values

### `webhooks/admin.py`
Django admin interface configuration:

```python
@admin.register(Webhook)
class WebhookAdmin(admin.ModelAdmin):
    list_display = ('id', 'event_type', 'created_at', 'updated_at')
    list_filter = ('event_type', 'created_at')
    search_fields = ('id', 'event_type')
    readonly_fields = ('id', 'created_at', 'updated_at')
```

### `webhooks/apps.py`
App configuration:
```python
class WebhooksConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'webhooks'
```

### `webhooks/messages.py`
Notification message templates:

**Variable:** `outflow_message`
```python
outflow_message = '''
*Olá, uma nova saída foi registrada no SGE*

Produto:*{}*
Quantidade:*{}*
Valor da venda: *R$ {}*
Lucro com a venda: *R$ {}*
'''
```

### `webhooks/templates/outflow.html`
HTML email template for outflow notifications:

**Template Variables:**
- `{{ product }}` - Product name
- `{{ quantity }}` - Quantity sold
- `{{ total_value }}` - Total sale value
- `{{ profit_value }}` - Profit from sale

### `webhooks/tests.py`
Unit test module (currently empty scaffold).

---

## File Relationships

```
┌─────────────────────────────────────────────────────────────┐
│                        app/urls.py                          │
│              (Root URL Configuration)                       │
└─────────────────────┬───────────────────────────────────────┘
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
┌─────────────────┐     ┌─────────────────────────┐
│  Django Admin   │     │  WebhookOrderView       │
│  /admin/        │     │  /api/V1/webhooks/order/│
└─────────────────┘     └───────────┬─────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼
           ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
           │   Webhook   │ │  CallMeBot  │ │    Email    │
           │   (Model)   │ │  (Service)  │ │   (SMTP)    │
           └─────────────┘ └─────────────┘ └─────────────┘
```

---

## File Size and Complexity

| File | Purpose | Complexity |
|------|---------|------------|
| `settings.py` | Configuration | Medium |
| `urls.py` | Routing | Low |
| `models.py` | Data layer | Low |
| `views.py` | Business logic | Medium |
| `callmebot.py` | External service | Low |
| `admin.py` | Admin UI | Low |

---

## Next Steps

- Review [Guidelines & Standards](guidelines.md) for coding conventions
- Check [API Endpoints](api-endpoints.md) for integration details
- See [System Modeling](system-modeling.md) for architecture diagrams
