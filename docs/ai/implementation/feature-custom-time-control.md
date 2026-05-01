---
phase: implementation
title: Implementation Guide
description: Technical implementation notes, patterns, and code guidelines
---

# Implementation Guide

## Code Structure
**How is the code organized?**
- `Whess/server/roomManager.js`
- `Whess/server/server.js`
- `Whess/client/src/pages/Home.js`

## Implementation Notes

### Core Features
- **Server default values:** Default to 15 minutes if `durationMinutes` is falsy or not in `[3, 5, 10, 15, 30]`.
- **Validation:** 
  ```javascript
  const validDurations = [3, 5, 10, 15, 30];
  if (!validDurations.includes(durationMinutes)) {
      durationMinutes = 15;
  }
  const initialTime = durationMinutes * 60;
  ```
- **Socket IO Event mapping:**
  Client side `Home.js`:
  ```javascript
  const [duration, setDuration] = useState(15);
  // ...
  socket.emit('createRoom', { durationMinutes: duration }, (response) => { ... });
  ```
  Server side `server.js`:
  ```javascript
  socket.on('createRoom', (data, callback) => {
      const durationMinutes = data?.durationMinutes || 15;
      const { roomId, sessionToken } = roomManager.createRoom(durationMinutes);
      // ...
  });
  ```

### Patterns & Best Practices
- Keep increment logic `0`. Don't try to generalize it prematurely since increment isn't requested.
