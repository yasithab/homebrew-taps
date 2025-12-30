# AWS SSM Tool

**Securely connect to private AWS resources without SSH keys, public IPs, or VPNs.**

This CLI tool creates a unified interface over **AWS Systems Manager (SSM)**. It automatically handles the complex logic of tunneling through a Bastion Host to reach private databases, caches, load balancers, and containers.

## ✨ Features

*   **🔒 Secure Port Forwarding:** Create tunnels from `localhost` to private AWS resources via a Bastion Host.
    *   **RDS** (Aurora, MySQL, Postgres, MariaDB)
    *   **Internal Load Balancers** (ALB / NLB) — *Automatically filters out public LBs*
    *   **ElastiCache** (Redis, Valkey, Memcached)
    *   **OpenSearch** (Elasticsearch)
    *   **DocumentDB** (MongoDB compatible)
    *   **Redshift** (Data Warehousing)
    *   **EC2** (Private Web Servers, Admin Panels)
    *   **ECS** (Direct tunnel to Fargate/ECS Task private IPs)
*   **💻 Interactive Shell (`exec`):** Instant shell access to EC2 instances or ECS Containers.
*   **🧠 Smart Discovery:**
    *   Auto-resolves Bastion Hosts via Tags or Environment Variables.
    *   Fuzzy search for all resources (type "mongo" to find "docdb", "redis" to find "elasticache").
*   **⚡️ Zero Config Required:** Falls back to interactive menus if no configuration is present.

---

## 🛠 Prerequisites

1.  **AWS CLI v2** configured with valid credentials (`aws configure` or `aws sso login`).
2.  **Session Manager Plugin** installed. ([Install Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html))

---

## 📦 Installation (Homebrew)

This tool is available via a private tap. You can install it easily on macOS or Linux.

```bash
# 1. Add the tap
brew tap yasithab/homebrew-taps

# 2. Install the tool
brew install ssm
```

**To Update:**
```bash
brew upgrade ssm
```

---

## ⚙️ Configuration (Optional)

The tool requires a **Bastion Host** (Jump Host) to create tunnels. It determines which Bastion to use in this order:

### 1. Environment Variables (Fastest)
Add these to your `~/.zshrc` or `~/.bashrc` to skip the detection step.

```bash
# Hardcode specific EC2 ID
export SSM_BASTION_EC2_INSTANCE="i-0123456789abcdef0"
```

### 2. Tag Discovery (Flexible)
Automatically find the bastion based on AWS Tags.

```bash
# Find EC2 where Tag 'Name' is 'BastionHost'
export SSM_BASTION_TAGS="Name=BastionHost" 

# OR, if using an ECS Fargate Bastion:
export SSM_BASTION_TYPE="ecs"
export SSM_BASTION_TAGS="Service=MyBastionService"
```

### 3. Manual Mode (Default)
If no variables are set, the tool will simply ask you to select a running EC2 instance to use as the Bastion from a list.

---

## 🎮 Usage Guide

### 1. Connect (Port Forwarding)

This creates a tunnel from your machine to a remote resource.

**Interactive Mode:**
Just run `ssm connect`. The tool will guide you through:
1.  Selecting the Bastion (if not configured).
2.  Selecting the Service Type (`rds`, `alb`, `redis`, etc.).
3.  Selecting the specific Resource.
4.  Choosing Local/Remote ports.

```bash
ssm connect
```

**Shortcut Mode:**
Skip the service menu by providing the service name argument.

```bash
# Internal Load Balancers
ssm connect loadbalancer

# Databases
ssm connect rds
ssm connect docdb
ssm connect redshift

# Caching & Search
ssm connect elasticache
ssm connect opensearch

# Compute
ssm connect ec2   # Tunnel to a private port (e.g., 8080)
ssm connect ecs   # Tunnel to a specific container port
```

---

### 2. Exec (Shell Access)

Open an interactive shell (`/bin/sh` or `bash`) inside a remote instance or container.

**Interactive Mode:**
```bash
ssm exec
# ? Select Target Type:
#   ec2
#   ecs
```

**Shortcut Mode:**
```bash
# Shell into an EC2 Instance
ssm exec ec2

# Shell into an ECS Fargate Task
ssm exec ecs
```

---

## 🔍 Examples

**Scenario 1: Accessing an Internal Jenkins behind an ALB**
```bash
ssm connect loadbalancer
# 1. Selects 'internal-jenkins-alb'
# 2. Asks for Remote Port (80)
# 3. Asks for Local Port (8080)
# Result: Open localhost:8080 in your browser.
```

**Scenario 2: Debugging a Go Application in Fargate**
```bash
ssm exec ecs
# 1. Selects Cluster -> Service -> Task -> Container
# Result: You are now inside the container shell.
```

**Scenario 3: Connecting to Redis (ElastiCache)**
```bash
ssm connect elasticache
# 1. Selects Redis Cluster
# 2. Tunnels port 6379
# Result: Connect via redis-cli -h localhost -p 6379
```

---

## ❓ Troubleshooting

**Error: "SessionManagerPlugin is not found"**
*   **Fix:** You must install the [AWS Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html).

**Error: "TargetNotConnected"**
*   **Fix (EC2):** Ensure the instance has an IAM Role with `AmazonSSMManagedInstanceCore`.
*   **Fix (ECS):** Ensure the task definition has a Task Role with SSM permissions, and the Service has "Execute Command" enabled.

**Error: "InvalidParameters ... portNumber"**
*   **Fix:** Run `brew upgrade ssm` to ensure you are using the latest version.
