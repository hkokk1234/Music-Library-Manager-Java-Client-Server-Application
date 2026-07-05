# Music Library Manager — Java Client-Server Application

![Java](https://img.shields.io/badge/Java-8%2B-orange?logo=openjdk&logoColor=white)
![Sockets](https://img.shields.io/badge/Networking-TCP%20Sockets-green)
![Swing](https://img.shields.io/badge/GUI-Swing-blueviolet)

A client-server desktop application for managing a music library — songs and albums — built in Java using **TCP sockets** for communication and **Swing** for the GUI. Multiple clients can connect to a central server to add, update, delete, and search for songs and albums in real time.

## Features

**Songs**
- Add a new song (title, artist, duration)
- Update a song's title
- Delete a song
- Retrieve the full list of songs

**Albums**
- Add a new album (title, description, genre, release year, associated song)
- Update an album's details
- Delete an album
- Search for an album by title

**Client**
- Simple Swing main menu that checks server availability before opening the Album or Music management windows
- Clear error dialog if the server is unreachable

## How It Works

The application uses a lightweight **text-based protocol over TCP sockets**, with the client sending command strings (e.g. `ADD SONG:...`, `SEARCH ALBUM:...`) and the server replying with a plain string (or a list, for `GETSONGS`) via `ObjectOutputStream`/`ObjectInputStream`.

```
 mainmenuclient (Swing)
        │
        ├─ albumclient / clientmusic / searchalbumclient
        │        │
        │        ▼
        │   TCP Socket (port 888)
        │        │
        ▼        ▼
      server.java — holds songs (Set) and albums (Map) in memory
```

**Supported commands:**

| Command | Description |
|---|---|
| `ADD SONG:title:artist:duration` | Adds a new song |
| `UPDATE SONG:oldTitle:newTitle` | Renames an existing song |
| `DELETE SONG:title` | Removes a song |
| `GETSONGS` | Returns the list of all songs |
| `ADD ALBUM:title:description:genre:year:song` | Adds a new album |
| `UPDATE ALBUM:oldTitle:newTitle:description:genre:year:song` | Updates an album |
| `DELETE ALBUM:title` | Removes an album |
| `SEARCH ALBUM:title` | Looks up an album by title |

> **Note:** The server keeps all data in memory (`HashMap`/`HashSet`) — nothing is persisted to disk, so the library resets whenever the server restarts. See [Possible Improvements](#possible-improvements).

## Tech Stack

| Component  | Technology              |
|------------|--------------------------|
| Language   | Java (JDK 8+)            |
| Networking | `java.net` (TCP Sockets) |
| GUI        | Java Swing               |
| Data model | `Album`, `Song` (immutable value classes) |

## Project Structure

```
JavaApplication37/
├── src/javaapplication37/
│   ├── Album.java              # Immutable album model
│   ├── Song.java               # Immutable song model
│   ├── server.java             # TCP socket server, handles all commands (port 888)
│   ├── mainmenuclient.java     # Client entry point / main menu GUI
│   ├── albumclient.java        # Album management window (insert/update/delete/search)
│   ├── clientmusic.java        # Song management window (insert/update/delete)
│   └── searchalbumclient.java  # Album search window
├── build.xml                   # Ant build file
├── nbproject/                  # NetBeans project metadata
└── .vscode/                    # VS Code project settings
```

## Getting Started

### Prerequisites

- JDK 8 or newer

### Compile

```bash
cd src
javac -d ../build/classes javaapplication37/*.java
```

### Run

1. **Start the server:**

   ```bash
   java -cp build/classes javaapplication37.server
   ```

   The server listens on port `888` and also opens the client's main menu window (`mainmenuclient`) automatically on startup.

2. **Start additional clients** (optional, one per machine/user):

   ```bash
   java -cp build/classes javaapplication37.mainmenuclient
   ```

3. From the main menu, click **Album** or **Music** to open the respective management window. If the server isn't reachable, a connection error dialog is shown instead.

## Possible Improvements

- Persist songs/albums to a file or database instead of keeping them only in memory.
- Return structured data (e.g. JSON or serialized objects) instead of colon-delimited strings, to avoid parsing issues if a field itself contains a colon.
- Add input validation with user-facing error messages (e.g. for negative years/durations, currently only logged to the console).
- Separate the server's own GUI launch (`new mainmenuclient()` inside `server.main`) from the server process — typically a server shouldn't spin up client UI.
- Add unit tests for the request-parsing logic in `server.java`.

## Author

- Zacharias Kokkinakis

## License

This project was developed for academic purposes.
