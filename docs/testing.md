# Testing Guide

## Overview

This document describes the testing strategy and practices for the Webhooks Inventory Management System.

### Current Testing Status

⚠️ **WARNING:** The project currently has **NO TESTS IMPLEMENTED**.

The `webhooks/tests.py` file contains only the default Django test scaffold:

```python
from django.test import TestCase

# Create your tests here.
```

---

## Testing Framework

### Tools

| Tool | Purpose |
|------|---------|
| **unittest** | Python's built-in testing framework |
| **Django Test Framework** | Django-specific testing utilities |
| **DRF Test Framework** | REST framework testing utilities |

### Running Tests

```bash
# Run all tests
python manage.py test

# Run tests for specific app
python manage.py test webhooks

# Run tests with verbosity
python manage.py test -v 2

# Run tests and keep database for debugging
python manage.py test --keepdb

# Run tests with coverage (requires coverage.py)
coverage run manage.py test
coverage report
```

---

## Test Structure

### File Organization

```
webhooks/
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_views.py
│   ├── test_serializers.py
│   ├── test_services.py
│   └── test_integration.py
└── tests.py (legacy, to be removed)
```

### Test Naming Conventions

```python
# Test class naming
class <ModelName>Test(TestCase)
class <ViewName>Test(APITestCase)

# Test method naming
def test_<unit>_<action>_<expected_result>(self)

# Examples
class WebhookModelTest(TestCase):
    def test_webhook_creation_should_create_instance(self):
        pass
    
    def test_webhook_str_should_return_formatted_string(self):
        pass

class WebhookOrderViewTest(APITestCase):
    def test_post_valid_payload_should_return_200(self):
        pass
    
    def test_post_invalid_payload_should_return_400(self):
        pass
```

---

## Model Tests

### Webhook Model Tests

```python
# tests/test_models.py
from django.test import TestCase
from webhooks.models import Webhook
import uuid

class WebhookModelTest(TestCase):
    """Test cases for Webhook model."""
    
    def setUp(self):
        """Set up test data."""
        self.webhook_data = {
            'event_type': 'outflow_created',
            'event': '{"product": "Test", "quantity": 5}'
        }
    
    def test_webhook_creation_should_create_instance(self):
        """Test that webhook is created with correct data."""
        webhook = Webhook.objects.create(**self.webhook_data)
        
        self.assertIsNotNone(webhook.id)
        self.assertIsInstance(webhook.id, uuid.UUID)
        self.assertEqual(webhook.event_type, 'outflow_created')
        self.assertEqual(webhook.event, self.webhook_data['event'])
    
    def test_webhook_auto_timestamps_should_set_created_at(self):
        """Test that created_at is automatically set."""
        webhook = Webhook.objects.create(**self.webhook_data)
        
        self.assertIsNotNone(webhook.created_at)
    
    def test_webhook_auto_timestamps_should_set_updated_at(self):
        """Test that updated_at is automatically set."""
        webhook = Webhook.objects.create(**self.webhook_data)
        
        self.assertIsNotNone(webhook.updated_at)
    
    def test_webhook_str_should_return_formatted_string(self):
        """Test string representation of webhook."""
        webhook = Webhook.objects.create(**self.webhook_data)
        
        expected = f'{webhook.id} - outflow_created'
        self.assertEqual(str(webhook), expected)
    
    def test_webhook_ordering_should_be_newest_first(self):
        """Test that webhooks are ordered by created_at descending."""
        webhook1 = Webhook.objects.create(event_type='event_1')
        webhook2 = Webhook.objects.create(event_type='event_2')
        
        webhooks = list(Webhook.objects.all())
        
        self.assertEqual(webhooks[0], webhook2)
        self.assertEqual(webhooks[1], webhook1)
```

---

## View Tests

### WebhookOrderView Tests

