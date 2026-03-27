# Webhooks Inventory Management System

This README is now the project documentation entry point.

## Documentation

- Live docs (HTML): https://gabrieldlobo.github.io/03-Webhooks-Inventory-Management-System/
- Main docs: [Home](https://gabrieldlobo.github.io/03-Webhooks-Inventory-Management-System/)
- Quick start guide: [Getting Started](https://gabrieldlobo.github.io/03-Webhooks-Inventory-Management-System/getting-started/)
- API endpoints: [API Endpoints](https://gabrieldlobo.github.io/03-Webhooks-Inventory-Management-System/api-endpoints/)

## Stack and Persistence

- Django + Django REST Framework
- Django ORM + Django migrations
- SQLite by default (development)

This project does not use SQLAlchemy or Alembic.

## Run Application

With the existing virtual environment activated:

```bash
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

## Run Documentation (MkDocs)

With the virtual environment activated:

```bash
pip install -r requirements.txt
mkdocs serve -a 127.0.0.1:8001
```

Open:

- http://127.0.0.1:8000/ (Django application)
- http://127.0.0.1:8001/ (MkDocs)

To generate the static site:

```bash
mkdocs build
```

Generated files are placed in `site/`.

## Publish to GitHub Pages (optional)

```bash
mkdocs gh-deploy
```

If the repository uses the `gh-pages` branch, documentation will be published there automatically.