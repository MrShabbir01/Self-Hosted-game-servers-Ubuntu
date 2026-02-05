# Architecture Overview

In this document, I describe the design, deployment, and operation of
self-hosted dedicated game servers running on Ubuntu Linux under a Proxmox
virtualized environment.

The goal of this setup is to provide stable, secure, and maintainable hosting
for multiple Steam-based dedicated game servers using a common operational
pattern.

## Environment Summary
- Hypervisor: Proxmox VE
- Guest OS: Ubuntu (Desktop edition, configured for server-style operation)
- Hosting Model: One Linux VM hosting multiple game servers
- Game Platform: SteamCMD
- Firewall: UFW (default deny)
- Service Model: User-level execution with startup scripts

## Design Decisions

### Operating System Choice
Ubuntu was selected due to its long-term stability, predictable update cycle, and extensive community support. At the time of deployment, Ubuntu Desktop was chosen instead of Ubuntu Server due to familiarity. Non-essential services were disabled post-installation, and the system was configured to behave as a headless server.
The overall design remains compatible with Ubuntu Server and can be migrated without any changes in the architecture.

### Security and User Isolation
Game servers do not run as root.
Each server runs under a dedicated non-privileged user account, ensuring that a compromise of one service does not grant full system access.
This approach limits blast radius and follows standard Linux hardening practices.

## Virtualization Layer
The Ubuntu game server runs as a virtual machine inside Proxmox. This provides:
•	Hardware isolation
•	Snapshot and backup capabilities
•	Resource control (CPU, RAM)
•	Simplified recovery in case of failure

The game server VM is treated as disposable infrastructure — rebuildable from documentation and configuration files.

## SteamCMD Deployment Model
SteamCMD is used for unattended installation and updates of all dedicated game servers.
Key characteristics of the SteamCMD setup:
•	Anonymous login
•	Explicit install directories per game
•	Script-based installation and updates
•	Centralized binary management
A symbolic link is created within the game user’s home directory to standardize paths and simplify maintenance.
This allows consistent management across different games while minimizing duplication.

## System Tuning and Resource Limits
Dedicated game servers are sensitive to system limits, especially file descriptors and network sockets. The following system-level tuning was applied:
•	Increased maximum open file descriptors (fs.file-max)
•	Adjusted soft and hard nofile limits via /etc/security/limits.conf
•	Runtime limits aligned using ulimit
These changes prevent startup failures, connection drops, and instability under concurrent player load.

## Network and Firewall Configuration
The system uses UFW with a default-deny posture.
Only required ports are explicitly opened:
•	Game-specific UDP ports (e.g., 7777, 7778)
•	Steam query and communication ports (e.g., 27015–27051)
No unnecessary services are exposed. Port rules are documented per game to avoid conflicts and ensure predictable connectivity.

## Game Server Structure
The repository follows a shared-pattern design:
•	Common configuration for all servers (SteamCMD, tuning, firewall concepts)
•	Game-specific directories for individual server differences
Each game directory contains:
•	Startup scripts (start_server.sh)
•	Configuration files (Game.ini, GameUserSettings.ini, etc.)
•	Notes on ports, launch flags, and known issues
Games such as Valheim and ARK: Survival Evolved differ in configuration, but share the same operational lifecycle.

## Service Startup and Management
Game servers are launched using controlled startup scripts rather than manual execution.
Startup scripts handle:
•	Environment preparation
•	Correct binary invocation
•	Logging behavior
•	Launch flags and server parameters
This ensures repeatability and reduces operator error during restarts and updates.

## Backup and Maintenance Strategy
•	Game data is backed up at the filesystem level
•	Server updates are performed via SteamCMD without OS reinstallation
•	Configuration files are version-controlled in Git
•	Logs are retained per server instance
The system is designed so that a failed VM can be rebuilt using documented steps and stored configurations.

## Operational Observations
•	Increased file descriptor limits significantly improved stability
•	Dedicated users simplified permissions and security management
•	Explicit firewall rules reduced exposure and troubleshooting complexity
•	Script-based startup reduced human error during restarts

## Known Limitations and Future Improvements
•	Migration to Ubuntu Server for reduced footprint
•	Conversion of startup scripts into systemd services
•	Automated update scheduling
•	Centralized monitoring and alerting
•	Evaluation of containerized game servers where applicable
