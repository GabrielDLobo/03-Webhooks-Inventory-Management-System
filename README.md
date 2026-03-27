# Webhooks Inventory Management System

This README is now the project documentation entry point.

## Documentation

- Main docs: [docs/index.md](docs/index.md)
- Quick start guide: [docs/getting-started.md](docs/getting-started.md)
- API endpoints: [docs/api-endpoints.md](docs/api-endpoints.md)

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