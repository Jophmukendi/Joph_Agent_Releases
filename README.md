
# Joph Agent

**Version:** 1.0.0  
**Platform:** Linux  
**Application:** JophMonitor  
**Release Status:** Initial Stable Release

## Overview

Joph Agent is a lightweight Linux monitoring application designed to collect system, service, and Docker container information and transmit monitoring reports to a centralized JophMonitor server.

The agent operates as a background service managed by systemd. It communicates with JophMonitor through HTTPS APIs, retrieves monitoring policies, and stores monitoring reports locally using SQLite.

Joph Agent is designed for centralized infrastructure monitoring across multiple Linux servers.

### Key Features

- Automated installation on supported Linux distributions.
- Device enrollment with JophMonitor.
- Permanent device authentication.
- CPU, memory, and disk utilization monitoring.
- Linux service monitoring.
- Docker container monitoring.
- Centralized monitoring policy synchronization.
- Local SQLite database for monitoring reports.
- HTTPS communication with JophMonitor.
- Background execution using systemd.
- Command-line administration.
- SHA-256 verification of release packages.

---

## 1. System Requirements

### 1.1 Supported Environment

Joph Agent is designed for Linux systems that use systemd.

The installer supports the following package managers:

| Package Manager | Linux Distribution Examples |
|---|---|
| APT | Ubuntu, Debian |
| DNF | Fedora, RHEL-compatible distributions |
| Pacman | Arch Linux |

**Note:** Package-manager support does not guarantee compatibility with every distribution or version. Verify the installation on the target system.

### 1.2 Required Components

- Linux operating system.
- systemd service manager.
- Python 3 and the required Python dependencies.
- Network connectivity to a JophMonitor server.
- HTTPS access to the configured JophMonitor API.
- Root or sudo privileges for installation.

Docker is optional and is required only for container monitoring.

### 1.3 Network Requirements

The agent must be able to establish outbound HTTPS connections to the configured JophMonitor server.

Default HTTPS port:

```text
TCP 443
```

Example server:

```text
https://monitor.jophnex.com
```

The JophMonitor server must expose a compatible API.

---

## 2. Architecture

Joph Agent uses a centralized monitoring architecture.

```text
+--------------------------------------+
|             Linux Server             |
|                                      |
|  +--------------------------------+  |
|  |          Joph Agent            |  |
|  |                                |  |
|  |  - System Monitoring           |  |
|  |  - Service Monitoring          |  |
|  |  - Container Monitoring        |  |
|  |  - Policy Synchronization      |  |
|  |  - Device Authentication       |  |
|  +---------------+----------------+  |
|                  |                   |
|          +-------v-------+           |
|          | SQLite DB     |           |
|          | agent.db      |           |
|          +---------------+           |
+------------------+-------------------+
                   |
                   | HTTPS / REST API
                   |
+------------------v-------------------+
|            JophMonitor               |
|                                      |
|  - Device Enrollment                 |
|  - Agent Authentication              |
|  - Monitoring Policies               |
|  - Monitoring Reports                |
|  - Centralized Data Storage          |
+--------------------------------------+
```

### 2.1 Agent Responsibilities

Joph Agent performs the following operations:

1. Establishes a connection with JophMonitor.
2. Enrolls the Linux device when required.
3. Authenticates using its saved device credentials.
4. Downloads monitoring policies.
5. Collects monitoring information.
6. Stores reports in the local SQLite database.
7. Transmits monitoring information to JophMonitor.
8. Repeats monitoring operations according to configured policies.

---

## 3. Installation

### 3.1 Download the Installer

Download the installation script from the public GitHub release.

```bash
curl -fL -o install.sh \
  https://github.com/Jophmukendi/Joph_Agent_Releases/releases/download/v1.0.0/install.sh
```

### 3.2 Run the Installer

Execute the installation script with root privileges.

```bash
sudo bash install.sh \
  --server-url https://monitor.jophnex.com
```

Replace the server URL with the address of your JophMonitor instance when applicable.

### 3.3 Installation Process

The installer performs the following operations:

1. Detects the Linux distribution and package manager.
2. Installs required system dependencies.
3. Downloads the Joph Agent Python wheel.
4. Verifies the downloaded package using SHA-256.
5. Creates the dedicated `joph-agent` system account.
6. Creates the Python virtual environment.
7. Installs the Joph Agent package.
8. Creates the persistent application data directory.
9. Validates the JophMonitor server connection.
10. Saves the server configuration.
11. Installs the systemd service.
12. Enables and starts Joph Agent.

