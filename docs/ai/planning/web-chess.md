---
phase: planning
title: Project Planning & Task Breakdown — ChessWeb PoC
description: Kế hoạch triển khai chi tiết cho ChessWeb PoC. Phase-based task breakdown với dependencies, estimates, milestones, risks. Tech: React + Vite + Socket.IO + Node.js, server-side clock 15+0, disconnect/reconnect, room lifecycle, AI integration.
---

# Planning: ChessWeb PoC — Chess Multiplayer với AI ELO Prediction

## Milestones

- [ ] **M1: Foundation** — Scaffold repo, setup server + frontend, session token infrastructure
- [ ] **M2: Multiplayer Core** — Socket events đúng spec, board auto-flip/promote, 2-player sync
- [ ] **M3: Game Logic** — Clock 15+0 pause/resume, PGN collection, time tracking, game over detection
- [ ] **M4: Disconnect/Reconnect** — session token, clock pause/resume, exit room, cleanup timers
- [ ] **M5: AI Integration** — AI client, Result Modal, fallback
- [ ] **M6: Polish & QA** — Play Again, check indicator, debug logging, nghiệm thu checklist

---

## Task Breakdown

### Phase 1: Foundation — Cơ sở hạ tầng

> **Mục tiêu:** Clean codebase, setup routing, session token, backend structure

#### 1.1: Scaffold ChessWeb Repository

- [x] Khởi tạo Vite + React project
- [x] Cài dependencies: react-chessboard, chess.js, socket.io-client, nanoid, react-router-dom, concurrently
- [x] Setup ESLint config (ESM compatible)
- [x] Tạo file structure: `server/`, `src/pages/`, `src/components/`

#### 1.2: Backend — Server Files

- [x] `server/index.js` — Express + Socket.IO (port 3000)
- [x] `server/roomManager.js` — Room creation, join, session token (ESM, `export const rooms = new Map()`)
- [x] `server/gameLogic.js` — Skeleton (validate move, build PGN, detect game over)
- [x] `server/clockManager.js` — Skeleton (pause/resume, time tracking)
- [x] `server/aiClient.js` — Skeleton (mock)
- [x] Setup `npm run start` với concurrently (dev + server)

#### 1.3: Frontend — Core Files

- [x] `src/socket.js` — Socket.IO client singleton
- [x] `src/main.jsx` — React entry point
- [x] `src/App.jsx` — React Router (`/` → Lobby, `/room/:roomId` → GameRoom)
- [x] `src/index.css` — Dark theme CSS variables
- [x] `src/pages/Lobby.jsx` — Tạo/nhập phòng
- [x] `src/pages/GameRoom.jsx` — Phòng đấu (placeholder)
- [x] `src/components/Clock.jsx` — Clock component
- [x] `src/components/MoveHistory.jsx` — Move history (SAN)

#### 1.4: Session Token Infrastructure

- [x] Server: sinh nanoid cho mỗi player khi join, lưu vào `room.sessionTokens[color]`
- [x] Map `sessionToken → { roomId, color }` trong `disconnectedSessions`
- [x] Client: lưu `sessionToken` vào `localStorage` key `'chess_session_token'`

**Estimated:** ~4–5 giờ (đã hoàn thành trong phiên scaffold)

---

### Phase 2: Multiplayer Core — Giao tiếp Real-time đúng Spec

> **Mục tiêu:** Socket.IO events đúng contract, 2 người chơi đồng bộ bàn cờ

#### 2.1: Implement Room Manager đúng Spec

- [ ] `createRoom()`: sinh mã phòng 8 ký tự (nanoid), set `status: 'waiting'`, `players: { white: socket.id, black: null }`, sinh `sessionToken` cho người tạo
- [ ] `joinRoom()`: gán phe Đen cho người thứ 2, sinh `sessionToken`, set `status: 'playing'`, emit `opponent_joined`
- [ ] Từ chối người thứ 3: emit `room_full`
- [ ] **BUG FIX**: `room.players` check thay vì `room.length` (rooms là Map)
- [ ] **BUG FIX**: Khi tạo phòng — socket KHÔNG được disconnect sau khi tạo room

#### 2.2: Implement Socket Events đúng Contract