```python
# tests/test_views.py
from rest_framework.test import APITestCase
from rest_framework import status
from webhooks.models import Webhook
import json

class WebhookOrderViewTest(APITestCase):
    """Test cases for WebhookOrderView."""
    
    def setUp(self):
        """Set up test data."""
        self.url = '/api/V1/webhooks/order/'
        self.valid_payload = {
            'event_type': 'outflow_created',
            'product': 'Test Product',
            'quantity': 10,
            'product_cost_price': 50.00,
            'product_selling_price': 80.00
        }
    
    def test_post_valid_payload_should_return_200(self):
        """Test POST with valid payload returns success."""
        response = self.client.post(
            self.url,
            data=self.valid_payload,
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('total_value', response.data)
        self.assertIn('profit_value', response.data)
    
    def test_post_should_calculate_total_value(self):
        """Test that total_value is calculated correctly."""
        response = self.client.post(
            self.url,
            data=self.valid_payload,
            format='json'
        )
        
        expected_total = 80.00 * 10  # selling_price * quantity
        self.assertEqual(response.data['total_value'], expected_total)
    
    def test_post_should_calculate_profit_value(self):
        """Test that profit_value is calculated correctly."""
        response = self.client.post(
            self.url,
            data=self.valid_payload,
            format='json'
        )
        
        expected_profit = (80.00 * 10) - (50.00 * 10)  # total - (cost * quantity)
        self.assertEqual(response.data['profit_value'], expected_profit)
    
    def test_post_should_create_webhook_record(self):
        """Test that webhook record is created in database."""
        initial_count = Webhook.objects.count()
        
        self.client.post(
            self.url,
            data=self.valid_payload,
            format='json'
        )
        
        self.assertEqual(Webhook.objects.count(), initial_count + 1)
    
    def test_post_missing_quantity_should_return_400(self):
        """Test POST without quantity returns bad request."""
        payload = self.valid_payload.copy()
        del payload['quantity']
        
        response = self.client.post(
            self.url,
            data=payload,
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    def test_post_invalid_quantity_should_return_400(self):
        """Test POST with negative quantity returns bad request."""
        payload = self.valid_payload.copy()
        payload['quantity'] = -5
        
        response = self.client.post(
            self.url,
            data=payload,
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
```

---

## Service Tests

### CallMeBot Service Tests

```python
# tests/test_services.py
from django.test import TestCase
from unittest.mock import patch, MagicMock
from services.callmebot import CallMeBot

class CallMeBotServiceTest(TestCase):
    """Test cases for CallMeBot service."""
    
    @patch('services.callmebot.requests.get')
    def test_send_message_should_call_correct_url(self, mock_get):
        """Test that send_message calls correct API URL."""
        mock_response = MagicMock()
        mock_response.text = 'Message sent'
        mock_get.return_value = mock_response
        
        bot = CallMeBot()
        bot.send_message('Test message')
        
        mock_get.assert_called_once()
        call_args = mock_get.call_args
        self.assertIn('phone=', call_args[1]['url'])
        self.assertIn('text=Test+message', call_args[1]['url'])
        self.assertIn('apikey=', call_args[1]['url'])
    
    @patch('services.callmebot.requests.get')
    def test_send_message_should_return_response(self, mock_get):
        """Test that send_message returns API response."""
        mock_response = MagicMock()
        mock_response.text = 'Success'
        mock_get.return_value = mock_response
        
        bot = CallMeBot()
        result = bot.send_message('Test message')
        
        self.assertEqual(result, 'Success')
    
    @patch('services.callmebot.requests.get')
    def test_send_message_should_handle_api_error(self, mock_get):
        """Test that send_message handles API errors gracefully."""
        mock_get.side_effect = Exception('API Error')
        
        bot = CallMeBot()
        
        with self.assertRaises(Exception):
            bot.send_message('Test message')
```

---

## Integration Tests

### End-to-End Webhook Tests

```python
# tests/test_integration.py
from rest_framework.test import APITestCase
from django.core import mail
from unittest.mock import patch
from webhooks.models import Webhook

class WebhookIntegrationTest(APITestCase):
    """Integration tests for webhook flow."""
    
    def setUp(self):
        self.url = '/api/V1/webhooks/order/'
        self.payload = {
            'event_type': 'outflow_created',
            'product': 'Integration Test Product',
            'quantity': 5,
            'product_cost_price': 100.00,
            'product_selling_price': 150.00
        }
    
    @patch('services.callmebot.requests.get')
    def test_full_webhook_flow(self, mock_whatsapp):
        """Test complete webhook processing flow."""
        # Mock WhatsApp API
        mock_whatsapp.return_value.text = 'Sent'
        
        # Send webhook
        response = self.client.post(
            self.url,
            data=self.payload,
            format='json'
        )
        
        # Assert response
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data['total_value'], 750.00)
        self.assertEqual(response.data['profit_value'], 250.00)
        
        # Assert database record
        self.assertEqual(Webhook.objects.count(), 1)
        webhook = Webhook.objects.first()
        self.assertEqual(webhook.event_type, 'outflow_created')
        
        # Assert WhatsApp notification
        mock_whatsapp.assert_called_once()
        
        # Assert email notification (if configured)
        # Note: In test mode, emails go to test outbox
        self.assertEqual(len(mail.outbox), 1)
        self.assertEqual(mail.outbox[0].subject, 'Nova Saída - SGE')
```

---

## Test Fixtures

### Creating Test Data

