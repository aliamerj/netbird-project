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
- **Introduce a new management-side config flag (e.g., enableFileTransfer: true)**
   - that governs whether the agent is allowed to participate in file transfer at all.
   - This is synced to the agent and acts as a hard gatekeeper
If disabled, the feature is entirely inactive regardless of CLI flags or user actions

- **Local Listener with Optional Runtime Signal**
  Once management has enabled it, the agent can start a local listener for file transfer requests:
   - Either automatically on startup (based on CLI/config flag like` --file-transfer`)
   - Or dynamically triggered at runtime via a dedicated internal signal message (e.g., `start_file_transfer_listener`)


- **Protobuf-Based Control Messages** (used in peer-to-peer communication) to support new message types for file transfer:
  - `file_offer` → initiate
  - `file_accept` / `file_deny`
  - `file_meta` → metadata (size, name)
  - `file_chunk` → actual data
  - `transfer_complete` / `transfer_error`

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
   - Works only if both peers are management-enabled and listening
   - Prints real-time status in teminal 

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
