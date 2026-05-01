---
phase: planning
title: Project Planning & Task Breakdown
description: Break down work into actionable tasks and estimate timeline
---

# Project Planning & Task Breakdown

## Milestones
**What are the major checkpoints?**
- [x] Milestone 1: Server accepts time control payload and initializes room correctly.
- [x] Milestone 2: Client UI allows selecting time control before creating room.
- [ ] Milestone 3: End-to-end testing confirms time control works, AI receives correct payload.

## Task Breakdown
**What specific work needs to be done?**

### Phase 1: Backend Updates
- [x] Task 1.1: Update `server/roomManager.js` -> `createRoom(durationMinutes)`
  - Add input validation (must be 3, 5, 10, 15, or 30).
  - Convert `durationMinutes` to seconds.
  - Set `timeControl.initial`, `whiteTimeLeft`, and `blackTimeLeft`.
- [x] Task 1.2: Update `server/server.js` -> `socket.on('createRoom')`
  - Read `durationMinutes` from the incoming socket payload.
  - Pass it to `createRoom`.

### Phase 2: Frontend Updates
- [x] Task 2.1: Update `client/src/pages/Lobby.js`
  - Add a UI element (e.g., `<select>` or group of buttons) for time control selection (3, 5, 10, 15, 30 mins).
  - Add state `durationMinutes` defaulting to `15`.
  - Update `socket.emit('createRoom', { durationMinutes })`.
- [x] Task 2.2: Verify `GameRoom.js` UI
  - Ensure the clock UI gracefully formats the new times (it should already be dynamic, just verify).

### Phase 3: Integration & Testing
- [ ] Task 3.1: Manual E2E Testing
  - Create a 5-minute room, verify both clocks start at 5:00.
  - Finish the game (e.g., checkmate or resign).
  - Verify server logs show `time_control: "5+0"` in the payload sent to AI Engine.

## Dependencies
**What needs to happen in what order?**
- Backend updates (Phase 1) should happen before Frontend updates (Phase 2), or they can be done concurrently.

## Risks & Mitigation
**What could go wrong?**
- **Risk:** Existing clients or test scripts that don't send `durationMinutes` might break.
  - **Mitigation:** Ensure `durationMinutes` defaults to `15` on the server if it's undefined or invalid.
