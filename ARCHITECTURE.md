# System Architecture

## 1. High-Level Architecture
This logical view illustrates how the React frontend interacts with the Node.js backend and external services.

```mermaid
flowchart TD
    %% Nodes
    Client[("💻 Client (Browser)
    React.js + Ace Editor")]
    
    subgraph Services ["External Services"]
        Auth[("🔐 Google OAuth
        (Identity Provider)")]
        JDoodle[("🚀 JDoodle API
        (Code Compilation)")]
        WebRTC_Mesh[("🕸️ WebRTC Mesh
        (P2P Signals)")]
    end
    
    subgraph Backend ["Backend Infrastructure (Node.js)"]
        Server[("⚙️ Express API
        (Room Management)")]
        Socket[("⚡ Socket.io Server
        (Signaling & State)")]
        DB[("🗄️ MongoDB
        (Persistent Storage)")]
    end

    %% Connections
    Client -- "1. Login (Google)" --> Auth
    Client -- "2. Verify/Login" --> Server
    Server -- "3. User Data" <--> DB
    
    Client -- "4. Sync (Diff-Match-Patch)" <--> Socket
    Socket -- "5. Signaling" --> Client
    
    Client -- "6. Compile Request" --> Server
    Server -- "7. Proxy Code" --> JDoodle
    JDoodle -- "8. Output output" --> Server
    
    Client -.-> WebRTC_Mesh
    WebRTC_Mesh -.-> Client
    
    %% Styling
    classDef box fill:#fff,stroke:#333,stroke-width:2px,color:#000;
    class Client,Auth,Server,Socket,DB,JDoodle,WebRTC_Mesh box
```

## 2. Database Design (ER Diagram)
The data model for users and rooms in MongoDB.

```mermaid
erDiagram
    User ||--o{ Room : "owns"
    User ||--o{ Room : "joins"
    
    User {
        ObjectId _id PK
        String name
        String email UK
        String password
        String avatar
        Object editor "Preferences (Theme, Font)"
    }

    Room {
        ObjectId _id PK
        String roomid "Unique Rule ID"
        String name
        String code "Current Source Code"
        String language
        ObjectId owner FK
        Date createdAt
    }
```

## 3. Real-time Collaboration (Sequence Diagram)
How code changes are synchronized between multiple users using Differential Synchronization.

```mermaid
sequenceDiagram
    autonumber
    participant Alice as User A
    participant Server as Socket.io Server
    participant Bob as User B
    
    Note over Alice, Bob: both joined 'room-1'

    Alice->>Alice: Edits Code
    Alice->>Server: emit('update', { roomid, patch })
    
    Note right of Server: Server applies patch to room state
    Server-->>Bob: emit('update', { patch })
    
    Bob->>Bob: Apply patch to Editor
    
    Alice->>Server: emit('updateIO', { input, output })
    Server-->>Bob: emit('updateIO', { input, output })
    
    Note over Alice, Bob: Whiteboard Drawing
    Alice->>Server: emit('drawData', { x, y, color })
    Server-->>Bob: emit('drawData', { x, y, color })
```

## 4. Video Chat Signaling (State Diagram)
The signaling process to establish a P2P connection via the Socket.io server.

```mermaid
sequenceDiagram
    participant Initiator
    participant Server as Signaling Server
    participant Receiver
    
    Initiator->>Server: emit('start-video', { roomID })
    Server-->>Initiator: emit('allUsers', [ReceiverID])
    
    Note over Initiator: Create Peer (Initiator: true)
    Initiator->>Server: emit('sending video signal', { signal, to: Receiver })
    
    Server-->>Receiver: emit('new video user joined', { signal, from: Initiator })
    
    Note over Receiver: Create Peer (Initiator: false)
    Receiver->>Server: emit('returning video signal', { signal, to: Initiator })
    
    Server-->>Initiator: emit('sender receiving final signal', { signal })
    
    Note over Initiator, Receiver: Encrypted P2P Stream Established
```