- [ ] Event `create_room` → `room_created` với `{ roomId, sessionToken }`
- [ ] Event `join_room` → `joined` với `{ color, roomId, fen, sessionToken }` + `opponent_joined` cho người kia
- [ ] Event `join_room` → `room_full` / `room_not_found` cho error
- [ ] Event `make_move` → validate → broadcast `move_made` tới opponent
- [ ] Từ chối nước đi nếu không phải lượt mình (`not_your_turn`)

#### 2.3: GameBoard — Auto-flip + Auto-promote

- [ ] Pass `orientation` prop (white/black) vào `ChessBoard`
- [ ] react-chessboard `boardOrientation={orientation}`
- [ ] **Auto-promote Queen**: `onPromotion={() => 'q'}`
- [ ] Player info hiển thị đúng bên: Đen (trái) | Trắng (phải)
- [ ] **BUG FIX**: Cả 2 phía đều hiển thị quân Trắng → fix `boardOrientation`

**Estimated:** ~4–6 giờ

---

### Phase 3: Game Logic — Clock, PGN, Game Over

> **Mục tiêu:** Server-side clock 15+0 pause/resume, thu thập dữ liệu AI, game over detection đầy đủ

#### 3.1: Implement Clock Manager (Server-side)

- [ ] `initClock(roomId, timeControl)`: parse `15+0` → `{ initial: 900, increment: 0 }`
- [ ] `startClock(roomId)`: bắt đầu `setInterval` 1 giây
- [ ] `pauseClock(roomId, side)`: dừng decrement cho bên `side`
- [ ] `resumeClock(roomId, side)`: tiếp tục decrement cho bên `side`
- [ ] `switchClock(roomId)`: dừng bên hiện tại, chạy bên kia, record `timeSpent` vào `room.clockTimes[]`
- [ ] `pauseAllClock(roomId)`: pause cả 2 bên (khi cả 2 disconnect)
- [ ] `getTimes(roomId)`: trả `{ whiteTime, blackTime }`
- [ ] `getActiveSide(roomId)`: trả `'white' | 'black' | null`
- [ ] Khi đồng hồ về 0 → callback `'timeout'` → trigger `handleGameOver`
- [ ] `stopClock(roomId)`: dọn dẹp interval
- [ ] `resetClock(roomId)`: reset về initial

#### 3.2: Wire Clock vào Game Flow

- [ ] Khi opponent join → `initClock` + `startClock` (Trắng bắt đầu)
- [ ] Mỗi `make_move` → `switchClock` → record `timeSpent`
- [ ] Broadcast `clock_update` mỗi 1 giây tới cả 2 client (kèm `activeSide`)
- [ ] Khi `game_over` → `stopClock`
- [ ] Khi disconnect → `pauseClock(roomId, disconnectedColor)`
- [ ] Khi reconnect → `resumeClock(roomId, reconnectedColor)`

#### 3.3: Implement GameOver Detection (Server-side)

- [ ] Dùng chess.js kiểm tra `isCheckmate()`, `isStalemate()`
- [ ] `handleResign(color)`: bên color bấm → bên kia thắng
- [ ] `handleTimeout(roomId, color)`: trigger khi clock về 0
- [ ] Sau khi detect game over → gọi `handleGameOver(room, result, reason, io)`
- [ ] Đặt `finishedAt = Date.now()` để tính room cleanup timer

#### 3.4: Implement PGN Collection

- [ ] Mỗi nước đi → lưu `san` (VD: "e4", "Nf3", "O-O") vào `room.moves[]`
- [ ] Implement `buildPGN(moves[])`: "1. e4 e5 2. Nf3 Nc6 3. Bb5 a6"
- [ ] Verify PGN có thể parse ngược bằng chess.js

#### 3.5: Frontend Clock Component (Server-sync)

- [ ] Props: `{ whiteTime, blackTime, activeSide, myColor }`
- [ ] Frontend suy ra status từ `activeSide` + opponent disconnect state:
  - `activeSide === 'white'` → Trắng: running, Đen: paused
  - `activeSide === 'black'` → Đen: running, Trắng: paused
- [ ] Format `MM:SS`
- [ ] Running: màu cam. Paused: màu xám. Stopped: màu đỏ.
- [ ] Low time warning: < 30s và < 10s

#### 3.6: Frontend Move History Component (SAN)

- [ ] Props: `moves: string[]`
- [ ] Format 2 cột: số nước | Trắng | Đen
- [ ] Auto-scroll xuống cuối khi có nước mới

#### 3.7: Frontend — Resign + Exit Buttons

