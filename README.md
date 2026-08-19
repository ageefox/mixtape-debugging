# Mixtape

A social music application where friends can share songs, build collaborative playlists, rate music, and track listening activity.

### CodePath AI201 · Project 5

Developed as part of CodePath AI201 using a provided Flask application. My work focused on navigating an unfamiliar codebase, reproducing reported bugs, tracing request and service-layer logic, implementing fixes, and documenting root causes.

---

## What I Worked On

I investigated and fixed three issues in the existing application.

### Duplicate Songs in Search Results

Songs associated with multiple tags could appear more than once in search results.

**Root cause:** The SQLAlchemy query joined the `song_tags` association table without removing duplicate `Song` rows.

**Fix:** Added `.distinct()` to the search query so each matching song is returned only once.

### Missing Rating Notifications

Song owners were notified when their song was added to a playlist, but not when another user rated it.

**Root cause:** The rating service created or updated the `Rating` record but did not create a notification for the song owner.

**Fix:** Added notification creation after a successful rating while preventing users from receiving notifications when rating their own songs.

### Last Playlist Song Missing

A playlist containing five songs returned only four.

**Root cause:** The playlist service returned `songs[:-1]`, unintentionally dropping the final song after the database query.

**Fix:** Updated the service to return the complete ordered list of playlist songs.

---

## Debugging Approach

For each issue, I:

1. Reproduced the reported behavior in the application.
2. Traced the request from the Flask route into the relevant service.
3. Inspected the database and application logic to identify the root cause.
4. Implemented a targeted fix.
5. Verified the corrected behavior and checked for related side effects.

Detailed reproduction steps and debugging notes are available in [`submission.md`](submission.md).

---

## Tech Stack

- Python
- Flask
- SQLAlchemy
- SQLite
- Pytest

---

## Project Structure

```text
mixtape-debugging/
├── app.py
├── models.py
├── routes/
│   ├── songs.py
│   ├── playlists.py
│   ├── users.py
│   └── feed.py
├── services/
│   ├── streak_service.py
│   ├── feed_service.py
│   ├── search_service.py
│   ├── notification_service.py
│   └── playlist_service.py
├── tests/
├── seed_data.py
├── submission.md
└── requirements.txt
```

---

## Running Locally

Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Seed the database:

```bash
python seed_data.py
```

Run the application:

```bash
FLASK_APP=app:create_app flask run
```

Run the tests for the fixes implemented in this project:

```bash
pytest tests/test_search.py tests/test_playlists.py
```
Note: The full starter test suite also includes a listening-streak test for an issue that was not addressed in this project.

---

## CodePath Project Context

This repository originated from the CodePath AI201 Project 5 starter application. The application architecture and initial codebase were provided; my contribution was the debugging, implementation, verification, and documentation of the fixes described above.
