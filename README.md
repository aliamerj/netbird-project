# Sharing files with netbird 
Implement a Taildrop-like file transfer feature for NetBird, we need to create a secure, peer-to-peer file-sharing system that integrates naturally into NetBird’s infrastructure allowing users to send files between their personal devices on a Netbird network.

## Overall Implementation Core Logic: 
### Build the file transfer functionality to work over NetBird’s peer-to-peer connections.
-  **Platform-Specific Code**:
    - **macOS/iOS/Android**: Integrate with the share menu (e.g., “Share via NetBird”).
    - **Windows**: Add to the share context menu or NetBird client interface.
    - **Linux**: Develop a command (e.g., `netbird send <file> <device>`)

## TASK 1: File Transfer Activation, Permissions, and Default Save Path
In this first step, I’ll implement the core logic for peer-to-peer file transfer inside the NetBird agent — following a model similar to SSH’s controlled file copy behavior.

### 🧩 What I’ll Do
- **File Transfer Listener (Opt-In)**
   - Add a new CLI/config flag (e.g., `--file-transfer`) to enable an agent-side listener that waits for incoming file transfer requests.
   - Can run persistently or be started on-demand via internal signal.


- **Protobuf-Based Control Messages** (used in peer-to-peer communication) to support new message types for file transfer: `file-offer`,`file_accept`,`file_deny`, `file_meta` and `file_chunk`.

- **User Authorization Flow** — When someone tries to send a file:
   - Receiver gets a request with file metadata (size, name, sender)
   - Receiver must approve via CLI/desktop UI
   - Optionally allow auto-accept from trusted peers via config

- **Default Save Path via Agent Settings**
  - Add support for a default download directory (e.g. `~/NetBirdTransfers`)
  - Allow override via CLI flag or agent config

- **Handle file streaming over the established tunnel**:
    - Once approved, file chunks will be streamed directly over the existing encrypted tunnel using a TCP/Unix socket connection negotiated internally.
      - No extra open ports
      - Data stays within the mesh tunnel
      - Uses same NAT traversal as other NetBird traffic

- **CLI prototype for Linux to send and receive files using the command**:
```bash
netbird send <file-path> <peer-id>
```
   - Uses internal messaging and existing peer connections
   - Will print file transfer status to terminal

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
