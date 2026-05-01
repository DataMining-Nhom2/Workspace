---
phase: testing
title: Testing Strategy
description: Define testing approach, test cases, and quality assurance
---

# Testing Strategy

## Test Coverage Goals
**What level of testing do we aim for?**
- Verify the server gracefully handles invalid or missing `durationMinutes`.
- Verify the client UI allows user selection and passes data correctly.
- Verify the game clock counts down accurately from the selected time.
- Verify AI Engine receives the correct `time_control` string.

## Unit Tests / Manual Verification
**What individual components need testing?**

### `server/roomManager.js`
- [x] Test case 1: Provide `durationMinutes = 5` -> Verify `room.timeControl.initial == 300`.
- [x] Test case 2: Provide `durationMinutes = null` -> Verify fallback to `900` (15 mins).
- [x] Test case 3: Provide `durationMinutes = 999` (invalid) -> Verify fallback to `900` (15 mins).

## Integration Tests
**How do we test component interactions?**
- [x] E2E Scenario 1: Select 10 mins, create room, check `GameRoom` UI to ensure `10:00` is displayed.
- [x] E2E Scenario 2: Play game in 5-min room until game over, ensure `server.js` logs show payload with `"time_control": "5+0"`.
