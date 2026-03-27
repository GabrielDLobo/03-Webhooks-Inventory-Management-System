# Webhooks Inventory Management System - Documentation

Welcome to the **Webhooks Inventory Management System** documentation. This guide provides comprehensive information about the project, from local setup to deployment.

## 📚 Documentation Index

### Getting Started

- [Overview, Prerequisites, Installation & Configuration](getting-started.md) - Everything needed to start the project

### Development

- [Project Structure](project-structure.md) - Directory and file organization
- [Guidelines](guidelines.md) - Coding standards and conventions
- [Development](development.md) - Workflow and development practices

### Technical Documentation

- [API Endpoints](api-endpoints.md) - Complete API reference
- [System Modeling](system-modeling.md) - Models, architecture, and flow diagrams
- [Authentication & Security](authentication-security.md) - Security mechanisms and recommendations

### Deployment & Contribution

- [Deploy](deploy.md) - Deployment instructions
- [Contributing](contributing.md) - Contribution process
- [Release Notes](release-notes.md) - Version history and changelog

## 🚀 Quick Start

```bash
# Clone repository
git clone <repository-url>
cd 03-Webhooks-Inventory-Management-System

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Run migrations
python manage.py migrate

# Run server
python manage.py runserver

# Run docs
mkdocs serve -a 127.0.0.1:8001
```

## 📖 What This Service Does

This Django-based webhook service:

- Receives sales/outflow events from the core inventory system
- Persists events for traceability and audit trail
- Calculates business values (`total_value` and `profit_value`)
- Sends notifications via WhatsApp (CallMeBot) and Email

Persistence is handled by Django ORM and Django migrations.
This project does not use SQLAlchemy or Alembic.

## 📞 Support

For issues, questions, or contributions, please use the [Contributing Guide](contributing.md).

Version: 1.0.0  
Last Updated: March 2026
