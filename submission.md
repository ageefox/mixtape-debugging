# Mixtape Bug Hunt Submission

## AI Usage

I used AI to help orient myself in the codebase, understand unfamiliar service functions, and organize my debugging notes. I verified the actual behavior by running the Flask app, checking routes, reproducing bugs manually, and reading the relevant service code myself.

## Codebase Map

### Main files and roles

- `app.py`: Creates the Flask application, initializes the SQLAlchemy database, and registers all route blueprints.

- `models.py`: Defines all SQLAlchemy models used by the application, including users, songs, playlists, notifications, ratings, listening events, and tags. It also defines the many-to-many association tables for friendships, playlist entries, and song tags.

- `routes/songs.py`: Handles endpoints related to songs, including viewing songs, searching, listening to songs, and rating songs.

- `routes/playlists.py`: Handles creating playlists, viewing playlists, and adding songs to playlists.

- `routes/users.py`: Handles user profiles, listening streaks, and notifications.

- `routes/feed.py`: Handles activity feed and "Friends Listening Now" endpoints.

- `services/streak_service.py`: Contains the business logic for updating and retrieving listening streaks.

- `services/feed_service.py`: Contains the business logic for generating the activity feed and currently listening friends.

- `services/search_service.py`: Implements song search functionality.

- `services/notification_service.py`: Creates and retrieves notifications for user actions.

- `services/playlist_service.py`: Retrieves playlist information and playlist songs.

- `seed_data.py`: Populates the database with sample users, songs, playlists, tags, ratings, and listening history for testing.

### Data flow example

Example: User rates a song.

1. A client sends a POST request to `/songs/<song_id>/rate`.
2. The request is handled by `routes/songs.py`.
3. The route calls the appropriate function in `notification_service.py`.
4. The service updates the database by creating a notification for the song owner, then returns the response.

### Patterns I noticed

- Routes are responsible for handling HTTP requests and responses.
- Most business logic is implemented inside the `services` directory.
- SQLAlchemy models represent the database entities and relationships.
- The application uses association tables for many-to-many relationships such as friendships, playlist songs, and song tags.


---
## Issue #4: Missing notification when a friend rates a song

### How I reproduced it

I searched for "Midnight" and found "Midnight Drive", which was shared by nova. I rated that song as darius through the `/songs/<song_id>/rate` endpoint. The rating was created successfully, but nova’s notifications only showed the existing playlist notification and did not include a rating notification.

### How I found the root cause

I followed the call chain from `POST /songs/<song_id>/rate` in `routes/songs.py` to `notification_service.rate_song()`. I compared this with the working playlist notification logic. The playlist path explicitly called `create_notification()`, while the rating path saved the rating and returned without creating any notification.

### The root cause

The rating service updated or created a `Rating` record, but never called `create_notification()` for the song owner. Because of that missing step, ratings were stored correctly but did not produce notifications.

### My fix and side-effect check

I added a `create_notification()` call after the rating was committed, but only when the rater is not the song owner. I retested by rating "Midnight Drive" as darius and confirmed nova received a new `song_rated` notification. I also checked that the existing playlist notification still appeared.

---

## Issue #3: Duplicate songs in search results

### How I reproduced it

I searched for "Crown Heights" and found that "Crown Heights Anthem," a song associated with multiple tags, could appear more than once in the search results. Songs with one or no tags did not show the same behavior.

### How I found the root cause

I traced the search request to `search_service.search_songs()`. The query joins `Song` to the `song_tags` association table so songs can be returned with their tag relationships. A song associated with multiple tags can produce multiple matching rows through that join, and the query did not explicitly eliminate duplicates.

### The root cause

The search query used an outer join on `song_tags` without applying `DISTINCT`. As a result, a song with multiple tag relationships could be represented multiple times in the query results.

### My fix and side-effect check

I added `.distinct()` to the song query so each matching song is returned only once. I verified that the multi-tag song "Crown Heights Anthem" now appears once and checked that searches for songs with one tag, no tags, and no matching results still behave correctly.

---

## Issue #5: Last song missing from playlist

### How I reproduced it

I retrieved a playlist containing five songs and found that only four were returned. The songs that did appear were in the correct position order, but the final song was consistently missing.

### How I found the root cause

I traced the playlist route to `playlist_service.get_playlist_songs()`. The database query correctly joined the playlist entries, filtered by playlist ID, and sorted the songs by their stored position. The problem occurred after the query: the return statement used `songs[:-1]`.

### The root cause

Python slicing with `songs[:-1]` returns every item except the final one. The database query was retrieving the complete playlist correctly, but the service deliberately dropped the last song while converting the results to dictionaries.

### My fix and side-effect check

I changed the return statement to iterate over the full `songs` list instead of `songs[:-1]`. I verified that a five-song playlist now returns all five songs in position order and that an empty playlist still returns an empty list without error.
