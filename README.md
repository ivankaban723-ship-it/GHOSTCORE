 GitHub Repository Presentation Pack for GHOSTCOREFile 1: README.md<div align="center">

# 🪩 GHOSTCORE

**Linux Process Observability Toolkit & Real-Time Universe Visualizer**

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![License](https://img.shields.io/badge/license-MIT-green.svg?style=for-the-badge)](LICENSE)
[![Build Status](https://img.shields.io/badge/CI-passing-brightgreen.svg?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/Mr-Ivan/GHOSTCORE/actions)
[![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg?style=for-the-badge)](https://github.com/psf/black)

*Observe your Linux OS like never before — as an interactive, live-simulated graph universe powered by high-performance event stream architecture.*

[Features](#-features) • [Installation](#-installation) • [Quick Start](#-quick-start) • [Architecture](#-architecture) • [Plugin API](#-plugin-api) • [Protocol Specification](#-gcp-protocol-v1)

</div>

---

## 🌟 Overview

**GHOSTCORE** turns raw Linux system metrics from `/proc` into a dynamic, physics-based graph topology. It provides modular process tracking, real-time socket broadcasting, historical SQLite storage, and a flexible Python plugin API for extended system auditing.

[ /procfs ] ──► ( ProcessMonitor ) ──► [ Engine Thread ]
│
┌──────────────────────────────────────┴──────────────────────────────────────┐
▼                                      ▼                                      ▼
( Interactive UI )                    ( SQLite Event Store )               ( Unix Stream Socket )
PySide6 + Force Simulation                Historical Metrics                      GCP JSON-L Stream
---

## ✨ Features

- 🌌 **Live Universe Graph:** Nodes rendered by process state, scaled logarithmically by RSS memory, and encircled by real-time CPU rings.
- ⚡ **Dual Layout Simulation:** Seamlessly toggle between a **Deterministic Radial Tree** and an interactive **Force-Directed Physics Simulation** (O(N²) optimized via NumPy).
- 👻 **Ghost Processes:** Exited processes linger momentarily as red fading "ghosts" for visual debug of short-lived child spawns.
- 🔍 **Deep Process Inspection:** View live CPU/RAM sparklines, ancestry parent-child tracing, open threads, system uptime, and on-demand local command line inspection.
- 🔌 **Extensible Plugin Framework:** Fault-isolated plugins with explicit permission gating and event-filtering capabilities.
- 📡 **GCP v1 Protocol:** High-throughput JSON-Lines protocol over Unix domain sockets with snapshot replay on client connection.
- 🖥️ **Headless CLI:** Execute top-like watch sessions, stream logs, record long-term data, and inspect historical databases without loading a GUI.

---

## 📦 Installation

### System Requirements
- **OS:** Linux (requires access to `/proc` filesystem)
- **Python:** `>= 3.10`

### Setup Environment

```bash
# Clone repository
git clone https://github.com/Mr-Ivan/GHOSTCORE.git
cd GHOSTCORE

# Create & activate virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies and local development binary
pip install --upgrade pip
pip install -r requirements.txt
pip install -e .
🚀 Quick Start1. Launch Interactive GUI# Standard UI start
ghostcore

# Launch with persistence DB, stream socket, and trusted custom plugins
ghostcore --db ~/.ghostcore/history.db --socket /tmp/ghostcore.sock --plugins-dir examples/plugins
Keyboard Shortcuts & ControlsShortcut / ControlDescriptionScroll WheelZoom in / Zoom out (clamped scale)Left Click + DragPan canvas or pin node position in force simulationClick Node / Top CPUInspect node stats, view sparklines & trace ancestry chainFFit graph into view areaRTrigger full force/radial re-layoutLToggle layout engine (Radial Tree ↔ Force Directed)Ctrl+FFocus filter input box (state:Z, process name, PID)Ctrl+PPause / Resume real-time updatesCtrl+EExport current system process state snapshot to GCP JSONL2. Headless CLI Engine# Live top-style terminal monitor
ghostcore-cli watch --top 15

# Record system events into SQLite and broadcast to a socket headlessly
ghostcore-cli record --db history.db --socket /tmp/ghostcore.sock

# Stream process events directly from active daemon socket
ghostcore-cli tail --socket /tmp/ghostcore.sock --type process.

# Query historical event logs recorded in the past 30 minutes
ghostcore-cli history --db history.db --since 30m --limit 500
🏗 ArchitectureFor in-depth technical documentation, see docs/ARCHITECTURE.md.   ProcfsCollector (Reads /proc)
          │
          ▼
   ProcessMonitor (Computes CPU % / Diff state)
          │
          ▼
    Engine.tick() ◄── PluginManager (Executes custom plugins)
          │
          ├───────────────┬──────────────────────┐
          ▼               ▼                      ▼
      EventBus       EventStore             GCPServer
     (Publish)    (SQLite History)     (Unix Domain Socket)
          │
          ▼
   EngineWorker (QThread Queue)
          │
          ▼
 ProcessGraphView (PySide6 Render Loop)
🔌 Plugin APIPlugins allow custom system monitoring (e.g., zombie process alerts, resource leak detectors). Plugins are isolated so runtime exceptions won't crash the core GUI or engine.Create a Python script inheriting from GhostPlugin:# examples/plugins/zombie_watch.py
from ghostcore.plugins.api import GhostPlugin
from ghostcore.core.protocol import GCPEvent

class ZombieWatch(GhostPlugin):
    name = "zombie-watch"
    version = "1.0.0"
    permissions = ("process.read",)

    def start(self) -> None:
        self._seen = set()

    def on_poll(self, result):
        zombies = {s.node_id: s for s in result.snapshots if s.state == "Z"}
        fresh = [s for node_id, s in zombies.items() if node_id not in self._seen]
        self._seen = set(zombies)

        return [
            GCPEvent(
                "alert.zombie",
                {"node_id": s.node_id, "pid": s.pid, "name": s.name},
                result.timestamp_ns
            )
            for s in fresh
        ]
Read more in docs/PLUGINS.md.📜 GCP Protocol v1The GhostCore Protocol (GCP) uses newline-delimited JSON Lines over Unix sockets:{
  "protocol": "GCP",
  "version": 1,
  "type": "process.sample",
  "timestamp_ns": 1700000000000000000,
  "payload": {
    "pid": 1337,
    "node_id": "1337:92810",
    "cpu_percent": 12.4,
    "memory_bytes": 104857600,
    "state": "R",
    "threads": 8
  }
}
Full details are documented in docs/GCP_PROTOCOL.md.🧪 Testing# Run unit tests
pytest -v

# Run with coverage report
pytest --cov=src/ghostcore tests/
🛡️ Security & Privacy NotesCommand Lines: Process arguments often contain sensitive tokens or passwords. Command lines are read on-demand for local UI display only and are never stored in SQLite or broadcast over socket streams.Socket Authorization: The GCP stream socket file is assigned strictly 0600 Linux permissions (accessible only by owner user).Plugins: Plugin discovery executes Python source code directly. Ensure --plugins-dir points only to trusted directories you control.⚖️ LicenseDistributed under the MIT License. See LICENSE file for details.
## File 2: `CONTRIBUTING.md`

```markdown
# Contributing to GHOSTCORE

Thank you for considering contributing to GHOSTCORE!

## Code Guidelines

1. **Format & Quality:** Keep standard PEP 8 compliance.
2. **Single-File Rules:** Core UI modules, event mechanisms, and engine handlers should keep clean boundary separation.
3. **Testing:** Write tests using `pytest` for all non-GUI components.
4. **Clean Commits:** Write clean, descriptive commit messages:
   - `feat: add memory sparkline ceiling clamp`
   - `fix: resolve race condition in Unix socket server shutdown`

## Pull Request Process

1. Fork the repository (`Mr-Ivan/GHOSTCORE`) and create your feature branch from `main`.
2. Run tests to confirm all checks pass (`pytest -v`).
3. Open a Pull Request referencing issue numbers where applicable.
