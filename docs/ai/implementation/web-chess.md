---
phase: implementation
title: Implementation Guide — ChessWeb PoC
description: Technical implementation notes, patterns, code guidelines, và trạng thái hiện tại của ChessWeb PoC. Bao gồm development setup, code structure, integration points, và các lưu ý kỹ thuật.
---

# Implementation Guide: ChessWeb PoC

> **Trạng thái:** Đang phát triển — Phase 1 (Foundation) đã scaffold, Phase 2+ còn pending.
> **Cập nhật lần cuối:** 2026-04-29

## Development Setup

### Prerequisites

- **Node.js** >= 18 (cho native `fetch` API)
- **Python + Conda** (`MMDS` environment) — cho AI Engine (MMD-G2)
- **Git** — version control

### Environment Setup

```bash
# 1. Clone/pull repos
git clone <ChessWeb-repo>
git clone <MMD-G2-repo>

# 2. Setup ChessWeb
cd ChessWeb
npm install

# 3. Setup AI Engine (MMD-G2) — chạy khi cần test AI
cd MMD-G2
conda activate MMDS
uvicorn src.ai_engine.main:app --port 8000 --reload
```

### Running the Application

```bash
# Development — chạy cả FE + BE song song
cd ChessWeb
npm run start

# Chỉ Frontend (Vite dev server)
npm run dev

# Chỉ Backend (Node.js server)
npm run server

# Production build
npm run build
npm run preview
```

### Configuration

| Variable             | File   | Default                 | Mô tả                |
| -------------------- | ------ | ----------------------- | -------------------- |
| `VITE_SOCKET_URL`    | `.env` | `http://localhost:3000` | Socket.IO server URL |
| `VITE_AI_ENGINE_URL` | `.env` | `http://localhost:8000` | AI Engine API URL    |

---

## Code Structure

### Directory Layout

```
ChessWeb/
├── server/                          # Backend Node.js — ESM modules
│   ├── index.js                    # Entry: Express + Socket.IO setup
│   ├── roomManager.js              # Room lifecycle + session tokens
│   ├── gameLogic.js               # Move validation + PGN build
│   ├── clockManager.js            # Server-side clock
│   └── aiClient.js               # AI Engine HTTP client
│
├── src/                            # Frontend React — ESM modules
│   ├── main.jsx                   # Entry point
│   ├── App.jsx                   # Router (react-router-dom v7)
│   ├── socket.js                 # Socket.IO client singleton
│   ├── index.css                 # Global styles (dark theme, CSS variables)
│   ├── pages/
│   │   ├── Lobby.jsx             # Tạo/nhập phòng
│   │   └── GameRoom.jsx          # Phòng đấu (placeholder — cần implement)
│   └── components/
│       ├── ChessBoard.jsx         # Bàn cờ wrapper (placeholder — cần implement)
│       ├── Clock.jsx             # Đồng hồ (scaffolded, cần wire)
│       ├── MoveHistory.jsx       # Lịch sử nước đi (scaffolded)
│       ├── ResultModal.jsx      # Bảng kết quả (placeholder)
│       ├── WaitingOverlay.jsx    # Chờ đối thủ (placeholder)
│       └── DisconnectedOverlay.jsx
│
├── package.json                    # ESM project
├── vite.config.js
├── eslint.config.js
└── index.html
```

---

## Current Implementation Status

### ✅ Đã scaffold (Phase 1)

**Backend (`server/`):**

- `server/index.js` — Express + Socket.IO setup trên port 3000
- `server/roomManager.js` — ESM exports, `rooms` Map, `createRoom()`, `joinRoom()` skeleton
- `server/gameLogic.js` — Skeleton với chess.js import
- `server/clockManager.js` — Skeleton (clock state management)
- `server/aiClient.js` — Skeleton (HTTP client placeholder)

**Frontend (`src/`):**

- `src/socket.js` — Socket.IO singleton với `autoConnect: false`
- `src/main.jsx` — React 19 entry point
- `src/App.jsx` — React Router v7 với routes `/` và `/room/:roomId`
- `src/index.css` — Dark theme CSS variables
- `src/pages/Lobby.jsx` — Lobby page (scaffolded)
- `src/pages/GameRoom.jsx` — GameRoom placeholder
- `src/components/Clock.jsx` — Clock component với formatTime
- `src/components/MoveHistory.jsx` — Move history component với SAN pairs

### ⚠️ Cần implement (Phase 2–6)

**Backend:**

