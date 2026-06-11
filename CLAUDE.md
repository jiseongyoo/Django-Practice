# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ Runtime: Miniconda `django-practice` env (read first)

**Every Python / `manage.py` / `pip` / `django-admin` invocation in this repo must run inside the Miniconda env `django-practice`.** A bare `python` from the system PATH will hit the wrong interpreter and is missing Django.

Canonical Python interpreter path:

```
C:\Users\jsing\miniconda3\envs\django-practice\python.exe
```

Pick the invocation pattern that matches the shell Claude Code is using:

| Shell | How to run a command |
|---|---|
| **Bash tool** (`conda` is NOT on PATH) | Use the full interpreter path: `"/c/Users/jsing/miniconda3/envs/django-practice/python.exe" manage.py runserver` |
| **PowerShell tool** (`conda` is NOT on PATH) | Use the call operator with the full path: `& "C:\Users\jsing\miniconda3\envs\django-practice\python.exe" manage.py runserver` |
| Interactive terminal the user runs | `conda activate django-practice` once, then plain `python manage.py ...` |

Verification one-liner (should print `Python 3.12.x` and `Django 6.0.x`):

```powershell
& "C:\Users\jsing\miniconda3\envs\django-practice\python.exe" -c "import sys, django; print(sys.version); print('Django', django.get_version())"
```

### Env management facts

- Dependencies are managed via conda, **not** `requirements.txt` / `pip freeze`. There is no lockfile in the repo.
- The env was created from `conda-forge` only — Anaconda's `main` / `r` / `msys2` channels were declined for ToS reasons. When installing more packages, use `-c conda-forge --override-channels`.
- VS Code's Python extension is pinned to this interpreter via `.vscode/settings.json` (`python.defaultInterpreterPath`), so any terminal VS Code launches will already have the env activated.

## Common commands

Run from the repo root (where `manage.py` lives):

```powershell
python manage.py runserver            # dev server at http://127.0.0.1:8000/
python manage.py migrate              # apply migrations
python manage.py makemigrations       # create migrations after model changes
python manage.py createsuperuser      # admin user for /admin/
python manage.py shell                # Django REPL
python manage.py test                 # run all tests
python manage.py test pages           # run one app's tests
python manage.py test pages.tests.HomeTests.test_home_status  # single test
```

## Architecture

Two-layer Django layout: a project package (`config/`) and one or more apps.

- **`config/`** — the project package. `config/settings.py` holds `INSTALLED_APPS`, DB config (SQLite at `db.sqlite3`), templates config (`APP_DIRS=True`, so per-app `templates/` are auto-discovered). `config/urls.py` is the root URLconf and delegates to each app via `include()`.
- **`pages/`** — sample app. Owns its own `urls.py` with `app_name = 'pages'` (use `{% url 'pages:home' %}` in templates). Templates live under `pages/templates/pages/...` — the nested `pages/` subdirectory is the convention that namespaces templates against other apps.

**Adding a new app** requires three wiring steps that are easy to forget:
1. `python manage.py startapp <name>`
2. Add `'<name>'` to `INSTALLED_APPS` in `config/settings.py`
3. Create `<name>/urls.py` with `app_name = '<name>'` and add `path('<prefix>/', include('<name>.urls'))` to `config/urls.py`
