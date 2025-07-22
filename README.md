# Sharing files with netbird 
Implement a Taildrop-like file transfer feature for NetBird, we need to create a secure, peer-to-peer file-sharing system that integrates naturally into NetBird’s infrastructure allowing users to send files between their personal devices on a Netbird network.

## Overall Implementation Core Logic: 
### Build the file transfer functionality to work over NetBird’s peer-to-peer connections.
-  **Platform-Specific Code**:
    - **macOS/iOS/Android**: Integrate with the share menu (e.g., “Share via NetBird”).
    - **Windows**: Add to the share context menu or NetBird client interface.
    - **Linux**: Develop a command (e.g., `netbird send <file> <device>`)

## TASK 1: Internal API & Message Handling for File Sharing
In this first step, I’ll implement the core logic for file sharing within NetBird agents, enabling secure file transfers between peers using NetBird’s existing peer-to-peer tunnel infrastructure.

### 🧩 What I’ll Do
- **Extend Wiretrustee messaging system** (used in peer-to-peer communication) to support new message types for file transfer: `file-offer`, `file-complete`, etc.
- **Define protobuf messages** and update message handlers on both sender and receiver sides of the agent.
- **No backend service involved** — communication will occur directly between agents using the existing signaling and NAT traversal system (via STUN/ICE).
- **Handle file streaming over the established tunnel**:
    - Use chunked transfer or simple streaming with progress feedback.
    - Encrypt file payloads end-to-end.
- **CLI prototype for Linux to send and receive files using the command**:
```bash
netbird send <file-path> <peer-id>
```

## TASK 2: Cross-Platform File Sharing UX Integration
Now that we have the internal file transfer logic working (from Task 1), this task focuses on making it usable and accessible across major platforms by integrating with native OS interfaces.

### 🧩 What I’ll Do
- Linux ( already done in Task 1)
   - Extend the CLI UX (from Task 1) with optional interactive prompts.
   - Optionally create a minimal GTK or terminal UI to make selection easier.

- macOS / iOS / Android
  - Create component (i.e. "Share via NetBird").
  - When a file is shared, it triggers a background call to netbird send <file> <target-device> using the CLI or internal bindings.

- Windows
   - Add a  menu item ("Send with NetBird")
   - Under the hood, it calls the same file-sending command.


#### 🔁 How It Works Technically
- All platforms will reuse the message handlers and transfer logic from Task 1 — no new protocol or backend work is needed.
- UI or shell integrations simply act as wrappers around the core agent, either invoking a CLI command or talking directly to the local NetBird service (via Unix socket or similar, if exposed).

## TASK 3: Nice to have
- Add optional settings like download location, transfer history, and permissions.
- Notify users when a new file is received.