### 3.4 Installation Directories

| Path | Description |
|---|---|
| `/opt/joph-agent/` | Application installation directory |
| `/opt/joph-agent/venv/` | Python virtual environment |
| `/var/lib/joph-agent/` | Persistent application data |
| `/var/lib/joph-agent/agent.db` | Local SQLite database |
| `/etc/systemd/system/joph-agent.service` | systemd service definition |
| `/usr/local/bin/joph-agent` | Administrative command wrapper |

**Important:** The application data directory contains operational configuration and potentially sensitive credentials. Access should be restricted to authorized accounts.

---

## 4. Verify the Installation

After installation, confirm that Joph Agent is running and connected to JophMonitor.

### 4.1 Display Agent Information

```bash
joph-agent show
```

Example output:

```text
Joph Agent - General Information

Agent Version:       1.0.0
Device ID:           3
Hostname:            linux-server
Operating System:    Linux
JophMonitor URL:     https://monitor.jophnex.com
Enrollment:          Enrolled
Service Status:      active
```

The Device ID is assigned by JophMonitor during enrollment.

### 4.2 Check the Service

```bash
sudo systemctl status joph-agent --no-pager
```

Expected service state:

```text
Active: active (running)
```

### 4.3 Verify Server Connectivity

```bash
curl -i https://monitor.jophnex.com/agent/health/
```

Expected response:

```json
{
  "application": "JophMonitor",
  "status": "ok",
  "api_version": "1.0.0"
}
```

A successful response confirms that the JophMonitor health endpoint is reachable.

It does not independently verify that the agent has enrolled or authenticated.

### 4.4 Check Agent Logs

```bash
sudo journalctl -u joph-agent -n 50 --no-pager
```

Look for messages indicating:

- Successful device enrollment.
- Successful authentication.
- Monitoring policy synchronization.
- Monitoring service startup.
- Local report creation.

---

## 5. Command-Line Administration

Joph Agent provides commands for inspecting and configuring the installed application.

### 5.1 Available Commands

| Command | Description |
|---|---|
| `joph-agent show` | Display general agent information |
| `joph-agent configure` | Configure the JophMonitor server |
| `joph-agent check_database` | Inspect pending local monitoring reports |
| `joph-agent --help` | Display available commands |

The `run` command is used to start the monitoring application and is normally invoked by systemd.

### 5.2 Display Agent Information

```bash
joph-agent show
```

This command displays information such as:

- Agent version.
- Device ID.
- Hostname.
- Operating system.
- JophMonitor URL.
- Enrollment status.
- Service status.

Sensitive authentication credentials are not displayed.

### 5.3 Configure the Server

```bash
joph-agent configure
```

This command allows an administrator to review or change the configured JophMonitor server.

Before changing the server, ensure that the destination server is reachable and compatible.

**Warning:** Changing the JophMonitor server may require the agent to establish a new device enrollment. Follow the command's confirmation prompts and preserve any pending reports before changing server identity.

### 5.4 Check the Local Database

```bash
joph-agent check_database
```

This command checks whether the local SQLite database contains monitoring reports that have not yet been sent to JophMonitor.

The command is intended for read-only diagnostics.

### 5.5 Display Help

```bash
joph-agent --help
```

Use this command to display the supported commands for the installed version.

---

## 6. Service Management

Joph Agent runs as a Linux systemd service.

### 6.1 Start the Agent

```bash
sudo systemctl start joph-agent
```

### 6.2 Stop the Agent

```bash
sudo systemctl stop joph-agent
```

### 6.3 Restart the Agent

```bash
sudo systemctl restart joph-agent
```

### 6.4 Check Service Status

```bash
sudo systemctl status joph-agent --no-pager
```

### 6.5 Enable Automatic Startup

```bash
sudo systemctl enable joph-agent
```

### 6.6 Disable Automatic Startup

```bash
sudo systemctl disable joph-agent
```

Disabling the service prevents automatic startup but does not uninstall Joph Agent.

### 6.7 View Real-Time Logs

```bash
sudo journalctl -u joph-agent -f
```

Press `Ctrl+C` to stop following the logs.

---

## 7. Device Enrollment and Authentication

### 7.1 Enrollment

When Joph Agent starts without an existing device enrollment, it requests a one-time enrollment token from JophMonitor.

The agent then submits the required device information to the server.

