# Ticket Booking System

A distributed ticket booking app in Java — split into a Swing client and two separate servers (one using RMI, one using plain sockets). Users can browse events, book and cancel tickets, and get pushed discount notifications; admins can manage events and users.

## Why three separate pieces?

- **Client** — the Swing GUI. Login, registration, browsing events, booking tickets, admin screens. Talks to Server1 over RMI, and also implements a callback interface so the server can push notifications back to it (e.g. "there's a discount on this show").
- **Server1** — the main RMI server. Handles logins, registrations, bookings, and admin actions. Saves everything to disk using plain Java serialization (`users.ser`, `events.ser`, `bookings.ser`).
- **Server2** — a second, independent socket server meant to handle event data specifically, using its own simple request/response protocol.

```
Client  ◄──RMI (port 9999)──►  Server1  ◄──TCP sockets (port 5000)──►  Server2
```

### A heads-up about Server1 ↔ Server2

Server1 has a `SocketClientHelper` class built specifically to talk to Server2, but as it stands, none of Server1's actual RMI methods (`getEvents`, `addEvent`, etc.) call it — they just use Server1's own local data store. So right now Server2 exists in the code but isn't actually wired into the flow. There's also a port mismatch: `SocketClientHelper` is pointed at port `9999`, but `Server2Main` listens on `5000`. Worth fixing both before relying on that integration.

## What you can do

**As a user:**
- Register / log in
- Browse events and their shows (date, time, price, available seats)
- Book tickets (with a simulated payment)
- View and cancel your own bookings
- Get notified in real time if there's a discount on something

**As an admin:**
- Add new events and shows
- Deactivate a show
- List all users
- Delete a user

## Project structure

```
.
├── Client/
│   └── src/
│       ├── com/example/client/
│       │   ├── communication/   # Client.java, RMI + callback interfaces
│       │   └── model/           # User, Event, Show, Booking, Payment
│       └── project/             # the actual Swing windows
├── Server1/
│   └── src/com/example/server1/
│       ├── Server1Main.java     # starts the RMI registry on port 9999
│       ├── ClientRMIImpl.java   # the RMI service itself
│       ├── DataStore.java       # loads/saves users, events, bookings
│       ├── CallbackManager.java # keeps track of client callbacks
│       └── SocketClientHelper.java  # meant to bridge to Server2
├── Server2/
│   └── src/com/example/server2/
│       ├── Server2Main.java     # socket server on port 5000
│       ├── ClientHandler.java   # handles one client connection per thread
│       ├── LogicManager.java    # GET_EVENTS / ADD_EVENT / DEACTIVATE_SHOW
│       └── DataStore.java       # saves events.ser
└── *.ser                        # data files generated at runtime
```

## Running it

You need a JDK (8+). NetBeans project files are included if you'd rather just open each folder there — otherwise, from each module's folder:

```bash
# Server1 — start this first
cd Server1
javac -d build/classes -cp "../Client/build/classes" $(find src -name "*.java")
java -cp "build/classes:../Client/build/classes" com.example.server1.Server1Main

# Server2 — run alongside Server1
cd Server2
javac -d build/classes $(find src -name "*.java")
java -cp "build/classes:../Client/build/classes" com.example.server2.Server2Main

# Client — run one instance per user
cd Client
javac -d build/classes -cp "build/classes" $(find src -name "*.java")
java -cp "build/classes" project.LoginFrame
```

(On Windows swap `:` for `;` in the classpath, or just use your IDE.)

By default the client looks for the server at `rmi://localhost:9999/BookingService` — change that in the client code if you're running the server elsewhere.

## Data storage

Everything is saved with plain Java serialization to `.ser` files — no real database. Fine for a class project, but these files are generated at runtime and shouldn't be committed to git (see `.gitignore`).

## What I'd improve with more time

- Actually connect `SocketClientHelper` to Server2 and fix the port mismatch
- Swap out `.ser` files for a real database
- Hash passwords instead of storing them as plain text
- Add proper input validation and error handling on both ends
- Write some tests for the booking/cancellation logic, especially around seat availability

## Authors

Stavros Moschis, Zacharias Kokkinakis.
