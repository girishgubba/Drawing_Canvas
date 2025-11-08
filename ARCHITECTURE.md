# System Architecture – Realtime Collaborative Whiteboard

This document explains the internal architecture of the system, including data flow, socket protocol, undo/redo design, concurrency handling, and performance decisions.

---

## 📡 1. High-Level Data Flow

![data_flow_diagram_simple](https://github.com/user-attachments/assets/6a4f0226-27cd-4aaf-b54b-bef1c37cbb05)


### **Flow of a Draw Event**

User draws on canvas (pointermove)
↓
Browser locally renders stroke for instant feedback
↓
Client emits Socket.IO event → stroke:points
↓
Server receives and re-broadcasts to all clients in room (except sender)
↓
Other users receive event → append points → render stroke on canvas


✅ Drawing always renders immediately for the sender  
✅ Other users render after server broadcast (100–200ms)

---

## 🔌 2. WebSocket Message Protocol

| Event Name        | Direction | Payload Example | Purpose |
|-------------------|-----------|-----------------|---------|
| `join`            | client → server | `{ roomId, color }` | Register user + join room |
| `welcome`         | server → client | `{ userId, users, oplog }` | Sync existing canvas + users |
| `users:update`    | server → all | `[ { id, name, color, online } ]` | Updates sidebar list |
| `cursor`          | both ways | `{ x, y, userId }` | Live cursor position |
| `stroke:start`    | both ways | `{ id, eraser, color, width }` | Begin new stroke |
| `stroke:points`   | both ways | `{ id, points[] }` | Append stroke points |
| `stroke:end`      | both ways | `{ id }` | Finalize stroke |
| `undo`            | client → server | none | Request global undo |
| `undo:applied`    | server → all | `{ op }` | Remove last operation |
| `clear`           | client → server | none | Clear full canvas |
| `cleared`         | server → all | none | Notify clients to wipe canvas |



## 🔄 3. Undo / Redo Strategy

### ✅ Current Behavior

- Undo is **global**, not per-user
- The server keeps a list of all strokes inside `rooms[roomId].oplog`
- Undo removes the **last stroke in history**, not based on author

[ strokeA, strokeB, strokeC ] → undo() → [ strokeA, strokeB ]

Undo flow:

Client → emits "undo"
Server → pops last op from oplog
Server → emits "undo:applied" to all clients
Clients → rebuild full canvas by replaying all ops


## ⚡ 4. Performance Decisions

| Optimization | Why |
|--------------|-----|
| Local canvas rendering before socket emit | Zero perceived latency for user |
| Batch point send (16ms throttle) | Reduces socket spam while preserving smooth drawing |
| Off-screen canvas buffer (`offCanvas`) | Redraw is instant, avoids re-rendering full DOM |
| No database writes | Faster real-time speed; avoids I/O delay |
| Socket broadcast excludes sender | Prevents flicker and double-render |
| Messages are minimal JSON | Keep bandwidth low for high-frequency events |

🚀 Current performance supports **~20 simultaneous users** on a typical laptop without lag.

---

## ⚔️ 5. Conflict Resolution (Simultaneous Drawing)

### Problem
What if **User A** and **User B** draw at the exact same time?

### Solution
Each stroke has a **unique ID**, and every active stroke is rendered independently:

User A stroke → id "s_abc1"
User B stroke → id "s_def9"

So even overlapping strokes do not conflict — they are just separate ops in the log.

Canvas is not “locked” for any user → fully concurrent drawing is allowed.

### Why this works

- Canvas is *append-only drawing* (not object-based)
- Order of arrival does not affect correctness
- Every stroke is isolated
- Only undo/clear modifies history, not stroke order

---

## 🧠 6. Server State Model

rooms = {
"roomId123": {
oplog: [ strokeOp1, strokeOp2, ... ],
users: [
{ id, name, color, online },
...
],
userCount: 3 // for username assignment
}
}


## 🛠️ 7. Files & Responsibilities

| File | Role |
|------|------|
| `client/canvas.js` | Canvas rendering + event bindings |
| `client/websocket.js` | Wraps socket emit/on logic |
| `client/main.js` | UI setup + joins server |
| `server/server.js` | Handles WS events + broadcast + room state |
| `server/rooms.js` | (optional) Extracted room data logic |
| `server/drawing-state.js` | (optional) Helper for stroke/state mgmt |

---