The server validates the enrollment request and assigns a Device ID.

The agent saves its enrollment credentials locally for subsequent authentication.

### 7.2 Permanent Authentication

After successful enrollment, Joph Agent uses its saved device credentials to authenticate with JophMonitor.

This avoids requesting a new enrollment token during every normal startup.

### 7.3 Enrollment API

The enrollment process uses the following JophMonitor endpoints:

```text
POST /agent/request-token/
POST /agent/enroll/
```

The token endpoint returns an enrollment token when the request succeeds.

### 7.4 Authentication Security

Joph Agent uses HTTPS for communication with JophMonitor.

Administrators should:

- Use a valid TLS certificate.
- Protect the application data directory.
- Avoid exposing authentication credentials in logs.
- Restrict access to the agent's local files.
- Avoid sharing enrollment tokens.

---

## 8. Monitoring Features

### 8.1 System Monitoring

Joph Agent collects system utilization information, including:

- CPU utilization.
- Memory utilization.
- Disk utilization.

Monitoring intervals are determined by the applicable monitoring policy.

### 8.2 Service Monitoring

Joph Agent monitors configured Linux services according to the service policies received from JophMonitor.

The service monitoring component determines the status of configured services and reports the collected information.

### 8.3 Docker Container Monitoring

Joph Agent supports Docker container monitoring.

Container information can include:

- Container identification.
- Container name.
- Container state.
- CPU utilization.
- Memory utilization.
- Memory usage.

**Important:** The agent's Linux service account must have appropriate access to the Docker API.

Access to `/var/run/docker.sock` is security-sensitive. Membership in the Docker group can grant root-equivalent privileges on the host and should not be enabled without evaluating the security implications.

### 8.4 Monitoring Policies

JophMonitor centrally defines monitoring policies.

Joph Agent periodically requests policy information from the server and uses it to control monitoring behavior.

The monitoring policy API includes:

```text
POST /agent/policy/
POST /agent/policy/services/
```

### 8.5 Monitoring Reports

The agent transmits monitoring information through the corresponding JophMonitor API endpoints.

```text
POST /agent/report/system/
POST /agent/report/containers/
```

Service reporting is managed by the service monitoring component.

---

## 9. Local Data Storage

Joph Agent uses SQLite to store monitoring information locally.

Default database location:

```text
/var/lib/joph-agent/agent.db
```

Local storage supports report buffering and allows the agent to retain monitoring information during interruptions in server connectivity.

### 9.1 Check Pending Reports

```bash
joph-agent check_database
```

Use this command to determine whether unsent reports remain in the local database.

### 9.2 Protect Local Data

Do not manually delete the SQLite database while the agent is running.

Deleting application data can remove:

- Pending monitoring reports.
- Local application state.
- Other persistent information required by the agent.

Before reinstalling or modifying the application, preserve the existing data directory.

---

## 10. Troubleshooting

### 10.1 Agent Service Is Inactive

**Symptom**

```text
Service Status: inactive
```

**Diagnosis**

Check the service:

```bash
sudo systemctl status joph-agent --no-pager
```

Inspect recent logs:

```bash
sudo journalctl -u joph-agent -n 50 --no-pager
```

**Resolution**

If the underlying error has been resolved, restart the service:

```bash
sudo systemctl restart joph-agent
```

Verify its status again.

### 10.2 JophMonitor Connection Timeout

**Symptom**

```text
HTTPSConnectionPool:
Read timed out
```

**Possible Causes**

- JophMonitor is unavailable.
- DNS resolves to an unreachable address.
- A firewall blocks the connection.
- The reverse proxy is not responding.
- The application server is experiencing delays.

**Diagnosis**

Check DNS resolution:

```bash
getent hosts monitor.jophnex.com
```

Check the health endpoint:

```bash
curl -v --connect-timeout 5 --max-time 20 \
  https://monitor.jophnex.com/agent/health/
```

**Resolution**

Verify that the server is reachable and that its HTTPS endpoint responds correctly.

If the Linux host and JophMonitor are on the same private network, verify that local DNS and routing are configured appropriately.

### 10.3 Enrollment Token Request Fails

**Symptom**

```text
Enrollment token request failed.
```

**Diagnosis**

Test the token endpoint:

```bash
curl -i --connect-timeout 5 --max-time 20 \
  -X POST \
  https://monitor.jophnex.com/agent/request-token/
```

A successful response should include:

