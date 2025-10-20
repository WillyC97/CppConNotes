# System Design

## Prompt 1 — Design a Playlist Management System

Prompt Recap

> Design a system to manage playlists — users can create playlists, add or remove tracks, reorder them, and share playlists with others.

### Step 1: Clarify Requirements

Before writing any design, ask clarifying questions — this shows good engineering thinking:

Questions you might ask:
* Are playlists private or can multiple users edit them concurrently?
* How large can a playlist get (number of tracks)?
* Do we need persistence to disk/DB or is this in-memory?
* Are we concerned with scaling to millions of users or just designing the class/module system?
* Should we support offline modifications?

How to say it aloud:

> “To make sure I design the right system, I’d like to clarify a few points. Can multiple users modify a playlist at the same time? Do we need persistent storage, and what scale should I target?”

This shows you think about concurrency, scale, and persistence — all relevant to the interview description.

### Step 2: Start with a Simple Design

Here, sketch the **core classes and relationships**, starting minimal:

Classes:
```c++
class Track {
public:
    std::string id;
    std::string name;
    std::string artist;
    int duration; // seconds
};

class Playlist {
private:
    std::string id;
    std::string name;
    std::vector<std::shared_ptr<Track>> tracks;
public:
    void addTrack(const std::shared_ptr<Track>& track);
    void removeTrack(const std::string& trackId);
    void reorderTrack(size_t oldIndex, size_t newIndex);
};

class User {
public:
    std::string id;
    std::string name;
    std::vector<std::shared_ptr<Playlist>> playlists;
};

class PlaylistManager {
public:
    std::unordered_map<std::string, std::shared_ptr<Playlist>> allPlaylists;

    std::shared_ptr<Playlist> createPlaylist(const std::string& name);
    void deletePlaylist(const std::string& playlistId);
};

```

How to explain aloud:

* “Track represents a song. Playlist contains tracks using shared_ptr so multiple playlists or users can reference the same track without copying it.”

* “User owns playlists; PlaylistManager is a global manager to create/delete playlists and keep track of all playlists.”

* “I’m using std::vector to store tracks since we often iterate in order, and unordered_map for quick lookup of playlists by ID.”

### Step 3: Add Complexity Iteratively (5–7 min)

Once the simple design is clear, add realistic engineering concerns:

1. Concurrency

* If multiple users can edit the same playlist, you need thread safety:
```c++
std::mutex playlistMutex; // inside Playlist
```

* Lock it when adding/removing/reordering tracks.

> “To handle concurrent modifications, I would protect the track vector with a mutex to prevent race conditions.”

2. Sharing Playlists

* If playlists are shared, store references in multiple users:
```c++
std::unordered_map<std::string, std::shared_ptr<Playlist>> sharedPlaylists;
```

* Discuss ownership — shared_ptr is appropriate here.


3. Persistence

* Add save/load methods in PlaylistManager:
```c++
void saveToDatabase();
void loadFromDatabase();
```

* Mention trade-offs: frequent DB writes vs. in-memory caching.

4. Caching / Performance

* Cache recently accessed playlists for fast retrieval: LRUCache<Playlist>.
* Talk about trade-offs: faster reads vs. memory usage.

5. API Layer (Optional)

* If extended to server-side, you’d add endpoints:

* `createPlaylist(userID, playlistName)`
* `addTrack(playlistID, trackID)`
*  `getPlaylist(playlistID)`

> “Even though this is a class design interview, I like to show awareness of how this module could be exposed to clients.”
