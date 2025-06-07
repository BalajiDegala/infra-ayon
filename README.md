# Infra Ayon

This repository now contains a simple Django backend and a React frontend skeleton.

## Backend

The `backend` directory was created using `django-admin startproject`. Install dependencies and run the development server:

```bash
pip install -r requirements.txt
cd backend
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

## Frontend

The `frontend` folder contains a very small React application. Install dependencies with npm and start the dev server:

```bash
cd frontend
npm install
npm start
```

## Static site

The original static pages remain in `fly/`.
