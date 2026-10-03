# tanshuku (短縮)

A URL shortener built with plain Go + PostgreSQL. Started as a way to learn Go from the ground up, now an evolving personal project I keep extending with new features.

> "Tanshuku" (短縮) means "to abbreviate/shorten" in Japanese.

## Stack

- **Go 1.22+** — using only the standard library for the HTTP server  (`net/http`, with the native method + path-parameter routing syntax)
- **PostgreSQL** — persistence via `database/sql` + the `lib/pq` driver
- **godotenv** — environment variable loading from a `.env` file

## Features

- `POST /shorten` — takes a long URL and returns a short code
  - If the URL has already been shortened before, it returns the existing code instead of creating a new one
  - Code generation includes collision handling (retries up to 100 times to generate a unique code before giving up)
- `GET /{code}` — redirects to the corresponding original URL

## Running locally

### Prerequisites

- Go 1.22 or higher
- PostgreSQL running locally

### 1. Clone and configure

```bash
git clone https://github.com/YOUR_USERNAME/tanshuku.git
cd tanshuku
cp .env.example .env
# edit .env with your actual connection string
```

### 2. Create the database and table

```bash
createdb tanshuku
psql -d tanshuku -c "
CREATE TABLE IF NOT EXISTS links (
    id BIGSERIAL PRIMARY KEY,
    original_url TEXT UNIQUE NOT NULL,
    short_code TEXT UNIQUE NOT NULL,
    author TEXT DEFAULT 'guest',
    created_at TIMESTAMP NOT NULL DEFAULT now()
);
"
```

### 3. Install dependencies and run

```bash
go mod tidy
go run main.go
```

The server starts at `http://localhost:8080`.

## Testing

**Shorten a URL:**
```bash
curl -X POST localhost:8080/shorten -d '{"url":"https://example.com"}'
```

**Visit the short link (redirects):**
```bash
curl -v localhost:8080/RECEIVED_CODE
```

## Roadmap

- [ ] Click analytics (access logging per link, stats endpoint)
- [ ] Private mode (opt-out of click tracking for specific links)
- [ ] Automated tests
- [ ] Deployment (Neon/Supabase for the database + a free hosting service for the app)
