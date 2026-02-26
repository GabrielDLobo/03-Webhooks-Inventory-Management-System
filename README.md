# Webhooks Inventory Management System — Notifications Service (Django + DRF)

This project is a small Django service (with Django REST Framework) that exposes a **webhook endpoint** to receive order/sale events and trigger **notifications**.

It is designed to complement the main inventory system by handling integrations and alerts separately.

## What it does

When an order/sale webhook is received, the service:

1. **Persists the raw event** (JSON) to the database for traceability.
2. **Computes business values** such as:
   - `total_value = product_selling_price * quantity`
   - `profit_value = total_value - (product_cost_price * quantity)`
3. Sends notifications:
   - **Message notification** via CallMeBot (typically WhatsApp/Telegram depending on your setup).
   - **Email notification** (HTML template) to the configured admin receiver.

## Tech Stack

- Python / Django
- Django REST Framework
- SQLite (default, dev)
- Email sending via Django `send_mail`
- CallMeBot integration (HTTP request)

## Webhook Endpoint

### `POST /api/V1/webhooks/order/`

**Expected payload fields** (typical):

- `event_type` (string)
- `product` (string)
- `quantity` (number)
- `product_cost_price` (number)
- `product_selling_price` (number)

**Response**

- Returns the received payload plus computed fields:
  - `total_value`
  - `profit_value`

### Example request

```bash
curl -X POST http://127.0.0.1:8000/api/V1/webhooks/order/ \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "outflow_created",
    "product": "Notebook",
    "quantity": 2,
    "product_cost_price": 2500,
    "product_selling_price": 3200
  }'
```

## Configuration

This service typically requires environment variables (or settings) for:

### Email

- `EMAIL_HOST`
- `EMAIL_PORT`
- `EMAIL_HOST_USER`
- `EMAIL_HOST_PASSWORD`
- `EMAIL_USE_TLS` (or SSL)
- `EMAIL_ADMIN_RECEIVER` (recipient)

### CallMeBot

- `CALLMEBOT_API_URL`
- `CALLMEBOT_PHONE_NUMBER`
- `CALLMEBOT_API_KEY`

> Tip: use `python-decouple` or a `.env` file and read the values in `settings.py`.

## Getting Started (development)

### 1) Clone and create a virtual environment

```bash
git clone https://github.com/GabrielDLobo/03-Webhooks-Inventory-Management-System.git
cd 03-Webhooks-Inventory-Management-System

python -m venv .venv
# Linux/Mac:
source .venv/bin/activate
# Windows:
# .venv\Scripts\activate
```

### 2) Install dependencies

If the repository has a `requirements.txt`:

```bash
pip install -r requirements.txt
```

Minimum recommended:

```bash
pip install django djangorestframework python-decouple requests
```

### 3) Run migrations

```bash
python manage.py migrate
```

### 4) Run the server

```bash
python manage.py runserver
```

## Relationship with the core Inventory Management System

- **02-Inventory-Management-System** is the main application where inventory and sales are recorded.
- This repository acts as an **integration boundary** to:
  - receive events from the core system (or any external system),
  - store webhook events,
  - notify users/admins via messaging and email.

## Notes / Known Issues

Review the CallMeBot service implementation to ensure request parameter names and variables are correct (e.g., `url` vs `urf`, `response` naming). If notifications are not being delivered, start debugging there first.

## License

No license is currently defined in this repository.

## Related Projects

- [02-Inventory-Management-System](https://github.com/GabrielDLobo/02-Inventory-Management-System) - Main inventory management application

---