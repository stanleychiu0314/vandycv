# VandyCV

An AI-assisted resume builder that turns a student's raw academic and work history into
industry-specific resume copy, then exports it as a formatted PDF.

**[Demo video](https://youtu.be/Ob2rA8JFz4I)**

<!-- Add a screenshot here. `![VandyCV](docs/screenshot.png)` -->

---

## Why

Students writing their first resume face two problems at once: they do not know what
their experience is worth, and they do not know how their target industry phrases
things. Generic templates solve the second problem badly and the first not at all.

VandyCV takes structured input — coursework, roles, activities, target field — and
generates descriptions written in the vocabulary of that field, then lets the user edit
and export.

## How it works

```
Next.js client ──REST──> Flask API ──> PostgreSQL (SQLAlchemy)
                             │
                             └──> OpenAI API ──> generated copy ──> cache
```

The client collects structured input through a questionnaire rather than a free-text
box, which keeps prompts consistent and makes the output reproducible. The Flask API
owns generation, persistence, and the export path. Firebase issues and verifies tokens;
the API trusts the token, not the client.

## Engineering notes

**Prompt design over prompt length.** Generated copy has to sound like the target
industry, not like a language model. The prompts pass structured fields rather than
prose, with the target field as an explicit parameter, so output stays consistent across
users instead of drifting with phrasing.

**Response caching to control latency and token cost.** Generation is the slow and
expensive path. Identical structured inputs return cached output rather than re-calling
the API, which cut both response time and per-user token spend.

**Schema designed for the read pattern.** Users own many resumes and open them
repeatedly, so the schema and indexes are built around retrieving one user's full resume
set quickly rather than around write throughput.

**Token-based auth, verified server-side.** Firebase handles identity. The API verifies
the token on every request rather than trusting a client-supplied user ID.

## Stack

| Layer | |
|---|---|
| Frontend | Next.js, React, Tailwind |
| Backend | Flask, SQLAlchemy |
| Database | PostgreSQL |
| Generation | OpenAI API |
| Auth | Firebase |
| Hosting | Vercel (client), Render (API and database) |

## Running it locally

Requires Node 18+, Python 3.10+, and PostgreSQL.

```bash
git clone https://github.com/stanleychiu0314/vandycv.git
cd vandycv
```

**Backend**

```bash
cd server
python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env              # then fill in the values below
python3 server.py
```

**Frontend**, in a second terminal

```bash
cd client
npm install
npm run dev
```

The client runs on `http://localhost:3000` and expects the API on `http://localhost:5000`.

**PostgreSQL**

```bash
brew install postgresql           # macOS
brew services start postgresql
createdb vandycv
```

Linux and Windows: see the [PostgreSQL install guide](https://www.postgresql.org/download/).

## Environment variables

Create `server/.env`:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `OPENAI_API_KEY` | Generation |
| `FIREBASE_PROJECT_ID` | Token verification |
| `FIREBASE_PRIVATE_KEY` | Token verification |
| `FIREBASE_CLIENT_EMAIL` | Token verification |

<!-- VERIFY these against the actual code before publishing. -->

## Team

Built with [Adam Chen](#) and [Luka Mushkudiani](#) as a three-person project,
August to December 2024.

I owned the backend and data layer: the Flask API, the PostgreSQL schema and indexing,
the OpenAI integration and prompt design, and the deployment setup. Adam and Luka led
the React component work and UI. Firebase auth and API design were done jointly.

## License

MIT. See [LICENSE](LICENSE).
