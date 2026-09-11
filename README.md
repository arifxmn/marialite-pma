# MariaLite-PMA: The Ultra Lightweight MySQL / MariaDB + phpMyAdmin Docker Setup

A hyper-optimized, single-container docker environment combining MariaDB (100% MySQL Compatible) and phpMyAdmin, engineered for absolute minimal resource consumption, maximum effeciency, ease-of-use and instant startup times.

## Target Audience

The project is mainly aimed at developers, testers, students and system administrators who need a frictionless, highly-optimized, low-overhead and already configured database management environment with a nice & easy integrated web interface.

- **Solo Developers & Students:** Looking for a hassle-free way to manage their mysql / mariadb databases with a fast, responsive GUI without bloating their local machines.
- **CI/CD Pipelines:** Require ephemeral, fast-booting database environments for automated testing.
- **Edge / IoT Developers:** Working on constrained hardware (e.g. Raspberry Pi, low-memory VPS) where every megabyte of RAM counts.
- **Rapid Prototypers:** Need to spin up a complete database environment in seconds to test schema designs.
- **or, Anyone:** Who want a carefully engineered, lightweight, fast, efficient and complete easy-to-use mysql setup.

## Why?

Standard LAMP/LEMP Docker stacks are notoriously bloated. A typical setup requires separate containers for Nginx, PHP-FPM and MariaDB, resulting in:

1. **High Resource Usage:** Idling at 300MB–500MB+ of RAM.
2. **Network Overhead:** Inter-container TCP communication adds latency and CPU overhead.
3. **Complexity:** Managing multiple `docker-compose` services, volumes, and networks for a simple development task.
4. **Slow Boot Times:** Process managers like `supervisord` and heavy PHP-FPM pools take seconds to initialize.

**The Motivation:** I wanted a "bare-metal" equivalent in Docker. A single, unified container that boots in under 2 seconds, idles at near-zero CPU and uses less than 30MB of RAM, without sacrificing the core functionality of a modern database GUI.

## Scopes

**Core Goals:**

- Deliver a fully functional MySQL / MariaDB and phpMyAdmin environment in a **single Docker container**.
- Achieve the absolute theoretical minimum for image size, idle RAM and CPU usage.
- Provide a zero-configuration experience via environment variables.

**In Scope:**

- Alpine Linux 3.24 with native, pre-compiled PHP 8.4.25 and MariaDB 11.8.8 packages.
- PHP's built-in development server (no Nginx/Apache/PHP-FPM).
- Unix socket communication between phpMyAdmin and MariaDB. No Networking/ TCP overhead or latency.
- Aggressive stripping of unused files, translations and documentation.

**Out of Scope:**

- **Production deployments** (Security features like InnoDB doublewrite are intentionally disabled for performance).
- Full application server capabilities (No Node.js, Python or custom app code hosting).
- High-availability or clustered database setups.

## High-Level Architecture

```text
+-------------------------------------------------------------+
|                 Single Docker Container                     |
|                                                             |
|  +-----------------------+       +-----------------------+  |
|  |      phpMyAdmin       |       |        MariaDB        |  |
|  |    (PHP 8.4 CLI)      |       |                       |  |
|  |                       |       |                       |  |
|  |  HTTP Port: 8080      |<=====>|  Unix Socket          |  |
|  |                       |       |  /run/mysqld/mysqld   |  |
|  +-----------------------+       |  TCP Port: 3306       |  |
|                                  +-----------------------+  |
|                                                             |
|  [ entrypoint.sh ]                                          |
|  - Initializes DB on first run                              |
|  - Spawns MariaDB & PHP as background PIDs                  |
|  - Uses native `wait` for 0.0% CPU process monitoring       |
|  - Traps SIGTERM for graceful dual-process shutdown         |
+-------------------------------------------------------------+
```

### Key Design Decisions

1. **Native Alpine Packages over Source Compilation:** Using `apk add php84-*` instead of compiling PHP from source shaves ~100MB off the base image.
2. **Unix Socket Routing:** phpMyAdmin connects to MariaDB via `/run/mysqld/mysqld.sock`, bypassing the TCP/IP stack entirely for faster queries and lower CPU usage.
3. **Zero-CPU Process Management:** Instead of a `sleep 1` polling loop or a heavy process manager, the entrypoint uses the native shell `wait` command. This puts the shell to sleep at the kernel level, resulting in **0.0% idle CPU**.
4. **Extreme DB Tuning:** The `my.cnf` restricts the InnoDB buffer pool to 16MB, disables binary logging and turns off heavy performance schemas.

## Progress to Date

- **Test-v1 (Multi-Container):** Initial proof-of-concept using standard official images. (Result: ~450MB image, ~150MB+ RAM).
- **Test-v2 (Single Container):** Combined services into one image using a custom entrypoint. (Result: ~300MB image, ~100MB RAM).
- **Init. Release: (Extreme Optimization - Current):** Migrated to native Alpine packages, English-only phpMyAdmin, and zero-CPU process monitoring.
  - _Measurable Results:_ **~90MB image**, **~25MB idle RAM**, **0.0% idle CPU**, and **<1.5s cold start time**.

## Value Proposition

| Feature            | Standard LAMP Stack                | Nano-DB-PMA (This Project)      |
| :----------------- | :--------------------------------- | :------------------------------ |
| **Architecture**   | 3+ Containers (Nginx, PHP-FPM, DB) | **1 Single Container**          |
| **Image Size**     | ~500MB - 800MB                     | **~90MB (Uncompressed)**        |
| **Idle RAM**       | ~200MB - 400MB                     | **~25MB - 30MB**                |
| **Idle CPU**       | ~1-5% (Process managers/polling)   | **0.0% (Kernel-level `wait`)**  |
| **Startup Time**   | 5 - 15 seconds                     | **< 1.5 seconds**               |
| **Internal Comms** | TCP/IP (Network overhead)          | **Unix Socket (Zero overhead)** |

**Novelty:** The implementation of a dual-process monitor using native shell `wait` and signal trapping is a unique approach that eliminates the need for `supervisord` or `s6-overlay`, saving both image size and runtime memory.

## Getting Started

### Prerequisites

- [Docker](https://www.docker.com/get-started) and [Docker Compose](https://docs.docker.com/compose/install/) installed.

### 1. Clone the Repository

```bash
git clone https://github.com/arifxmn/marialite-pma.git
cd marialite-pma
```

### 2. Configure Environment

Create a `.env` file or modify the `docker-compose.yml` directly:

```env
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=dev_db
```

### 3. Up and Run

```bash
docker compose up -d
```

### 4. Access the Environment

- **phpMyAdmin GUI:** Open your browser and navigate to `http://localhost:8080`
- **Direct DB Connection:** Connect your local DB client to `localhost:3306`
- **Credentials:** Use `root` both for username and password.

### Expected Outcome

The container will start in under 2 seconds. You will see a minimal, english-only phpMyAdmin interface. The database will be initialized with your specified user and database.