- [ ] Nút "Xin Thua" → emit `resign` event lên server
- [ ] Nút "Thoát Phòng" → emit `exit_room` event
- [ ] Server: `exit_room` khi đang `waiting` → xóa phòng. Khi đang `playing` → xử lý như resign.

#### 3.8: Waiting Overlay

- [ ] Hiện khi đã join nhưng `status === 'waiting'`
- [ ] Hiển thị link phòng để copy + nút "Copy Link" (dùng `navigator.clipboard.writeText`)

**Estimated:** ~10–12 giờ

---

### Phase 4: Disconnect / Reconnect Handling

> **Mục tiêu:** Giữ room state, clock pause/resume, session token restore

#### 4.1: Handle Disconnect (Server-side)

- [ ] `socket.on('disconnect')`: lookup room by socketId → tìm color
- [ ] Set `room.players[color] = null`
- [ ] Call `pauseClock(roomId, color)` — clock của bên đó dừng
- [ ] Check: nếu `room.players.white === null && room.players.black === null`:
  - Cả 2 disconnect → đặt cleanup timer 30 giây
- [ ] Else: 1 người còn → emit `opponent_disconnected` tới player còn lại

#### 4.2: Handle Reconnect (Server-side)

- [ ] Event `reconnect`: lookup `disconnectedSessions`
- [ ] Match sessionToken → get `roomId` và `color`
- [ ] Verify room còn tồn tại → restore
- [ ] Update `room.players[color] = socket.id`
- [ ] Call `resumeClock(roomId, color)`
- [ ] Cancel cleanup timer nếu có
- [ ] Emit `reconnected` tới player với full game state
- [ ] Emit `opponent_reconnected` tới opponent
- [ ] Xóa khỏi `disconnectedSessions`

#### 4.3: Handle Reconnect (Client-side)

- [ ] On mount: check localStorage for `sessionToken`, emit `reconnect` if found
- [ ] Socket.IO auto-reconnect: `socket.io.on('reconnect_attempt', ...)` → check token
- [ ] Khi nhận `reconnected`: restore game state (fen, moves, clock, etc.)
- [ ] Khi nhận `opponent_reconnected`: tắt DisconnectedOverlay

#### 4.4: Frontend Disconnected Overlay

- [ ] `DisconnectedOverlay.jsx`: hiện khi nhận `opponent_disconnected`
- [ ] Text: "Đối thủ đã disconnect. Đang chờ reconnect..."
- [ ] Đồng hồ opponent hiển thị trạng thái PAUSED

#### 4.5: Room Cleanup System

- [ ] `roomCleanupTimers` Map: `roomId → setTimeoutId`
- [ ] `scheduleRoomCleanup(roomId, delayMs)`: đặt timer → `cleanupRoom(roomId)`
- [ ] `cancelCleanupTimer(roomId)`: clear timer nếu có
- [ ] `cleanupRoom(roomId)`: xóa phòng khỏi `rooms`, xóa khỏi `disconnectedSessions`, stop clock
- [ ] Sau `game_over`: `scheduleFinishedRoomCleanup(roomId)` → 5 phút
- [ ] Khi cả 2 disconnect (Phase 4.1): `scheduleRoomCleanup(roomId, 30000)` → 30 giây
- [ ] Khi `waiting` timeout: `scheduleRoomCleanup(roomId, 300000)` → 5 phút

**Estimated:** ~6–8 giờ

---

### Phase 5: AI Integration — Kết nối AI Engine + Result Modal

> **Mục tiêu:** Kết nối thực sự với MMD-G2, hiện ELO + Explanation

#### 5.1: Implement AI Client (Server-side)

- [ ] `aiClient.js`: HTTP POST dùng native `fetch`
- [ ] Build request body: `{ pgn, clock_times, result, time_control: "15+0" }`
- [ ] Env var `AI_ENGINE_URL` (default: `http://localhost:8000`)
- [ ] Timeout 30 giây dùng `AbortSignal.timeout(30000)`
- [ ] Parse response, validate `success` field
- [ ] Return `null` nếu lỗi (để caller xử lý fallback)

#### 5.2: Wire AI vào Game Over Flow

- [ ] `handleGameOver(room, result, reason, io)`:
  1. Emit `game_over` + `ai_loading` tới cả 2 client
  2. `buildPGN` + lấy `clockTimes`
  3. `await requestELOPrediction(room)`
  4. Nếu thành công → emit `ai_result`
  5. Nếu lỗi → emit `ai_error` với message fallback
  6. Đặt `finishedAt = Date.now()` cho cleanup timer