```json
{
  "action_result": 1,
  "token": "generated-token"
}
```

The token value is generated by the server and must not be treated as a fixed value.

**Resolution**

If the endpoint fails, investigate the JophMonitor application logs and database connectivity.

If the endpoint works, inspect the agent logs and retry the service.

### 10.4 Docker Permission Denied

**Symptom**

```text
permission denied while trying to connect
to the Docker API
```

**Cause**

The account running Joph Agent cannot access the Docker socket.

**Diagnosis**

```bash
ls -l /var/run/docker.sock
```

Check the agent account:

```bash
id joph-agent
```

**Resolution**

Configure an appropriately restricted Docker monitoring access method.

Do not automatically grant unrestricted Docker access to the agent account.

### 10.5 Agent Is Not Enrolled

**Symptom**

```text
Device ID: Not enrolled
Enrollment: Not enrolled
```

**Diagnosis**

Check the JophMonitor endpoint:

```bash
curl -i https://monitor.jophnex.com/agent/health/
```

Inspect enrollment errors:

```bash
sudo journalctl -u joph-agent -n 100 --no-pager
```

**Resolution**

Correct connectivity or enrollment errors before restarting the service.

```bash
sudo systemctl restart joph-agent
```

Verify enrollment:

```bash
joph-agent show
```

### 10.6 Reports Are Not Reaching JophMonitor

**Diagnosis**

Check pending reports:

```bash
joph-agent check_database
```

Inspect agent logs:

```bash
sudo journalctl -u joph-agent -n 100 --no-pager
```

Verify server connectivity:

```bash
curl -i https://monitor.jophnex.com/agent/health/
```

**Resolution**

Confirm that:

1. The agent is enrolled.
2. Device authentication succeeds.
3. The server is reachable.
4. Monitoring policies are available.
5. The report API is responding correctly.

Do not delete pending reports to resolve synchronization errors.

---

## 11. Release Files

Joph Agent v1.0.0 is distributed through the public GitHub release repository.

**Release Repository:**  
https://github.com/Jophmukendi/Joph_Agent_Releases

**Release Tag:** `v1.0.0`

### 11.1 Release Assets

| File | Description |
|---|---|
| `install.sh` | Universal Linux installation script |
| `joph_agent-1.0.0-py3-none-any.whl` | Python distribution package |
| `joph-agent-wrapper.sh` | Command-line wrapper |
| `SHA256SUMS` | SHA-256 checksums for release verification |

### 11.2 Package Integrity

The installer verifies the downloaded package against the published SHA-256 checksums.

Checksum verification detects unexpected changes to downloaded files.

For stronger release authenticity guarantees, signed release artifacts may be introduced in future versions.

---

## 12. Security Considerations

Joph Agent is designed to operate using a dedicated Linux service account.

Recommended security practices include:

1. Restrict access to persistent agent data.
2. Use HTTPS with valid TLS certificates.
3. Protect enrollment and authentication credentials.
4. Apply the principle of least privilege.
5. Review Docker API permissions.
6. Verify release artifacts before installation.
7. Avoid exposing credentials through administrative commands or logs.

**Distribution Notice:** Python wheel packages can contain inspectable Python bytecode. A private source repository does not make distributed Python application logic impossible to inspect.

---

## 13. Version Information

| Property | Value |
|---|---|
| Product | Joph Agent |
| Version | 1.0.0 |
| Monitoring Server | JophMonitor |
| API Compatibility | JophMonitor API 1.0.0 |
| Operating System | Linux |
| Service Manager | systemd |
| Local Database | SQLite |
| Communication | HTTPS |
| Distribution | GitHub Releases |

---

## 14. Future Improvements

The following capabilities are being considered for future releases:

- Automatic enrollment retry.
- Improved systemd recovery behavior.
- More reliable offline report synchronization.
- Advanced agent health diagnostics.
- Restricted Docker monitoring access.
- Improved installation validation.
- Safe upgrade and rollback procedures.
- Automatic application updates.

These capabilities are planned improvements and are not guaranteed to be available in version 1.0.0.

---

## 15. License

Refer to the repository's `LICENSE` file for applicable license terms, if provided.

## 16. Project Links

**Public Releases:**  
https://github.com/Jophmukendi/Joph_Agent_Releases

**JophMonitor:**  
https://monitor.jophnex.com

**Joph Nexus:**  
https://jophnex.com

---

**Joph Agent v1.0.0 — Linux System, Service, and Container Monitoring**
