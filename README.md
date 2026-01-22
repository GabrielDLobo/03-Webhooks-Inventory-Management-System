# 03-Webhooks-Inventory-Management-System

A webhook integration service designed to enable real-time communication between services of the inventory management system.  This project is a continuation and extension of the main inventory management system, providing webhook functionality for event-driven architecture and external integrations.

This service works in conjunction with the main project:  [02-Inventory-Management-System](https://github.com/GabrielDLobo/02-Inventory-Management-System)

## Main Features

- Real-time webhook event handling for inventory operations
- Integration with external services via HTTP callbacks
- Message delivery system using CallMeBot API
- Event logging and monitoring
- RESTful API endpoints for webhook management
- Asynchronous event processing
- Secure webhook validation and authentication

---

## Table of Contents

- [Technologies](#technologies)
- [Requirements](#requirements)
- [Setup Instructions](#setup-instructions)
- [Running Locally](#running-locally)
- [Development Mode](#development-mode)
- [Integration](#integration)
- [License](#license)

---

## Technologies

This project leverages the following key technologies:

- **Backend**:
  - Django 5.2.5
  - Django Rest Framework 3.16.1
  - Python 3.8+
- **Webhook Management**:
  - webhooks 0.4.2
  - standardjson 0.3.1
  - uuid 1.30
- **External Services**:
  - CallMeBot API integration
  - requests 2.32.4
- **Configuration**:
  - python-decouple 3.8

---

## Requirements

Make sure you have the following installed: 

- Python 3.8+
- pip (Python package manager)
- Access to the main Inventory Management System
- CallMeBot API credentials (optional, for notifications)

---

## Setup Instructions

### Clone the Repository

```bash
git clone https://github.com/GabrielDLobo/03-Webhooks-Inventory-Management-System.git
cd 03-Webhooks-Inventory-Management-System
```

### Install Dependencies

Install Python dependencies:

```bash
pip install -r requirements.txt
```

### Configure Environment Variables

Create a `.env` file in the project root:

```env
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

CALLMEBOT_API_KEY=your-callmebot-api-key
CALLMEBOT_PHONE_NUMBER=your-phone-number

MAIN_INVENTORY_URL=http://localhost:8000
```

### Run Database Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Running Locally

### Without Docker

1. Run database migrations:
   ```bash
   python manage.py migrate
   ```

2. Start the development server:
   ```bash
   python manage.py runserver 8001
   ```

3. Access the webhook service at `http://localhost:8001`.

Note: This service typically runs on a different port than the main inventory system to avoid conflicts.

---

## Development Mode

When developing, you can use the provided `manage.py` file to run jobs, migrations, and other Django commands. 

### Testing Webhooks

You can test webhook endpoints using curl or any HTTP client:

```bash
curl -X POST http://localhost:8001/webhooks/endpoint/ \
  -H "Content-Type: application/json" \
  -d '{"event":  "product.created", "data": {... }}'
```

---

## Project Structure

```
.
|-- app/
|-- webhooks/
|   |-- models.py
|   |-- views.py
|   |-- admin.py
|   |-- messages.py
|   |-- templates/
|   |-- migrations/
|-- services/
|   |-- callmebot.py
```

- `webhooks/` - Main webhook application module
- `services/` - External service integrations (CallMeBot, etc.)
- `models.py` - Webhook event models and data structures
- `views.py` - API endpoints for webhook processing
- `messages.py` - Message templates and formatting
- `callmebot.py` - CallMeBot API integration for notifications

---

## Integration

### Connecting to Main Inventory System

This webhook service is designed to work alongside the main inventory management system. Configure the main system to send webhook events to this service: 

1. Set the webhook URL in the main inventory system settings
2. Configure authentication tokens for secure communication
3. Define which events should trigger webhooks (product updates, stock changes, etc.)

### Supported Webhook Events

- Product creation and updates
- Stock level changes
- Inflow and outflow operations
- Supplier updates
- Category and brand modifications

### External Service Integration

The service currently supports integration with CallMeBot for WhatsApp notifications. Additional integrations can be added in the `services/` directory.

---

## API Endpoints

The service exposes RESTful endpoints for webhook management:

- `POST /webhooks/receive/` - Receive webhook events
- `GET /webhooks/logs/` - View webhook delivery logs
- `POST /webhooks/retry/` - Retry failed webhook deliveries

---

## Security

- Validate webhook signatures to ensure authenticity
- Use HTTPS in production environments
- Store API keys and secrets in environment variables
- Implement rate limiting to prevent abuse
- Log all webhook activities for audit purposes

---

## License

This project is open source, but no license was explicitly defined. 

Feel free to clone, use, and contribute back! 

---

## Contributing

Pull requests are welcome!  For large changes, consider opening an issue first to discuss what you would like to change.

---

## Related Projects

- [02-Inventory-Management-System](https://github.com/GabrielDLobo/02-Inventory-Management-System) - Main inventory management application

---