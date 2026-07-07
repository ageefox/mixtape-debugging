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