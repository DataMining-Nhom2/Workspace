---
phase: requirements
title: Requirements & Problem Understanding
description: Clarify the problem space, gather requirements, and define success criteria
---

# Requirements & Problem Understanding

## Problem Statement
**What problem are we solving?**
- Players are currently forced to play 15-minute games because the time control is hardcoded to 900 seconds in the backend.
- This lack of flexibility makes the web chess application less appealing for players who prefer quick blitz games (e.g., 3m or 5m) or longer games (30m).
- The Room Creation flow currently does not support passing room configuration parameters like time control.

## Goals & Objectives
**What do we want to achieve?**
- **Primary goals**: Allow players to select a time control preset (3m, 5m, 10m, 15m, 30m) when creating a new room.
- **Secondary goals**: Ensure the AI integration still receives the correct `time_control` format (e.g., "5+0") at the end of the game for ELO prediction.
- **Non-goals**: We are explicitly *not* implementing time increments (e.g., +2 seconds per move) in this iteration. We are also *not* implementing custom text input for arbitrary time limits.

## User Stories & Use Cases
**How will users interact with the solution?**
- As a player creating a room, I want to see a dropdown or a set of buttons to choose between 3, 5, 10, 15, and 30 minutes before creating the room.
- As a player joining a room, my clock and my opponent's clock should correctly reflect the time control chosen by the room creator when the game starts.

## Success Criteria
**How will we know when we're done?**
- A time selection UI is present on the Home screen (or wherever rooms are created).
- The `createRoom` socket payload includes the selected time (in minutes or seconds).
- The server's `createRoom` function initializes the room with the correct `whiteTimeLeft`, `blackTimeLeft`, and `timeControl` values.
- The `GameRoom` component correctly displays the chosen time on the clocks when the game starts.
- At the end of the game, the payload sent to the AI endpoint includes the correct `time_control` string (e.g., "10+0").

## Constraints & Assumptions
**What limitations do we need to work within?**
- Only predefined presets are allowed: 3, 5, 10, 15, 30 minutes.
- The increment is always 0.
- The default option should remain 15 minutes to preserve existing behavior if the user doesn't change it.

## Questions & Open Items
**What do we still need to clarify?**
- User has confirmed Approach 1. No open questions remain.
