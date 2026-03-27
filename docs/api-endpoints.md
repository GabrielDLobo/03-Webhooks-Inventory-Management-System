# API Endpoints

## Overview

This section documents all available API endpoints in the Webhooks Inventory Management System.

### Base URL

**Development:** `http://127.0.0.1:8000`  
**Production:** `https://your-domain.com`

### API Version

All endpoints are versioned under `/api/V1/`

---

## Endpoints

### POST /api/V1/webhooks/order/

Receives webhook events from the main inventory management system when a sale/outflow is created.

#### Authentication

**None** - This endpoint is currently public. 

⚠️ **Security Warning:** For production use, implement API key or token authentication.

#### Request Headers

| Header | Value | Required |
|--------|-------|----------|
| `Content-Type` | `application/json` | Yes |

#### Request Body

```json
{
  "event_type": "outflow_created",
  "product": "Product Name",
  "quantity": 10,
  "product_cost_price": 50.00,
  "product_selling_price": 80.00
}
```

#### Request Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `event_type` | string | No | Type of webhook event (e.g., `outflow_created`) |
| `product` | string | Yes | Name of the product |
| `quantity` | integer | Yes | Quantity of products sold |
| `product_cost_price` | number | Yes | Cost price per unit |
| `product_selling_price` | number | Yes | Selling price per unit |

#### Processing Flow

1. **Event Persistence**: Creates a `Webhook` record with event data
2. **Business Calculations**:
   - `total_value = product_selling_price × quantity`
   - `profit_value = total_value - (product_cost_price × quantity)`
3. **WhatsApp Notification**: Sends message via CallMeBot
4. **Email Notification**: Sends HTML email via SMTP

#### Response

**Status Code:** `200 OK`

```json
{
  "event_type": "outflow_created",
  "product": "Product Name",
  "quantity": 10,
  "product_cost_price": 50.00,
  "product_selling_price": 80.00,
  "total_value": 800.00,
  "profit_value": 300.00
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `event_type` | string | Type of event |
| `product` | string | Product name |
| `quantity` | integer | Quantity sold |
| `product_cost_price` | number | Cost price per unit |
| `product_selling_price` | number | Selling price per unit |
| `total_value` | number | **Calculated**: Total sale value |
| `profit_value` | number | **Calculated**: Profit from sale |

#### Error Responses

##### 400 Bad Request - Missing Required Fields

```json
{
  "error": "Missing required field: 'quantity'"
}
```

##### 500 Internal Server Error

```json
{
  "error": "Internal server error"
}
```

#### Example Usage

**cURL:**
```bash
curl -X POST http://127.0.0.1:8000/api/V1/webhooks/order/ \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "outflow_created",
    "product": "Notebook Dell Inspiron",
    "quantity": 2,
    "product_cost_price": 2500.00,
    "product_selling_price": 3200.00
  }'
```

**Python (requests):**
```python
import requests

url = "http://127.0.0.1:8000/api/V1/webhooks/order/"
payload = {
    "event_type": "outflow_created",
    "product": "Notebook Dell Inspiron",
    "quantity": 2,
    "product_cost_price": 2500.00,
    "product_selling_price": 3200.00
}

response = requests.post(url, json=payload)
print(response.json())
```

**JavaScript (fetch):**
```javascript
fetch('http://127.0.0.1:8000/api/V1/webhooks/order/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    event_type: 'outflow_created',
    product: 'Notebook Dell Inspiron',
    quantity: 2,
    product_cost_price: 2500.00,
    product_selling_price: 3200.00
  })
})
.then(response => response.json())
.then(data => console.log(data));
```

---

## Admin Interface

### GET /admin/

Django admin interface for managing webhook events and system configuration.

#### Authentication

**Required:** Yes - Django admin authentication

#### Access

1. Navigate to `http://127.0.0.1:8000/admin/`
2. Login with superuser credentials
3. Access Webhook events under "Webhooks" section

#### Features

- View all webhook events
- Filter by event type and date
- Search by ID or event type
- Export data (with custom admin actions)

---

## HTTP Status Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Request successful |
| 400 | Bad Request | Invalid request (missing fields) |
| 401 | Unauthorized | Authentication required (admin) |
| 403 | Forbidden | Insufficient permissions (admin) |
| 404 | Not Found | Endpoint not found |
| 500 | Internal Server Error | Server error |

---

## Rate Limiting

**Current Status:** Not implemented

⚠️ **Recommendation:** Implement rate limiting for production:

```python
# Example using django-ratelimit
from ratelimit.decorators import ratelimit

@ratelimit(key='ip', rate='100/h')
def post(self, request):
    # ...
```

---

## Webhook Event Types

| Event Type | Description | Trigger |
|------------|-------------|---------|
| `outflow_created` | New outflow/sale created | Main inventory system |
| `outflow_updated` | Outflow updated | Main inventory system |
| `outflow_deleted` | Outflow deleted | Main inventory system |

---

## Notification Templates

### WhatsApp Message (CallMeBot)

```
*Olá, uma nova saída foi registrada no SGE*

Produto:*{product}*
Quantidade:*{quantity}*
Valor da venda: *R$ {total_value}*
Lucro com a venda: *R$ {profit_value}*
```

### Email Template

HTML template located at `webhooks/templates/outflow.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Nova Saída - SGE</title>
</head>
<body>
    <h1>Nova Saída Registrada</h1>
    <p><strong>Produto:</strong> {{ product }}</p>
    <p><strong>Quantidade:</strong> {{ quantity }}</p>
    <p><strong>Valor Total:</strong> R$ {{ total_value }}</p>
    <p><strong>Lucro:</strong> R$ {{ profit_value }}</p>
</body>
</html>
```

---

## Integration Guide

### From Main Inventory System

The main inventory system should send webhooks after creating an outflow:

```python
# Example from main inventory system
import requests

WEBHOOK_URL = "http://your-webhook-service.com/api/V1/webhooks/order/"

def notify_outflow_created(outflow):
    payload = {
        "event_type": "outflow_created",
        "product": outflow.product.name,
        "quantity": outflow.quantity,
        "product_cost_price": float(outflow.product.cost_price),
        "product_selling_price": float(outflow.selling_price)
    }
    
    try:
        response = requests.post(WEBHOOK_URL, json=payload, timeout=5)
        return response.status_code == 200
    except requests.exceptions.RequestException:
        # Log error but don't fail the outflow creation
        logger.error("Failed to send webhook notification")
        return False
```

### Best Practices

1. **Async Processing**: Send webhooks asynchronously to not block main operation
2. **Retry Logic**: Implement retry mechanism for failed webhooks
3. **Timeout**: Set reasonable timeout (5 seconds recommended)
4. **Error Handling**: Log failures but don't fail the main operation
5. **Security**: Use HTTPS in production

---

## Testing Endpoints

### Using Django Test Client

```python
from django.test import TestCase
from rest_framework.test import APIClient

class WebhookEndpointTest(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.url = '/api/V1/webhooks/order/'
    
    def test_webhook_success(self):
        payload = {
            "event_type": "outflow_created",
            "product": "Test Product",
            "quantity": 5,
            "product_cost_price": 100.00,
            "product_selling_price": 150.00
        }
        
        response = self.client.post(self.url, payload, format='json')
        
        self.assertEqual(response.status_code, 200)
        self.assertIn('total_value', response.data)
        self.assertEqual(response.data['total_value'], 750.00)
```

---

## Next Steps

- Review [System Modeling](system-modeling.md) for data flow diagrams
- Check [Authentication & Security](authentication-security.md) for security considerations
- See [Development Guide](development.md) for implementation details
