Requires knowledge of Docker, IP mapping, and storage configuration. Not a plug-and-play setup.


# 🧩 AnythingLLM MCP HTTP Bridge  

**Lightweight HTTP bridge for AnythingLLM to connect multiple local MCP servers securely via Docker.**  
A clean, modular, and safe way to run multiple MCP servers without Docker-in-Docker or direct process spawning.  

![Architecture Diagram](https://user-images.githubusercontent.com/0000000/placeholder-diagram.png)

---

## 🚀 Overview  

This bridge allows **AnythingLLM** to communicate with multiple **MCP servers** inside isolated Docker containers.  
It provides a single, secure HTTP entry point while keeping each MCP fully sandboxed.  

### ✅ Key Features  
- 🔒 **Safe & clean:** No host or Docker-in-Docker access.  
- 🧱 **Modular:** Each MCP runs in its own container with a small FastAPI server.  
- 🌐 **Bridge-controlled:** AnythingLLM only talks to the bridge (`mini-bridge`), never to the MCPs directly.  
- ⚙️ **Dynamic registry:** MCP servers are auto-loaded from `mcp_registry.json`.  
- ♻️ **Auto reload:** The bridge reloads configuration changes live without restarting containers.  
- 🕒 **Example MCPs:** Includes a `Dummy-MCP` for testing and a working `MCP-Time` module returning system time.  

---

## 🧠 Architecture  

```
───────────────────────────────
Docker Network: test-net
───────────────────────────────
│
├─ anythingllm  (Port 3001)
│     ↳ Main app – talks only to mini-bridge
│
├─ mini-bridge  (Port 4100)
│     ↳ Forwards JSON-RPC requests to registered MCPs
│     ↳ Reloads mcp_registry.json dynamically
│
├─ dummy-mcp    (Port 4200)
│     ↳ JSON-RPC test MCP returning handshake info
│
└─ mcp-time     (Port 4210)
      ↳ Returns current UTC time
───────────────────────────────
```

---

## ⚙️ Quick Setup  

```bash
git clone https://github.com/yourname/anythingllm-mcp-http-bridge.git
cd anythingllm-mcp-http-bridge
docker compose up --build
```

Bridge log output:  
```
[Bridge] Registry reloaded: ['dummy', 'time']
[Bridge] Default route → dummy (200)
```

Then open AnythingLLM → MCP Settings →  
Add new server: `http://mini-bridge:4100`

---

## 📜 MCP Registry Example  

`mini_bridge/config/mcp_registry.json`

```json
{
  "autoReload": true,
  "servers": [
    {
      "id": "dummy",
      "name": "Dummy MCP",
      "url": "http://dummy-mcp:4200",
      "type": "streamable",
      "enabled": true
    },
    {
      "id": "time",
      "name": "Time MCP",
      "url": "http://mcp-time:4210",
      "type": "streamable",
      "enabled": true
    }
  ]
}
```

---

## 🧩 Available MCP Modules  

| MCP | Description | Example call |
|-----|--------------|--------------|
| **Dummy-MCP** | Minimal test server, responds to `initialize`, `ping`, etc. | `POST /dummy` |
| **MCP-Time** | Returns current UTC time | `tools/call → get_time` |

Test manually:  
```bash
curl -X POST http://localhost:4100/time      -H "Content-Type: application/json"      -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_time"}}'
```

Response:  
```json
{"jsonrpc":"2.0","id":1,"result":{"message":"The current time is 18:42:10 UTC"}}
```

---

## 🧩 Folder Structure  

```
├── docker-compose.yml
├── mini_bridge/
│   ├── mini_bridge_v2.py
│   ├── Dockerfile
│   └── config/
│       └── mcp_registry.json
├── dummy_MCP/
│   ├── dummy_mcp.py
│   └── Dockerfile
└── mcp_time/
    ├── mcp_time.py
    └── Dockerfile
```

---

## 🔒 Security  

Each MCP server runs in its **own container** on a private Docker network.  
The bridge is the **only exposed interface** and accepts **JSON-RPC over HTTP** — nothing else.  

- No direct container access from AnythingLLM  
- No Docker-in-Docker  
- No shell or file system commands  
- Safe, stateless JSON-based communication only  

---

## 🧾 License  

**MIT License**  
Feel free to use, fork, and build upon this project.  
See the full [LICENSE](./LICENSE) file for details.  

---

## 💬 Development Note  

I’ll keep improving this project as my own independent solution.  
If the **AnythingLLM** team finds the idea useful, feel free to build on it or integrate parts of it.  

As of now, tool output is still returned in a raw format instead of a fully formatted chat reply — once that’s refined, the public release will follow.  

---

## 🌟 Credits  

Created by **Danny**  
Built with ❤️ using FastAPI, Docker, and curiosity.
