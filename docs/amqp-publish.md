# AMQP Publisher CLI

**Ultra-reliable, high-performance tool for publishing messages to RabbitMQ.**

This CLI tool (`amqp-publish`) is designed for stability and data safety. It allows you to stream massive datasets into RabbitMQ without consuming memory, while automatically handling network blips, server restarts, and back-pressure.

## ✨ Features

*   **🛡 Bulletproof Resilience:**
    *   **Auto-Reconnect:** Automatically pauses and reconnects if the RabbitMQ connection drops.
    *   **Publisher Confirms:** Waits for the server to acknowledge receipt of every message (ACK) before proceeding. No data loss on socket closures.
    *   **Smart Retries:** Exponential backoff strategy when the server is unreachable.
*   **⚡️ High Performance:**
    *   **Streaming Input:** Uses buffered I/O to read input files line-by-line. Can process multi-gigabyte files with minimal RAM usage.
    *   **Low Overhead:** Written in pure Go.
*   **🔌 Flexible Delivery:**
    *   **Persistence:** Toggle between transient (fast) or persistent (durable) delivery modes.
    *   **Routing:** Supports direct exchange publishing or queue-direct routing.
*   **🛑 Graceful Shutdown:** Safely stops on `Ctrl+C` without corrupting the current in-flight message.

---

## 🛠 Prerequisites

1.  A running **RabbitMQ Server** (Local or Cloud).
2.  Network access to the AMQP port (usually `5672` or `5671` for TLS).

---

## 📦 Installation (Homebrew)

This tool is available via a private tap. You can install it easily on macOS or Linux.

```bash
# 1. Add the tap
brew tap yasithab/homebrew-taps

# 2. Install the tool
brew install amqp-publish
```

**To Update:**
```bash
brew upgrade amqp-publish
```

---

## 🎮 Usage Guide

### 1. Simple Publish (Single Message)

Send a quick "Hello World" message to verify connectivity.

```bash
amqp-publish \
  -uri="amqp://guest:guest@localhost:5672/" \
  -routing-key="my-queue" \
  -body="Hello World"
```

### 2. Bulk Loader (File Streaming)

Efficiently load thousands (or millions) of messages from a file. Each line in the file becomes one message.

```bash
amqp-publish \
  -uri="amqp://user:pass@host:5672/" \
  -exchange="orders-exchange" \
  -routing-key="new.orders" \
  -input-file="./large-dataset.txt"
```

### 3. Critical Data (Persistent Mode)

Use the `-persistent` flag to mark messages as durable. If RabbitMQ restarts, these messages will be saved to disk (assuming the queue is also durable).

```bash
amqp-publish \
  -uri="..." \
  -routing-key="critical-jobs" \
  -body="Must not lose this" \
  -persistent
```

---

## ⚙️ Configuration Flags

| Flag | Default | Description |
|------|---------|-------------|
| `-uri` | `amqp://...` | The AMQP connection string. |
| `-exchange` | `""` | The Exchange name (leave blank for default). |
| `-routing-key`| `""` | The Routing Key or Queue Name. |
| `-input-file` | `""` | Path to a file containing messages (one per line). |
| `-body` | `""` | A single text string to publish. |
| `-persistent` | `false` | Sets delivery mode to Persistent (2). Default is Transient (1). |
| `-timeout` | `5s` | Network timeout per publish attempt. |

---

## 🔍 Scenarios

**Scenario 1: Replaying Dead Letter Logs**
You have a log file of failed JSON objects and need to push them back into the processing queue.
```bash
amqp-publish -uri="..." -routing-key="processing-queue" -input-file="failed_requests.jsonl"
```

**Scenario 2: Stress Testing a Consumer**
Generate a file with 100,000 dummy lines and blast them at your consumer application to see how it handles load.
```bash
amqp-publish -uri="..." -routing-key="load-test" -input-file="100k_lines.txt"
```

---

## ❓ Troubleshooting

**Error: "connection refused"**
*   **Fix:** Check your `-uri`. Ensure the host is reachable and the port (usually 5672) is open.

**Error: "Precondition Failed"**
*   **Fix:** You might be trying to publish to an exchange that exists but with different properties (e.g., trying to publish to a `direct` exchange as `topic`).

**Stuck on "Attempt 1/10 failed..."**
*   **Cause:** The tool cannot reach the server or the server is blocking connections (Resource Alarm).
*   **Fix:** Check RabbitMQ logs for "Memory Alarm" or "Disk Alarm".
