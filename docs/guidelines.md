# Guidelines & Standards

## Code Style Guidelines

### Python Style Guide

This project follows **PEP 8** - Python Style Guide conventions.

#### General Rules

- **Indentation:** 4 spaces (no tabs)
- **Line Length:** Maximum 79 characters (PEP 8), 100 characters acceptable
- **Blank Lines:** 
  - 2 blank lines between top-level functions/classes
  - 1 blank line between methods in a class
- **Imports:** 
  - Standard library imports first
  - Third-party imports second
  - Local application imports third
  - Each group separated by blank line

#### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Variables | snake_case | `total_value`, `event_type` |
| Functions | snake_case | `send_message()`, `calculate_profit()` |
| Classes | PascalCase | `WebhookOrderView`, `CallMeBot` |
| Constants | UPPER_CASE | `CALLMEBOT_API_URL`, `EMAIL_PORT` |
| Private Attributes | Double underscore prefix | `__base_url`, `__api_key` |
| Files | snake_case | `callmebot.py`, `outflow_message.py` |

#### Code Organization

```python
# Standard library imports
import os
import json
from datetime import datetime

# Third-party imports
from django.db import models
from rest_framework.views import APIView
import requests

# Local application imports
from .models import Webhook
from services.callmebot import CallMeBot
```

---

## Django-Specific Guidelines

### Models

```python
from django.db import models
import uuid

class Webhook(models.Model):
    """Model to store webhook events for traceability."""
    
    id = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,
        editable=False,
        help_text="Unique identifier for the webhook event"
    )
    event_type = models.CharField(
        max_length=50,
        blank=True,
        null=True,
        help_text="Type of the webhook event"
    )
    event = models.TextField(
        blank=True,
        null=True,
        help_text="Full JSON payload of the event"
    )
    updated_at = models.DateTimeField(
        auto_now=True,
        help_text="Last update timestamp"
    )
    created_at = models.DateTimeField(
        auto_now_add=True,
        help_text="Creation timestamp"
    )
    
    class Meta:
        """Meta options for the model."""
        ordering = ['-created_at']
        verbose_name = 'Webhook Event'
        verbose_name_plural = 'Webhook Events'
    
    def __str__(self):
        return f'{self.id} - {self.event_type}'
```

**Model Guidelines:**
- Use docstrings for models and fields
- Define `Meta` class for ordering and verbose names
- Implement `__str__` method for admin display
- Use `help_text` for field documentation
- Keep business logic out of models (use services)

### Views

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class WebhookOrderView(APIView):
    """View to handle incoming order webhook events."""
    
    def post(self, request, *args, **kwargs):
        """
        Process incoming webhook order event.
        
        Args:
            request: HTTP POST request with event data
            
        Returns:
            Response with event data and calculated values
        """
        # Persist webhook event
        webhook = Webhook.objects.create(
            event_type=request.data.get('event_type'),
            event=json.dumps(request.data)
        )
        
        # Calculate business values
        total_value = request.data.get('product_selling_price', 0) * request.data.get('quantity', 0)
        profit_value = total_value - (request.data.get('product_cost_price', 0) * request.data.get('quantity', 0))
        
        # Send notifications
        # ... notification logic
        
        return Response({
            **request.data,
            'total_value': total_value,
            'profit_value': profit_value
        }, status=status.HTTP_200_OK)
```

**View Guidelines:**
- Use class-based views for consistency
- Include docstrings with Args and Returns
- Keep views thin (delegate logic to services)
- Use appropriate HTTP status codes
- Handle exceptions gracefully

### Admin

```python
from django.contrib import admin
from .models import Webhook

@admin.register(Webhook)
class WebhookAdmin(admin.ModelAdmin):
    """Admin configuration for Webhook model."""
    
    list_display = ('id', 'event_type', 'created_at', 'updated_at')
    list_filter = ('event_type', 'created_at')
    search_fields = ('id', 'event_type')
    readonly_fields = ('id', 'created_at', 'updated_at')
    date_hierarchy = 'created_at'
    ordering = ('-created_at',)
```

**Admin Guidelines:**
- Use `@admin.register()` decorator
- Define `list_display` for overview
- Add `list_filter` for filtering
- Include `search_fields` for searchability
- Set `readonly_fields` for immutable fields

---

## Project Patterns

### Service Pattern

Business logic and external service integrations are encapsulated in the `services/` directory:

```python
# services/callmebot.py
from django.conf import settings
import requests

class CallMeBot:
    """Service for sending WhatsApp messages via CallMeBot API."""
    
    def __init__(self):
        self.__base_url = settings.CALLMEBOT_API_URL
        self.__phone_number = settings.CALLMEBOT_PHONE_NUMBER
        self.__api_key = settings.CALLMEBOT_API_KEY
    
    def send_message(self, message: str) -> str:
        """
        Send WhatsApp message.
        
        Args:
            message: Message text to send
            
        Returns:
            API response text
        """
        response = requests.get(
            url=f'{self.__base_url}?phone={self.__phone_number}&text={message}&apikey={self.__api_key}'
        )
        return response.text
