# Drawing_Canvas
A browser-based collaborative drawing app built using **HTML Canvas**, and **JavaScript**.  
Multiple users can draw on the same canvas in real-time, with live cursor tracking, undo, clear, and per-user color + username.

---

## 🚀 Features

✅ Real-time multi-user drawing  
✅ Live cursor tracking for all users  
✅ Undo & Clear for entire room  
✅ Auto-generated usernames (`user-1`, `user-2`, …)  
✅ Online user list with activity indicator  
✅ No frameworks on frontend (JS + Canvas)  

---

## 📁 Project Structure

FLAM_ASSIGN/
│
├── client/ # Frontend
│ ├── index.html
│ ├── canvas.js
│ ├── main.js
│ ├── websocket.js
│ ├── style.css
│ └── node_modules/
│
└── server/ # Backend
├── server.js
├── rooms.js 
├── drawing-state.js
└── node_modules/


## 🛠️ Setup

### ✅ 1. Install dependencies

From the **project root**:
npm install

✅ 2. Start the server
npm start
node server/server.js

✅ 3. Open the app in browser

http://localhost:3000
👥 How to Test With Multiple Users
Open two browser tabs pointing to:
http://localhost:3000

Draw in one tab → it should instantly appear in the other

Each tab will appear as user-1, user-2, etc.

Try:

Drawing simultaneously

Undo (Ctrl+Z)

Clear button

Changing brush color and size

Watching cursor labels update live

✅ No need to create accounts
✅ No login / auth required

🐞 Known Limitations / Bugs
Issue	Status
No redo feature implemented	❌ Not yet
Undo removes last global stroke (not per-user)	✅ Intended but could be improved
Canvas is not persistent — refresh resets it
Large fast strokes may drop some points
No mobile drawing support yet	❌ Desktop only
User numbering (user-1...) resets when server restarts	✅ Expected

⏱️ Time Spent on Project
Task	Time
Research + planning	~1 hour
Backend implementation	~2 hours
Canvas drawing engine + sync	~3 hours
Cursor tracking + user list UI	~1 hour
Bug fixing + final polish	~1 hour
README + Architecture	~15 mins

Total: ~7 hours

🧩 Possible Improvements (Future Work)
✅ Support drawing on mobile/touch devices

✅ Add redo stack

✅ Save canvas state in DB (Redis / MongoDB)

✅ Multi-room UI selector (instead of query param)

✅ Allow custom usernames before joining

✅ Export canvas as PNG

✅ Add chat sidebar

✅ Add per-user undo instead of global undo