```python
# tests/fixtures.py
from webhooks.models import Webhook

def create_webhook(**kwargs):
    """Helper function to create webhook for tests."""
    data = {
        'event_type': 'outflow_created',
        'event': '{"test": "data"}',
        **kwargs
    }
    return Webhook.objects.create(**data)

# Usage in tests
class MyTest(TestCase):
    def setUp(self):
        self.webhook = create_webhook(event_type='test_event')
```

### JSON Fixtures

```json
// tests/fixtures/webhook_payloads.json
{
  "valid_outflow": {
    "event_type": "outflow_created",
    "product": "Test Product",
    "quantity": 10,
    "product_cost_price": 50.00,
    "product_selling_price": 80.00
  },
  "invalid_negative_quantity": {
    "event_type": "outflow_created",
    "product": "Test Product",
    "quantity": -5,
    "product_cost_price": 50.00,
    "product_selling_price": 80.00
  },
  "missing_required_field": {
    "event_type": "outflow_created",
    "product": "Test Product"
  }
}
```

---

## Mocking External Services

### Mocking CallMeBot

```python
from unittest.mock import patch, MagicMock

class WebhookTest(APITestCase):
    
    @patch('services.callmebot.requests.get')
    def test_webhook_with_mocked_whatsapp(self, mock_get):
        """Test webhook with mocked WhatsApp service."""
        mock_response = MagicMock()
        mock_response.text = 'Message sent successfully'
        mock_get.return_value = mock_response
        
        response = self.client.post(
            '/api/V1/webhooks/order/',
            data=self.payload,
            format='json'
        )
        
        self.assertEqual(response.status_code, 200)
        mock_get.assert_called_once()
```

### Mocking Email

```python
from django.test import override_settings

@override_settings(EMAIL_BACKEND='django.core.mail.backends.locmem.EmailBackend')
class EmailTest(APITestCase):
    """Test email notifications with in-memory backend."""
    
    def test_email_sent(self):
        from django.core import mail
        
        # Trigger email
        self.client.post('/api/V1/webhooks/order/', data=self.payload)
        
        # Assert email sent
        self.assertEqual(len(mail.outbox), 1)
        self.assertEqual(mail.outbox[0].subject, 'Nova Saída - SGE')
```

---

## Test Coverage

### Installing Coverage Tools

```bash
pip install coverage
```

### Running Coverage

```bash
# Run tests with coverage
coverage run --source='.' manage.py test

# Generate report
coverage report

# Generate HTML report
coverage html
# Open htmlcov/index.html in browser
```

### Coverage Goals

| Component | Target Coverage |
|-----------|----------------|
| Models | 90% |
| Views | 85% |
| Services | 95% |
| Serializers | 90% |
| **Overall** | **85%** |

---

## Continuous Integration

### GitHub Actions Example

```yaml
# .github/workflows/tests.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.10
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
    
    - name: Run tests
      run: |
        python manage.py test
    
    - name: Upload coverage
      uses: codecov/codecov-action@v2
```

---

## Test Best Practices

### DO's

```python
# ✅ Use descriptive test names
def test_webhook_creation_with_valid_data_should_return_200(self):
    pass

# ✅ Use setUp for common test data
def setUp(self):
    self.valid_payload = {...}

# ✅ Test one thing per test method
def test_total_value_calculation(self):
    pass

def test_profit_value_calculation(self):
    pass

# ✅ Use assertions that give clear error messages
self.assertEqual(actual, expected, "Custom error message")
```

### DON'Ts

```python
# ❌ Don't test multiple things in one test
def test_everything(self):
    # Too many assertions
    pass

# ❌ Don't use magic numbers
self.assertEqual(result, 750.00)  # What is 750.00?

# ✅ Better
expected_total = 80.00 * 10  # selling_price * quantity
self.assertEqual(result, expected_total)

# ❌ Don't depend on test execution order
# Each test should be independent
```

---

## Test Implementation Priority

### Phase 1: Critical Path (High Priority)

1. ✅ Webhook model tests
2. ✅ WebhookOrderView tests (valid/invalid payloads)
3. ✅ Business calculation tests
4. ✅ Service integration tests (mocked)

### Phase 2: Extended Coverage (Medium Priority)

1. ⬜ Admin interface tests
2. ⬜ Email notification tests
3. ⬜ WhatsApp notification tests
4. ⬜ Serializer validation tests

### Phase 3: Edge Cases (Low Priority)

1. ⬜ Performance tests
2. ⬜ Load tests
3. ⬜ Security tests
4. ⬜ Integration tests with external services

---

## Next Steps

- Implement Phase 1 tests (Critical Path)
- Set up CI/CD pipeline
- Configure code coverage reporting
- Review [Development Guide](development.md) for development practices
- Check [Deploy Guide](deploy.md) for deployment testing