```

**Service Pattern Benefits:**
- Separation of concerns
- Easy to test in isolation
- Reusable across the application
- Clear dependency boundaries

### Message Template Pattern

Notification messages are defined in a dedicated `messages.py` file:

```python
# webhooks/messages.py
outflow_message = '''
*Olá, uma nova saída foi registrada no SGE*

Produto:*{}*
Quantidade:*{}*
Valor da venda: *R$ {}*
Lucro com a venda: *R$ {}*
'''
```

**Usage:**
```python
from .messages import outflow_message

message = outflow_message.format(
    product_name,
    quantity,
    total_value,
    profit_value
)
```

---

## Error Handling

### Exception Handling Pattern

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
import logging

logger = logging.getLogger(__name__)

class WebhookOrderView(APIView):
    def post(self, request, *args, **kwargs):
        try:
            # Business logic
            pass
        except KeyError as e:
            logger.error(f'Missing required field: {e}')
            return Response(
                {'error': f'Missing required field: {e}'},
                status=status.HTTP_400_BAD_REQUEST
            )
        except Exception as e:
            logger.error(f'Unexpected error: {e}')
            return Response(
                {'error': 'Internal server error'},
                status=status.HTTP_500_INTERNAL_SERVER_ERROR
            )
```

**Error Handling Guidelines:**
- Log all errors with appropriate detail
- Return meaningful error messages to clients
- Use appropriate HTTP status codes
- Never expose stack traces in production

---

## Documentation Standards

### Docstring Format

Use Google-style docstrings:

```python
def calculate_profit(selling_price: float, cost_price: float, quantity: int) -> float:
    """
    Calculate profit from a sale.
    
    Args:
        selling_price: Price per unit sold
        cost_price: Cost per unit
        quantity: Number of units sold
        
    Returns:
        Total profit from the sale
        
    Raises:
        ValueError: If any parameter is negative
    """
    if selling_price < 0 or cost_price < 0 or quantity < 0:
        raise ValueError("All parameters must be non-negative")
    
    return (selling_price - cost_price) * quantity
```

### Comment Guidelines

```python
# ✅ Good: Explains WHY
# Using UUID for distributed system compatibility
id = models.UUIDField(primary_key=True, default=uuid.uuid4)

# ❌ Bad: Explains WHAT (code is self-explanatory)
# This creates a webhook object
webhook = Webhook.objects.create(...)
```

---

## Git Commit Guidelines

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation changes |
| `style` | Code style changes (formatting) |
| `refactor` | Code refactoring |
| `test` | Adding tests |
| `chore` | Maintenance tasks |

### Examples

```
feat(webhooks): add email notification support

- Implement SMTP email sending
- Add HTML email template
- Configure email settings

Closes #123

---

fix(callmebot): correct variable name typos

- Fix 'respose' to 'response'
- Fix 'urf' to 'url'

---

docs: update API endpoint documentation

- Add request/response examples
- Document error responses
```

---

## Testing Guidelines

### Test Structure

```python
from django.test import TestCase
from rest_framework.test import APITestCase
from .models import Webhook

class WebhookModelTest(TestCase):
    """Test cases for Webhook model."""
    
    def test_webhook_creation(self):
        """Test that webhook is created correctly."""
        webhook = Webhook.objects.create(
            event_type='outflow_created',
            event='{"test": "data"}'
        )
        
        self.assertIsNotNone(webhook.id)
        self.assertEqual(webhook.event_type, 'outflow_created')
        self.assertIsNotNone(webhook.created_at)
```

**Testing Guidelines:**
- Test names should describe the behavior being tested
- Use AAA pattern (Arrange, Act, Assert)
- Test edge cases and error conditions
- Mock external services (CallMeBot, Email)

---

## Security Guidelines

### Environment Variables

```python
# ✅ Good: Use environment variables
from decouple import config
SECRET_KEY = config('SECRET_KEY')

# ❌ Bad: Hardcoded secrets
SECRET_KEY = 'super-secret-key-123'
```

### Sensitive Data

- Never commit `.env` files
- Never log sensitive data (passwords, API keys)
- Use HTTPS in production
- Implement authentication for webhook endpoints

---

## Code Review Checklist

- [ ] Code follows PEP 8 style guide
- [ ] All functions/classes have docstrings
- [ ] Error handling is implemented
- [ ] Tests are included (if applicable)
- [ ] No hardcoded values (use environment variables)
- [ ] No sensitive data in logs
- [ ] Git commit message follows conventions
- [ ] Documentation is updated (if applicable)

---

## Next Steps

- Review [Project Structure](project-structure.md) for file organization
- Check [API Endpoints](api-endpoints.md) for integration patterns
- See [Development Guide](development.md) for workflow details