#### 5.3: Frontend AI Loading Overlay

- [ ] State: `aiLoading: boolean`
- [ ] Khi nhận `ai_loading` → set `aiLoading = true`
- [ ] Overlay toàn màn hình: spinner + "Đang phân tích ELO bằng AI..."
- [ ] Khi nhận `ai_result` hoặc `ai_error` → tắt overlay

#### 5.4: Frontend Result Modal

- [ ] `ResultModal.jsx`:
  - Header: kết quả ("Trắng Thắng / Đen Thắng / Hòa") + reason
  - 2 ô ELO lớn: Trắng | Đen
  - Stats: ECO code + name, CPL, Blunders
  - Explanation: scrollable text box
  - Nút `[ Chơi Lại ]`
- [ ] State từ `GameRoom`: `aiData`, `aiError`, `gameOverData`
- [ ] Render modal khi `aiData` hoặc `aiError` có giá trị

#### 5.5: Handle AI Error Fallback

- [ ] Khi nhận `ai_error`: hiện modal với kết quả cơ bản (Thắng/Thua/Hòa)
- [ ] Message: "Không thể kết nối AI Engine. Phân tích ELO tạm thời không khả dụng."
- [ ] Vẫn cho bấm "Chơi Lại" bình thường

**Estimated:** ~4–6 giờ

---

### Phase 6: Polish & QA — Hoàn thiện & Testing

> **Mục tiêu:** Tất cả checklist nghiệm thu pass, UX tốt

#### 6.1: Play Again Feature

- [ ] Server: emit `reset_game` tới cả 2 client khi bấm
- [ ] Frontend: reset `chess` instance, FEN, moves, clockTimes
- [ ] Reset clock: `resetClock(roomId)` → `startClock(roomId)`
- [ ] Xóa `result`, `resultReason` khỏi room state, reset `finishedAt`
- [ ] Ẩn Result Modal

#### 6.2: UX Polish

- [ ] Theme tối (dark mode) cho Lobby và Game
- [ ] **Check indicator**: dùng `chess.inCheck()` để detect. react-chessboard không có built-in → custom overlay CSS highlight ô vua đỏ
- [ ] Copy link button trong WaitingOverlay

#### 6.3: Debug Logging

- [ ] Log mỗi nước đi: `{ move, timeSpent, fen }`
- [ ] Log khi `game_over`: `{ result, reason, pgn, clockTimes.length }`
- [ ] Log AI request/response (success và error)
- [ ] Log disconnect/reconnect events với session token
- [ ] Log room cleanup: khi nào phòng bị xóa, lý do

#### 6.4: Test Scenarios (chạy trên trình duyệt)

- [ ] A tạo phòng → B vào phòng → A đi nước đầu → B đi nước tiếp (test flow cơ bản)
- [ ] A tạo phòng → A bị disconnect → A reconnect (test socket disconnect)
- [ ] A tạo phòng → B vào → B disconnect → B reconnect (test clock pause/resume)
- [ ] A tạo phòng → B vào → A hết giờ (test timeout)
- [ ] A tạo phòng → B vào → B bấm Xin Thua (test resign)

#### 6.5: Checklist Nghiệm Thu

- [ ] Tất cả 23 items trong Requirements "Success Criteria" pass

**Estimated:** ~3–4 giờ

---

## Dependencies

### External Dependencies

- **AI Engine Server** (repo MMD-G2) phải chạy trên cổng 8000 để test end-to-end
- Hoặc AI Client cần handle graceful degradation khi AI Engine chưa có

### Internal Dependencies (Phase ordering)

```
Phase 1 (Foundation)
    ↓
Phase 2 (Multiplayer Core) ← cần Phase 1 xong
    ↓
Phase 3 (Game Logic) ← cần Phase 2 socket events đúng
    ↓
Phase 4 (Disconnect/Reconnect) ← cần Phase 2 (Room Manager) + Phase 3 (Clock)
    ↓
Phase 5 (AI Integration) ← cần Phase 3 (PGN collection)
    ↓
Phase 6 (Polish & QA)
```

### Trong Phase