- Full `roomManager.js` logic (create, join, disconnect, reconnect)
- Full `gameLogic.js` (validateMove, buildPGN, detectGameOver)
- Full `clockManager.js` (pause/resume/switch, timeout detection)
- Full `aiClient.js` (HTTP POST với fetch)
- `server/index.js` — Socket event handlers đầy đủ

**Frontend:**

- `GameRoom.jsx` — Full implementation với socket event handlers
- `ChessBoard.jsx` — Full chessboard với react-chessboard, auto-flip, auto-promote
- `ResultModal.jsx` — ELO display, AI stats, LLM explanation
- `WaitingOverlay.jsx` — Link sharing, copy button
- `DisconnectedOverlay.jsx` — Opponent disconnect indicator

---

## Integration Points

### Frontend → Backend (Socket.IO)

```javascript
// src/socket.js
import { io } from "socket.io-client";

const SOCKET_URL = import.meta.env.VITE_SOCKET_URL || "http://localhost:3000";

export const socket = io(SOCKET_URL, {
  autoConnect: false,
  transports: ["websocket", "polling"],
});

// Client-side usage:
import { socket } from "./socket";

socket.connect();
socket.emit("create_room");
socket.on("room_created", (data) => {
  /* { roomId, sessionToken } */
});
```

### Backend → AI Engine

```javascript
// server/aiClient.js
const AI_ENGINE_URL = process.env.AI_ENGINE_URL || "http://localhost:8000";

async function requestELOPrediction(room) {
  const pgn = buildPGN(room.moves);
  const payload = {
    pgn,
    clock_times: room.clockTimes,
    result: room.result,
    time_control: "15+0",
  };

  try {
    const response = await fetch(`${AI_ENGINE_URL}/api/predict-elo`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload),
      signal: AbortSignal.timeout(30000),
    });
    const result = await response.json();
    return result.success ? result.data : null;
  } catch (error) {
    console.error("[AI] Connection failed:", error.message);
    return null;
  }
}
```

---

## Error Handling

### Backend Error Strategy

- **Socket errors**: emit `error` event với message cụ thể
- **AI errors**: return `null`, caller emit `ai_error` event
- **Game logic errors**: validate trước, reject early với clear message
- **Room not found**: emit `room_not_found` trước khi thao tác

### Frontend Error Handling

- **Socket connection errors**: hiện `ReconnectingOverlay`
- **Invalid move**: hiện toast/alert với message từ server
- **AI loading**: hiện overlay với spinner
- **AI error**: hiện ResultModal với fallback message

---

## Performance Considerations

- **Clock sync**: server sync mỗi 1 giây, client chỉ hiển thị (không tự đếm)
- **PGN build**: O(n) string concatenation, acceptable cho < 200 nước
- **AI timeout**: 30 giây — không block game flow
- **Room cleanup**: setTimeout thay vì interval để tiết kiệm resources

---

## Security Notes

- **Không có authentication** — chấp nhận cho PoC
- **CORS**: cho phép tất cả origins (dev mode)
- **Session token**: dùng nanoid, lưu trong localStorage (không bảo mật cao, chấp nhận cho PoC)
- **Server-side clock**: chống client manipulation, server là nguồn sự thật

---

## Migration Notes (Nếu cần)

### ESM → CommonJS

Project hiện dùng ESM (`"type": "module"` trong package.json). Nếu cần chuyển về CommonJS:

1. Xóa `"type": "module"` khỏi `package.json`
2. Đổi `import` → `require` trong tất cả `.js` files
3. Đổi `export` → `module.exports` trong server files

### Vite → CRA

Frontend dùng Vite. Nếu cần migrate về CRA (Create React App):

1. `npx create-react-app chess-web --template javascript`
2. Copy `src/`, `public/` files
3. Update `package.json` dependencies
4. Thay `index.js` entry point → `src/index.js` (không có `.jsx`)

---

## Debug Checklist

Khi gặp lỗi, kiểm tra theo thứ tự:

1. **Server đang chạy?** → `npm run server` hoặc `npm run start`
2. **Port 3000 bị chiếm?** → `lsof -i :3000`
3. **Socket kết nối được?** → Check browser DevTools > Network > WS
4. **Room tồn tại?** → Check server logs
5. **Clock chạy?** → Check `clockManager.js` interval
6. **AI Engine chạy?** → `curl http://localhost:8000/docs`
7. **ESM import/export?** → Kiểm tra `"type": "module"` trong package.json
