# Getting Started

## Project Overview

The **Webhooks Inventory Management System** is a Django-based notification service that complements the main inventory management system. It acts as an integration boundary to:

- Receive sales/outflow events via webhook
- Persist events in the database for traceability
- Calculate business values (total value and profit)
- Send notifications via:
  - **WhatsApp** (CallMeBot integration)
  - **Email** (HTML template)

### Key Features

- **Webhook Reception**: Accepts POST requests from the main inventory system
- **Event Persistence**: Stores all webhook events with UUID identification
- **Business Calculations**: Automatically calculates total value and profit
- **Multi-Channel Notifications**: Sends alerts via WhatsApp and Email
- **Admin Interface**: Built-in Django admin for event management

---

## Prerequisites

Before you begin, ensure you have the following installed:

### Required Software

| Software | Version | Description |
|----------|---------|-------------|
| **Python** | 3.10+ | Programming language runtime |
| **pip** | Latest | Python package manager |
| **Git** | Latest | Version control system |

### Recommended Tools

| Tool | Description |
|------|-------------|
| **Virtual Environment** | `venv`, `virtualenv`, or `conda` for isolation |
| **Code Editor** | VS Code, PyCharm, or similar |
| **Database Browser** | DB Browser for SQLite (optional) |

### Knowledge Requirements

- Basic understanding of Python and Django
- Familiarity with REST APIs and webhooks
- Understanding of environment variables

---

## Installation

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd 03-Webhooks-Inventory-Management-System
```

### Step 2: Create Virtual Environment

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux/Mac
python3 -m venv venv
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 4: Configure Environment Variables

```bash
# Copy the example environment file
cp .env.example .env
```

Edit the `.env` file with your configuration (see [Project Configuration](#project-configuration) section).

### Step 5: Run Migrations

```bash
python manage.py migrate
```

### Step 6: Create Superuser (Optional)

```bash
python manage.py createsuperuser
```

### Step 7: Start Development Server

```bash
python manage.py runserver
```

The application will be available at: `http://127.0.0.1:8000/`

---

## Project Configuration

### Environment Variables

Create a `.env` file in the project root with the following variables:

```env
# CallMeBot WhatsApp API Configuration
CALLMEBOT_API_URL=https://api.callmebot.com/whatsapp.php
CALLMEBOT_PHONE_NUMBER=+1234567890
CALLMEBOT_API_KEY=your_api_key_here

# Email Configuration (SMTP)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=your_email@gmail.com
EMAIL_HOST_PASSWORD=your_app_password
EMAIL_ADMIN_RECEIVER=admin@example.com
EMAIL_USE_TLS=True

# Django Settings (optional overrides)
DEBUG=True
SECRET_KEY=your-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1
```

### Environment Variable Descriptions

| Variable | Description | Example |
|----------|-------------|---------|
| `CALLMEBOT_API_URL` | CallMeBot API endpoint | `https://api.callmebot.com/whatsapp.php` |
| `CALLMEBOT_PHONE_NUMBER` | Destination phone number for WhatsApp | `+1234567890` |
| `CALLMEBOT_API_KEY` | API key for CallMeBot authentication | `abc123xyz` |
| `EMAIL_HOST` | SMTP server address | `smtp.gmail.com` |
| `EMAIL_PORT` | SMTP server port | `587` (TLS) or `465` (SSL) |
| `EMAIL_HOST_USER` | Sender email address | `notifications@example.com` |
| `EMAIL_HOST_PASSWORD` | Email account password or app password | `your_password` |
| `EMAIL_ADMIN_RECEIVER` | Admin email to receive notifications | `admin@example.com` |
| `DEBUG` | Django debug mode | `True` or `False` |
| `SECRET_KEY` | Django secret key for cryptographic signing | `random-secret-key` |
| `ALLOWED_HOSTS` | Comma-separated list of allowed hosts | `localhost,example.com` |

### Email Configuration Notes

#### Gmail Setup

If using Gmail as SMTP server:

1. Enable 2-Factor Authentication on your Google account
2. Generate an **App Password**:
   - Go to Google Account → Security → 2-Step Verification → App passwords
   - Select "Mail" and your device
   - Copy the generated 16-character password
3. Use the app password in `EMAIL_HOST_PASSWORD`

#### Port Configuration

| Port | Encryption | Setting |
|------|------------|---------|
| 587 | TLS | `EMAIL_USE_TLS=True` |
| 465 | SSL | `EMAIL_USE_SSL=True` |

### CallMeBot Setup

1. Visit [CallMeBot](https://www.callmebot.com/blog/free-api-for-whatsapp-messages/)
2. Follow the setup instructions to get your API key
3. Send a test message to verify configuration

---

## Verification

### Test the Installation

1. **Check Django Installation**
   ```bash
   python -m django --version
   ```

2. **Verify Database**
   ```bash
   python manage.py dbshell
   # Should open SQLite shell without errors
   ```

3. **Test Webhook Endpoint**
   ```bash
   curl -X POST http://127.0.0.1:8000/api/V1/webhooks/order/ \
     -H "Content-Type: application/json" \
     -d '{
       "event_type": "outflow_created",
       "product": "Test Product",
       "quantity": 5,
       "product_cost_price": 100,
       "product_selling_price": 150
     }'
   ```

4. **Access Admin Interface**
   - Navigate to: `http://127.0.0.1:8000/admin/`
   - Login with superuser credentials

---

## Troubleshooting

### Common Issues

#### 1. ModuleNotFoundError

```bash
# Ensure virtual environment is activated
# Reinstall dependencies
pip install -r requirements.txt
```

#### 2. Database Errors

```bash
# Delete database and recreate migrations
rm db.sqlite3
python manage.py migrate
```

#### 3. Email Not Sending

- Verify `EMAIL_USE_TLS` or `EMAIL_USE_SSL` matches your port
- Check if app password is correct (Gmail requires app password, not regular password)
- Ensure less secure app access is enabled (if not using 2FA)

#### 4. CallMeBot Not Working

- Verify API key is correct
- Check phone number format (include country code, e.g., +1)
- Test API URL in browser first

---

## Next Steps

- Read the [Project Structure](project-structure.md) to understand the codebase
- Review the [API Endpoints](api-endpoints.md) for integration details
- Check [System Modeling](system-modeling.md) for architecture diagrams
