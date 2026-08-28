# Campus Lost & Found

Campus Lost & Found is a web application for reporting, finding, matching, and securely returning lost items on a university campus.

The project contains:

- A Next.js frontend
- A FastAPI backend
- PostgreSQL with PostGIS and pgvector
- An ML and evaluation workspace

## 1. Prerequisites

The easiest way to run the complete project is with Docker. Install these tools before continuing:

- Git: https://git-scm.com/downloads
- Docker Desktop: https://www.docker.com/products/docker-desktop/

You do not need to install PostgreSQL, Python, or Node.js separately when using Docker Compose.

### Windows requirements

- Windows 10 or Windows 11
- Docker Desktop configured to use the Linux engine
- Internet access for the first image build
- At least 8 GB of available RAM recommended
- At least 10 GB of free disk space recommended

After installing Docker Desktop, open it and wait until the status says **Engine running**.

## 2. Clone the repository

Open **Command Prompt** or **PowerShell** and run:

```cmd
git clone <repository-url>
cd <repository-folder>
```

Replace `<repository-url>` with the URL of this repository. For example:

```cmd
git clone https://github.com/your-account/your-repository.git
cd your-repository
```

The folder containing this README must also contain:

```text
docker-compose.yml
backend\
database\
frontend\
ml\
```

## 3. Start the project

From the repository root, run:

```cmd
docker compose up -d --build
```

The first build can take several minutes because Docker downloads the Python and Node.js dependencies and builds the database image.

The command starts three services:

| Service | URL or port | Purpose |
| --- | --- | --- |
| Frontend | http://localhost:3000 | Next.js web application |
| Backend | http://localhost:8000 | FastAPI API |
| API docs | http://localhost:8000/docs | Swagger API documentation |
| PostgreSQL | localhost:5432 | Application database |

The database image includes both PostGIS and pgvector, which are required by the schema.

## 4. Check that the project is running

Run:

```cmd
docker compose ps
```

You should see these services running:

```text
postgres
backend
frontend
```

The PostgreSQL service should show `healthy`.

You can also test the backend health endpoint:

```cmd
curl http://localhost:8000/health
```

Expected response:

```json
{"status":"healthy"}
```

Open the frontend in a browser:

```text
http://localhost:3000
```

## 5. Use the application

### Create an account

1. Open http://localhost:3000/register.
2. Enter your full name.
3. Enter an email ending with `@college.edu`.
4. Enter a password with at least 8 characters.
5. Confirm the password.
6. Select **Create account**.

The default campus email domain is `college.edu`.

### Sign in

1. Open http://localhost:3000/login.
2. Enter the registered campus email and password.
3. Select **Sign in**.

### Report a lost item

1. Sign in first.
2. Open http://localhost:3000/report/lost.
3. Enter the item title, description, category, campus zone, and date.
4. Optionally choose up to 3 image files or drag them into the upload area.
5. Select **Report Lost Item**.

### Report a found item

1. Sign in first.
2. Open http://localhost:3000/report/found.
3. Enter the item title, description, category, campus zone, and date.
4. Optionally choose up to 3 image files or drag them into the upload area.
5. Select **Report Found Item**.

Marking an item as high value hides its photos from everyone except you until a claim is verified.

### Browse items

Open http://localhost:3000/feed to browse and filter reported items.

## 6. Useful Docker commands

Run the project in the background:

```cmd
docker compose up -d
```

View service status:

```cmd
docker compose ps
```

View all logs:

```cmd
docker compose logs
```

View logs for one service:

```cmd
docker compose logs backend
docker compose logs frontend
docker compose logs postgres
```

Follow logs live:

```cmd
docker compose logs -f backend
```

Stop the services without deleting database data:

```cmd
docker compose stop
```

Start previously stopped services:

```cmd
docker compose start
```

Stop and remove the containers and network:

```cmd
docker compose down
```

Rebuild after changing dependencies or Dockerfiles:

```cmd
docker compose up -d --build
```

## 7. Updating the project

Before updating, stop or leave the project running as needed. From the repository root:

```cmd
git pull
docker compose up -d --build
```

Do not delete the Docker volume unless you intentionally want to delete the local database.

## 8. Troubleshooting

### Docker command is not recognized

Install Docker Desktop, restart Command Prompt or PowerShell, and verify:

```cmd
docker --version
docker compose version
```

### Docker engine is not running

Open Docker Desktop and wait for **Engine running**, then retry:

```cmd
docker compose up -d --build
```

### Frontend does not open on port 3000

Check the frontend logs:

```cmd
docker compose logs frontend
```

If another application uses port 3000, stop that application or change the frontend port mapping in `docker-compose.yml`.

### Backend does not start

Check the backend logs:

```cmd
docker compose logs backend
```

The backend waits for PostgreSQL to become healthy before starting.

### Database initialization fails

Check the PostgreSQL logs:

```cmd
docker compose logs postgres
```

If this is a new local installation and you do not need existing local data, recreate the database volume:

```cmd
docker compose down

docker volume rm lost_found_postgres_data

docker compose up -d --build
```

Warning: removing the volume permanently deletes the local PostgreSQL data.

### Registration or reporting returns 401

Sign in again at:

```text
http://localhost:3000/login
```

Then return to the report page. The application requires a valid login token for protected actions.

### Registration returns a campus email error

Use an email ending in:

```text
@college.edu
```

The allowed domain is configured by `CAMPUS_EMAIL_DOMAIN` in `docker-compose.yml`.

## 9. Optional manual development setup

Docker Compose is the recommended setup. For frontend-only work, you can also run the Next.js development server locally:

```cmd
cd frontend
npm install
npm run dev
```

Then open http://localhost:3000.

The backend and database must still be running for login, registration, feeds, and reports to work. The frontend calls the backend directly at the URL in `NEXT_PUBLIC_API_URL` (`http://localhost:8000/api` by default), so that origin must be listed in the backend's `ALLOWED_ORIGINS`. Serving the app from a different host, such as a LAN address, requires adding that origin to `ALLOWED_ORIGINS` as well.

## 10. Project structure

```text
backend/       FastAPI application, models, schemas, and services
database/      PostgreSQL schema and pgvector database Dockerfile
frontend/      Next.js application and frontend API client
ml/            Embeddings, retrieval, ranking, and evaluation code
docker-compose.yml
```

## 11. Testing and checks

Frontend type check:

```cmd
cd frontend
npm run type-check
```

Frontend production build:

```cmd
cd frontend
npm run build
```

Frontend lint:

```cmd
cd frontend
npm run lint
```

Backend tests. These run against a temporary SQLite database, so no Postgres or Docker is needed:

```cmd
cd backend
pip install -r requirements.txt
pytest tests/
```

ML evaluation:

```cmd
python ml/src/evaluation/run_eval.py
```

## 12. Stopping and restarting the project

To stop the project:

```cmd
docker compose down
```

To start it again later:

```cmd
cd /d "path\to\your\repository"
docker compose up -d
```

Then open:

```text
http://localhost:3000
```
