# Real-time Collaborative Whiteboard Application

A real-time collaborative whiteboard application built with Flask, WebSocket, and Redis. The application supports multi-user collaboration, real-time drawing, and persistent storage.

## Features

- Real-time collaborative drawing
- Multi-user support with room-based collaboration
- WebSocket-based real-time updates
- Redis-backed caching and state management
- Docker support with multi-architecture builds (AMD64 and ARM64)
- Production-ready with Gunicorn and eventlet workers

## Tech Stack

- Backend: Flask, Flask-SocketIO
- Real-time: WebSocket, python-socketio
- Cache: Redis
- Database: SQLAlchemy
- Server: Gunicorn with eventlet workers
- Containerization: Docker with multi-arch support

## Quick Start

### Using Docker

```bash
# Pull the image
docker pull rohitghumare64/whiteboard:latest

# Run the container
docker run -d \
  --name whiteboard \
  -p 5001:5000 \
  --env-file .env \
  rohitghumare64/whiteboard:latest
```

The application will be available at http://localhost:5001

### Local Development

1. Clone the repository:
```bash
git clone https://github.com/rohitg00/my-own-whiteboard.git
cd my-own-whiteboard
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Set up environment variables in `.env`:
```
FLASK_APP=app.py
FLASK_ENV=development
FLASK_DEBUG=1
REDIS_URL=your-redis-url
```

4. Run the application:
```bash
python app.py
```

## Docker Support

The application includes multi-architecture Docker support:

- AMD64 (x86_64) for standard PCs and servers
- ARM64 (AArch64) for Apple Silicon Macs and ARM servers

### Building Docker Image

```bash
# Build multi-arch image
docker buildx build --platform linux/amd64,linux/arm64 -t your-username/whiteboard:latest --push .
```

## Data Flow Architecture

### 1. Client-Side Flow
- **User Interaction** → **Whiteboard Class** → **SocketManager** → **Server**
  - User draws/interacts with canvas (`whiteboard.js` lines 155-260)
  - Events are captured and processed by Whiteboard class
  - SocketManager emits events to server (`socket.js` lines 55-67)

### 2. Server-Side Flow
- **WebSocket Events** → **Event Handlers** → **Cache/Database** → **Broadcast**
  ```
  User Action → Socket Event → Redis Cache → Database → Broadcast to Room
  ```

### 3. Key Components

#### Frontend Components:
- **Whiteboard Class**
  - Manages canvas interactions
  - Handles drawing tools and modes
  - Maintains local history for undo/redo
  - Syncs with other users

- **SocketManager**
  - Manages WebSocket connections
  - Handles room joining/leaving
  - Emits and receives real-time updates
  - Manages user presence

#### Backend Components:
- **Flask-SocketIO Server**
  - Handles real-time events
  - Manages room-based collaboration
  - Coordinates multi-user sessions

- **Redis Cache Layer**
  - Stores real-time cursor positions
  - Caches room state and drawing data
  - Manages user presence
  - Handles temporary data

- **SQLAlchemy Database**
  - Stores persistent drawing data
  - Manages room information
  - Handles long-term storage

### 4. Event Flow Examples

#### Drawing Event Flow:
```
User Draws → Canvas Event → Socket Emit → Server Receives → Redis Cache → Database Store → Broadcast to Room → Other Users Receive → Canvas Update
```

#### Cursor Movement Flow:
```
User Moves Cursor → Cursor Position Event → Socket Emit → Server Receives → Redis Cache → Broadcast to Room → Other Users Receive → Cursor Update
```

#### Room Joining Flow:
```
User Joins Room → Room Join Event → Socket Emit → Server Receives → Redis Cache → Database Store → Broadcast to Room → Other Users Receive → Room Update
```

### 5. Caching Strategy

- **Short-term Cache (Redis)**
  - Cursor positions (2s TTL)
  - Room state (24h TTL)
  - Active user list (5m TTL)
  - Drawing data (1h TTL)

- **Persistent Storage (SQLite)**
  - Room information
  - Drawing history
  - User sessions

### 6. Performance Optimizations

- Throttled cursor updates (50ms)
- Batch rendering for canvas
- Prefetching for frequently accessed rooms
- Connection pooling for Redis
- Optimized WebSocket event handling

## Contributing

Feel free to open issues and pull requests for any improvements.

## License

MIT License

## Author

Rohit Ghumare (ghumare64@gmail.com)