- 1.2 (Backend files) → 1.3 (Frontend files): thứ tự tự do
- 2.1 (Room Manager) → 2.2 (Socket events): Room Manager trước
- 2.3 (Board) → 2.2 (Socket events): tự do
- 3.5 (Clock UI) → 3.1 (Clock Manager): Clock Manager trước
- 4.2 (Reconnect server) → 4.3 (Reconnect client): server trước
- 5.3+5.4 (UI) → 5.2 (Wire AI): Wire AI trước
- 6.2 (UX) → 6.1 (Play Again): tự do

---

## Timeline & Estimates

| Phase     | Tên                  | Estimate      | Ghi chú                                      |
| --------- | -------------------- | ------------- | -------------------------------------------- |
| P1        | Foundation           | 4–5 giờ       | Scaffold, setup, session token               |
| P2        | Multiplayer Core     | 4–6 giờ       | Socket events + Board (bug fixes)            |
| P3        | Game Logic           | 10–12 giờ     | Clock pause/resume + PGN + game over         |
| P4        | Disconnect/Reconnect | 6–8 giờ       | Session token + clock pause/resume + cleanup |
| P5        | AI Integration       | 4–6 giờ       | AI client + Result Modal                     |
| P6        | Polish & QA          | 3–4 giờ       | Play Again + debug + checklist               |
| **Total** |                      | **31–41 giờ** | ~1.5 tuần làm việc (6–8 ngày)                |

---

## Risks & Mitigation

| Risk                                     | Likelihood      | Impact     | Mitigation                                                                |
| ---------------------------------------- | --------------- | ---------- | ------------------------------------------------------------------------- |
| AI Engine chưa chạy được khi test        | Cao             | Thấp       | AI Client handle null return + fallback UI                                |
| Clock drift (server vs client)           | Thấp            | Trung bình | Server-side clock + sync 1 giây                                           |
| Socket event mismatch (server vs client) | Trung bình      | Cao        | Event names chuẩn hóa                                                     |
| Session token collision                  | Thấp            | Cao        | Dùng nanoid (entropy cao)                                                 |
| Reconnect không restore đúng state       | Trung bình      | Cao        | Test kỹ: disconnect giữa ván, reconnect giữa ván, reconnect sau game_over |
| PGN format không parse được bởi AI       | Thấp            | Cao        | Test với chess.js parse ngược sau mỗi build                               |
| Room cleanup chạy quá sớm                | Trung bình      | Cao        | Cancel cleanup timer ngay khi có reconnect                                |
| Clock pause/resume logic phức tạp        | Trung bình      | Trung bình | Vẽ state diagram trước khi code                                           |
| Socket disconnect sau khi tạo phòng      | Cao (đã xảy ra) | Cao        | Fix logic `joinRoom` không trigger socket disconnect                      |

---

## Resources Needed

| Resource          | Chi tiết                                                             |
| ----------------- | -------------------------------------------------------------------- |
| **Team**          | 1 web developer                                                      |
| **AI Engine URL** | `http://localhost:8000` (hoặc cấu hình qua env `AI_ENGINE_URL`)      |
| **Node.js**       | >= 18 (cho native `fetch`)                                           |
| **NPM packages**  | react-router-dom, concurrently — others đã có                        |
| **Tools**         | 2 trình duyệt (hoặc 1 trình duyệt + 1 incognito) để test multiplayer |
| **AI Engine**     | Repo MMD-G2 chạy `uvicorn src.ai_engine.main:app --port 8000`        |

---

## Known Issues (from previous sessions)

Các bug đã phát hiện trong quá trình phát triển, cần fix triệt để:

1. **Game starts immediately with 1 player** — game không nên bắt đầu cho đến khi có đủ 2 người. Fix: `status !== 'playing'` guard trước khi start clock.
2. **Player 2 joining gets "room_full"** — logic check `players.black` bị sai. Fix: đảm bảo `joinRoom` chỉ gán khi `players.black === null`.
3. **Socket disconnect on room creation** — user tạo phòng nhưng bị disconnect ngay. Fix: kiểm tra `socket.emit` không trigger `disconnect` event.
4. **Room not found on fresh creation** — race condition giữa `createRoom` và `joinRoom`. Fix: đảm bảo room được tạo trước khi lookup.
5. **Both sides show white pieces** — `boardOrientation` logic sai cho phe Đen. Fix: truyền đúng orientation prop (`'black'` cho người chơi màu Đen).
6. **Black plays first instead of white** — `currentTurn` khởi tạo sai. Fix: `currentTurn` luôn bắt đầu là `'w'` (Trắng đi trước).
