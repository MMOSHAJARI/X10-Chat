# X10 Chat 💬

Encrypted internal team messaging — WhatsApp-style, runs on your own network.

## Features
- 🔒 **End-to-end encryption** — RSA-OAEP + AES-GCM. Server never sees plaintext.
- 💬 **Real-time messaging** — WebSocket with typing indicators & read receipts
- 📎 **File transfer** — Images, documents, videos up to 200 MB
- 📹 **Video & audio calls** — WebRTC peer-to-peer (no relay server needed on LAN)
- 🔗 **Encrypted invite links** — Single-use, 7-day expiry
- 📱 **Mobile-friendly** — WhatsApp-like UI, works great on phones
- 🚀 **Zero database** — Stores everything in a local JSON file

---

## Quick Start

### 1. Install
```bash
npm install
```

### 2. Run
```bash
npm start
```

### 3. Open
Visit `http://YOUR_SERVER_IP:3000` in your browser.

The server prints your IP addresses on startup:
```
╔════════════════════════════════════════╗
║         X10 Chat Server — Ready        ║
╠════════════════════════════════════════╣
║  🌐  http://192.168.1.100:3000         ║
║  🏠  http://localhost:3000             ║
╚════════════════════════════════════════╝
```

### 4. First-time Setup
On first run, you'll see a **"Create Admin"** form. Fill it in — this is your admin account.

### 5. Invite Your Team
1. Log in as admin
2. Click the **👤+** button (top left of sidebar)
3. Copy the invite link and send it to your colleague (via email, Slack, etc.)
4. They open the link, choose a username and password — done!

---

## Deploying on Your Network

### Option A: Run directly
```bash
# Keep it running after you close the terminal
npx pm2 start server.js --name x10chat
npx pm2 save
npx pm2 startup
```

### Option B: Run as a background service (Linux systemd)
```ini
# /etc/systemd/system/x10chat.service
[Unit]
Description=X10 Chat
After=network.target

[Service]
WorkingDirectory=/path/to/x10chat
ExecStart=/usr/bin/node server.js
Restart=always
User=youruser
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl enable x10chat
sudo systemctl start x10chat
```

### Custom port
```bash
PORT=8080 npm start
```

---

## Security Architecture

### Encryption
- Each user generates an **RSA-OAEP 2048-bit key pair** in the browser on first login
- The **private key never leaves the device** (stored in localStorage)
- The public key is registered with the server
- To send a message:
  1. Generate a random **AES-256-GCM** key
  2. Encrypt the message with AES
  3. Encrypt the AES key with the recipient's RSA public key
  4. Send the encrypted blob to the server
- The server **only stores and routes ciphertext** — it cannot read any messages

### Invitations
- Invite tokens are 32-byte random hex strings (256 bits of entropy)
- Single-use, expire after 7 days
- The invite URL contains the token — share it via a trusted channel

### Sessions
- Session tokens are 40-byte random hex strings
- Server-side sessions with 30-day expiry
- Max 5 active sessions per user

### Passwords
- Hashed with **bcrypt** (cost factor 12)
- Never stored or transmitted in plaintext

---

## Data Storage

All data is stored in `data/store.json`:
- User accounts (hashed passwords, public keys)
- Active sessions
- Invite tokens
- File metadata

Uploaded files are stored in `data/uploads/`.

**Back up the `data/` folder** to preserve all user accounts and uploads.

---

## Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `PORT` | `3000` | Server port |

---

## File Transfer Limits

- Maximum file size: **200 MB**
- Blocked extensions: `.exe`, `.bat`, `.cmd`, `.sh`, `.ps1`, `.com`

---

## Browser Requirements

Modern browsers that support:
- Web Crypto API (all modern browsers)
- WebRTC (Chrome, Firefox, Edge, Safari 11+)
- WebSocket (all modern browsers)

---

## Project Structure

```
x10chat/
├── server.js          ← Node.js backend
├── package.json
├── README.md
├── public/
│   └── index.html     ← Complete frontend (HTML + CSS + JS)
└── data/              ← Created on first run
    ├── store.json     ← User/session/invite data
    └── uploads/       ← Uploaded files
```

---

## License
MIT — use freely within your organisation.
