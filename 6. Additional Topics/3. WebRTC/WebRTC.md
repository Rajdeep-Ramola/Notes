# WebRTC — How It Works

WebRTC (Web Real-Time Communication) enables direct, peer-to-peer communication between devices over the internet. It operates over **UDP** for low-latency data transfer.

---

## Table of Contents

- [Discovering Your Public IP (ICE/TURN Servers)](#discovering-your-public-ip-iceturn-servers)
- [Establishing a Connection (Signaling)](#establishing-a-connection-signaling)
- [Drawbacks of WebRTC](#drawbacks-of-webrtc)
- [Solutions to Scalability](#solutions-to-scalability)
  - [MESH Topology](#mesh-topology)
  - [SFU (Selective Forwarding Unit)](#sfu-selective-forwarding-unit)

---

## Discovering Your Public IP (ICE/TURN Servers)

Before two devices can connect, each device needs to know its **publicly assigned IP address** — not just the local one behind a router.

**How it works:**
1. Your router sends a request to a **TURN/ICE server**.
2. The TURN server detects the public IP from which the request originated.
3. It sends that public IP back to your device.

Your device now knows its public-facing IP address and can share it with others.

---

## Establishing a Connection (Signaling)

Once each device knows its public IP, they need a way to **exchange that information** with each other. This is done through a process called **signaling**.

**How it works:**
1. An intermediate server (e.g., a Node.js server) is set up.
2. Devices connect to this server using a protocol like **WebSockets**.
3. The devices exchange their **Session Description Protocol (SDP)** — metadata describing their capabilities and connection details.
4. Once both sides have each other's SDP, a **direct P2P connection** is established.
5. The intermediate signaling server is no longer needed after this point.

---

## Drawbacks of WebRTC

WebRTC is inherently a **Peer-to-Peer (P2P)** architecture, which introduces limitations:

- Only **two devices** can be directly connected in a standard WebRTC session.
- A third device **cannot simply join** an existing connection.

---

## Solutions to Scalability

### MESH Topology

**Idea:** Every device forms a direct P2P connection with every other device.

**Problem:** As more devices join, the number of connections grows exponentially. This quickly becomes too resource-intensive to be practical.

```
Device A ── Device B
   │    ╲  ╱    │
   │     ╲╱     │
   │     ╱╲     │
   │    ╱  ╲    │
Device C ── Device D
```

---

### SFU (Selective Forwarding Unit)

**Used by:** Discord, and many other real-time communication platforms.

**Idea:** Instead of devices connecting directly to each other, all devices connect to a **central virtual machine (SFU server)**.

**How it works:**
1. A virtual machine runs on a server.
2. Every device forms a **P2P connection with this virtual machine** (not with each other).
3. The SFU receives streams from all connected devices.
4. It **combines and forwards** those streams to all other devices as a single broadcast.

**Why it's better:**
- Devices only need to maintain **one connection** (to the SFU), regardless of how many others are in the session.
- Far more scalable than MESH for larger groups.

```
Device A ──┐
Device B ──┼──► SFU Server ──► Broadcasts to all devices
Device C ──┘
```

---

## Summary

| Concept | Purpose |
|---|---|
| UDP | Transport protocol WebRTC uses for speed |
| ICE/TURN Server | Helps a device discover its public IP |
| Signaling Server | Facilitates initial SDP exchange between devices |
| SDP | Session description containing connection metadata |
| P2P | Direct connection between exactly two devices |
| MESH Topology | All devices connect to each other — doesn't scale well |
| SFU | Central server that receives and forwards streams — scalable |
