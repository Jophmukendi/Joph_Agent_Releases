
# Joph Agent

**Version:** 1.0.0  
**Platform:** Linux  
**Language:** Python  
**Monitoring Server:** JophMonitor  
**License:** See repository for license information

## 1. Overview

Joph Agent is a Python-based monitoring application that collects system, service, and Docker container information from Linux devices and sends monitoring reports to a centralized JophMonitor server.

The agent operates as a background service managed by systemd. It supports automatic device enrollment, secure authentication, centralized monitoring policies, and local report storage.

Joph Agent uses a client-server architecture. Each installed agent communicates with JophMonitor through HTTPS endpoints.

**Project Repository:**  
https://github.com/Jophmukendi/Joph_Agent

**Latest Release (1.0.0):**  
https://github.com/Jophmukendi/Joph_Agent/releases/tag/v1.0.0

---

## 2. Features

Joph Agent 1.0.0 provides the following capabilities.

### 2.1 Device Enrollment and Authentication

Joph Agent automatically enrolls a Linux device during its first successful startup.

The enrollment process performs the following operations:

1. Generates a permanent device credential and recovery secret.
2. Requests a one-time enrollment token from JophMonitor.
3. Collects device identification information.
4. Submits an enrollment request to JophMonitor.
5. Receives a unique device ID.
6. Saves the device ID and authentication credentials locally.
7. Authenticates the device with JophMonitor.
8. Retrieves the assigned monitoring policies.

After successful enrollment, the agent reuses its stored credentials during subsequent startups.

**Note:** Restarting an enrolled agent does not require a new enrollment unless its stored identity has been removed or reset.

### 2.2 System Monitoring

Joph Agent collects operating-system and hardware information, including:

- CPU utilization
- Memory utilization
- Disk utilization
- System uptime
- Hostname and operating-system information

System reports are collected according to the monitoring policy assigned by JophMonitor.

### 2.3 Service Monitoring

Joph Agent monitors Linux services defined in the service policies received from JophMonitor.

The agent checks the state of configured services and reports monitoring information to the server.

### 2.4 Docker Container Monitoring

Joph Agent supports Docker container monitoring.

The agent can retrieve container information such as:

- Container ID and name
- Container image
- Container state and status
- CPU utilization
- Memory utilization

**Requirement:** The agent must have permission to access the Docker Engine API.

Docker monitoring is optional. The agent can operate without Docker access, although container monitoring will not function.

### 2.5 Monitoring Policy Synchronization

JophMonitor centrally manages monitoring policies.

Joph Agent periodically retrieves updated policies from the server.

Policies control monitoring operations and collection intervals.

This design allows administrators to modify monitoring behavior without reinstalling the agent.

### 2.6 Local Report Storage

Joph Agent uses SQLite for local monitoring data storage.

The database file is located at:

```text
/var/lib/joph-agent/agent.db
```

Local storage supports temporary retention of monitoring reports before transmission to JophMonitor.

The `check_database` command provides information about locally stored system reports.

### 2.7 Background Execution

Joph Agent runs as a systemd service.

This provides:

- Background monitoring
- Automatic startup when enabled
- Standard Linux service management
- Service status inspection
- Centralized logging through journald

---

## 3. System Architecture

Joph Agent uses a client-server architecture.

```text
                    JophMonitor
                  Django Application
                         |
                         | HTTPS
                         |
                    Joph Agent
                         |
          +--------------+--------------+
          |              |              |
     System Monitor  Service Monitor  Docker Monitor
          |              |              |
          +--------------+--------------+
                         |
                   Local SQLite
                     agent.db
                         |
                  Report Delivery
                         |
                  JophMonitor API
                         |
                    MySQL Database
```

### 3.1 Component Responsibilities

| Component | Responsibility |
|---|---|
| JophAgent | Coordinates agent operations and lifecycle |
| Device | Retrieves device and system information |
| AgentConfigure | Manages the JophMonitor server URL |
| AgentCredential | Manages device authentication information |
| AgentPolicy | Stores monitoring policy settings |
| ServicePolicy | Manages service monitoring policies |
| PolicySync | Synchronizes monitoring policies |
| SystemReporter | Collects system monitoring information |
| ServiceReporter | Monitors configured services |
| ContainerReporter | Collects Docker container information |
| AgentDatabase | Manages local SQLite data |
| CLI | Provides administrative commands |
| systemd | Manages the agent background service |

---

## 4. System Requirements

Before installing Joph Agent, verify that the target Linux device meets the following requirements.

| Requirement | Description |
|---|---|
| Operating system | Compatible Linux distribution |
| Python | Python 3 with virtual environment support |
| Service manager | systemd |
| Package manager | Supported APT, DNF, or Pacman environment |
| Network | HTTPS access to JophMonitor |
| Privileges | sudo or root privileges for installation |
| Docker | Optional; required for container monitoring |

The installation script detects the Linux distribution and installs the required dependencies.

**Note:** Support for a package manager does not guarantee compatibility with every distribution or release. Test the installer on the target platform.

---

## 5. Installation

Joph Agent is distributed as a Python wheel through GitHub Releases.

The release includes the following files:

| File | Purpose |
|---|---|
| `install.sh` | Linux installation script |
| `joph_agent-1.0.0-py3-none-any.whl` | Python application package |
| `joph-agent-wrapper.sh` | Administrative command wrapper |
| `SHA256SUMS` | Release-file integrity verification |

### Step 1 — Download the Installer

Open a terminal on the target Linux device.

Execute:

```bash
curl -fL -o install.sh \
  https://github.com/Jophmukendi/Joph_Agent/releases/download/v1.0.0/install.sh
```

**Expected result:** The `install.sh` file is downloaded into the current directory.

### Step 2 — Install Joph Agent

Execute:

```bash
sudo bash install.sh \
  --server-url https://monitor.jophnex.com
```

Replace the URL with the address of your JophMonitor server when necessary.

The installer performs the following operations:

1. Detects the Linux distribution.
2. Installs required system dependencies.
3. Validates connectivity to JophMonitor.
4. Downloads the release files.
5. Verifies the downloaded files using SHA-256 checksums.
6. Creates the dedicated `joph-agent` system account.
7. Creates the application directory and Python virtual environment.
8. Installs the Python wheel and its dependencies.
9. Installs the administrative command wrapper.
10. Saves the JophMonitor server configuration.
11. Creates and enables the systemd service.
12. Starts Joph Agent.

### Step 3 — Verify Installation

After installation, the terminal should display messages similar to:

```text
[Joph Agent] Installing Python wheel
[Joph Agent] Saving JophMonitor configuration

Checking JophMonitor connection...
JophMonitor connection successful.
Configuration saved successfully.

[Joph Agent] Writing systemd service
[Joph Agent] Installed v1.0.0,
connected to https://monitor.jophnex.com
```

These messages indicate that installation completed successfully.

**Important:** Installation success does not necessarily mean device enrollment has completed. Verify enrollment using the procedures in Section 8.

---

## 6. Installation Directories

Joph Agent separates application files from persistent monitoring data.

| Directory or File | Description |
|---|---|
| `/opt/joph-agent/` | Application installation directory |
| `/opt/joph-agent/venv/` | Python virtual environment |
| `/var/lib/joph-agent/` | Persistent configuration and monitoring data |
| `/var/lib/joph-agent/agent.db` | Local SQLite database |
| `/var/lib/joph-agent/settings.json` | Agent configuration |
| `/usr/local/bin/joph-agent` | Administrative command wrapper |
| `/etc/systemd/system/joph-agent.service` | systemd service definition |

**Important:** The `/var/lib/joph-agent/` directory contains persistent agent information, including authentication credentials. Do not delete this directory during routine updates.

---

## 7. Command-Line Administration

Joph Agent provides a command-line interface for common administrative operations.

### 7.1 Display Available Commands

Execute:

```bash
joph-agent --help
```

**Expected output:**

```text
Usage: joph-agent {show|configure|check_database|--help}

show             Display general agent information
configure        Display or change JophMonitor URL
check_database   Inspect locally stored system reports

Service control:
sudo systemctl {start|stop|restart|status} joph-agent
```

### 7.2 Display Agent Information

Execute:

```bash
joph-agent show
```

**Example output:**

```text
Joph Agent - General Information

Agent Version:      1.0.0
Device ID:          3
Hostname:           jophnex
Operating System:   Linux
JophMonitor URL:    https://monitor.jophnex.com
Enrollment:         Enrolled
Service Status:     active
```

The command displays:

- Installed agent version
- Assigned device ID
- Hostname
- Operating-system information
- Configured JophMonitor URL
- Enrollment status
- systemd service status

The device ID is assigned by JophMonitor and varies between installations.

### 7.3 Configure JophMonitor

Execute:

```bash
joph-agent configure
```

This command displays or changes the configured JophMonitor server URL.

The configuration process validates the new server before saving changes.

**Important:** Changing the monitoring server may require a new device enrollment because authentication credentials are associated with the original server.

The installer uses the noninteractive configuration option:

```bash
joph-agent configure \
  --server-url https://monitor.jophnex.com
```

### 7.4 Inspect the Local Database

Execute:

```bash
joph-agent check_database
```

This command inspects the agent's local SQLite database.

It is intended to help administrators identify monitoring reports stored locally and determine whether reports remain to be synchronized.

The currently installed CLI describes this operation as inspecting locally stored system reports.

**Note:** Available report counts and synchronization details depend on the database schema and implementation in the installed version.

---

## 8. Post-Installation Verification

Perform the following checks after installation.

### Check 1 — Verify the Administrative CLI

Execute:

```bash
joph-agent --help
```

**Expected result:** The command displays the available administrative operations.

### Check 2 — Verify Device Enrollment

Execute:

```bash
joph-agent show
```

Confirm that the output includes:

```text
Device ID:          <assigned-device-id>
Enrollment:         Enrolled
Service Status:     active
```

If the device is not enrolled, review the troubleshooting procedures in Section 10.

### Check 3 — Verify the systemd Service

Execute:

```bash
sudo systemctl status joph-agent --no-pager
```

**Expected result:**

```text
Loaded: loaded (...; enabled)
Active: active (running)
```

The `active (running)` state indicates that the agent process is running.

### Check 4 — Verify JophMonitor Connectivity

Execute:

```bash
curl -i https://monitor.jophnex.com/agent/health/
```

**Expected HTTP status:**

```text
HTTP/1.1 200 OK
```

**Expected JSON response:**

```json
{
  "application": "JophMonitor",
  "status": "ok",
  "api_version": "1.0.0"
}
```

This confirms that the JophMonitor health endpoint is reachable.

It does not independently confirm device enrollment, authentication, or successful report delivery.

### Check 5 — Verify Agent Logs

Execute:

```bash
sudo journalctl -u joph-agent -n 30 --no-pager
```

Successful startup logs may include:

```text
Device enrolled and default policy created.
Agent authenticated successfully.
Monitoring policy saved successfully.
Service policies saved successfully.
Agent activated successfully.
Policy synchronization started.
System monitoring started.
Service monitoring thread started.
Container monitoring thread started.
System report saved locally.
```

The enrollment message is expected during initial enrollment. It is not required on every subsequent startup.

### Check 6 — Inspect Local Monitoring Reports

Execute:

```bash
joph-agent check_database
```

Review the output for locally stored monitoring information.

### Check 7 — Verify Requests on JophMonitor

On the Linux host running the JophMonitor Docker container, execute:

```bash
docker logs --tail 50 JophMonitor-dev
```

Look for successful HTTP requests:

```text
POST /agent/auth/ HTTP/1.1 200
POST /agent/policy/ HTTP/1.1 200
POST /agent/policy/services/ HTTP/1.1 200
POST /agent/report/system/ HTTP/1.1 200
POST /agent/report/containers/ HTTP/1.1 200
```

These entries indicate successful HTTP responses from JophMonitor.

To confirm that reports were stored, inspect the corresponding application records or database tables.

---

## 9. Managing the Joph Agent Service

Joph Agent uses systemd for service management.

### 9.1 Check Service Status

```bash
sudo systemctl status joph-agent --no-pager
```

### 9.2 Start the Service

```bash
sudo systemctl start joph-agent
```

### 9.3 Stop the Service

```bash
sudo systemctl stop joph-agent
```

### 9.4 Restart the Service

```bash
sudo systemctl restart joph-agent
```

### 9.5 Enable Automatic Startup

```bash
sudo systemctl enable joph-agent
```

### 9.6 Disable Automatic Startup

```bash
sudo systemctl disable joph-agent
```

Disabling the service prevents automatic startup but does not uninstall Joph Agent.

### 9.7 View Recent Logs

```bash
sudo journalctl -u joph-agent -n 50 --no-pager
```

### 9.8 Follow Logs in Real Time

```bash
sudo journalctl -u joph-agent -f
```

Press `Ctrl+C` to stop following the logs.

---

## 10. Troubleshooting

This section describes common problems, their possible causes, and diagnostic procedures.

### 10.1 Device Is Not Enrolled

**Symptoms**

```text
Device ID:          Not enrolled
Enrollment:         Not enrolled
Service Status:     inactive
```

**Possible causes**

- The enrollment-token request failed.
- JophMonitor is unavailable.
- The enrollment endpoint returned an error.
- The agent's stored credentials were removed.
- Device enrollment failed during startup.

**Resolution**

1. Inspect the service logs.

   ```bash
   sudo journalctl -u joph-agent -n 50 --no-pager
   ```

2. Verify JophMonitor connectivity.

   ```bash
   curl -i https://monitor.jophnex.com/agent/health/
   ```

3. Test the enrollment endpoint.

   ```bash
   curl -i --connect-timeout 5 --max-time 20 \
     -X POST \
     https://monitor.jophnex.com/agent/request-token/
   ```

4. Resolve any connection or server errors.

5. Restart the agent.

   ```bash
   sudo systemctl restart joph-agent
   ```

6. Verify enrollment.

   ```bash
   joph-agent show
   ```

**Expected result:** The agent displays an assigned device ID and `Enrollment: Enrolled`.

**Security note:** A successful enrollment-token request returns a sensitive one-time token. Do not share or publish it.

### 10.2 Enrollment Request Times Out

**Symptoms**

```text
Token request failed:
HTTPSConnectionPool(
    host='monitor.jophnex.com',
    port=443
): Read timed out.
```

**Cause**

Joph Agent did not receive an HTTP response from the enrollment endpoint within the configured timeout.

Possible causes include network problems, a slow application response, or server-side processing errors.

**Resolution**

1. Check DNS resolution.

   ```bash
   getent hosts monitor.jophnex.com
   ```

2. Verify the health endpoint.

   ```bash
   curl -v https://monitor.jophnex.com/agent/health/
   ```

3. Test the enrollment endpoint.

   ```bash
   curl -i --connect-timeout 5 --max-time 20 \
     -X POST \
     https://monitor.jophnex.com/agent/request-token/
   ```

4. Inspect JophMonitor logs.

   ```bash
   docker logs --tail 100 JophMonitor-dev
   ```

**Expected result:** The enrollment endpoint returns a successful response within the configured timeout.

### 10.3 JophMonitor Returns HTTP 500

**Symptoms**

```text
Internal Server Error:
/agent/request-token/
```

**Cause**

JophMonitor encountered an internal error while processing the request.

Possible causes include:

- Django application exception
- Database connection failure
- Stored procedure failure
- Unexpected application response
- Database transaction error

**Resolution**

1. Inspect Django logs.

   ```bash
   docker logs --tail 100 JophMonitor-dev
   ```

2. Verify database connectivity.

3. Test the enrollment-token stored procedure in the JophMonitoring database.

   ```sql
   CALL stpInsertEnrollmentToken(
       'test_enrollment_token'
   );
   ```

4. Inspect the returned result code.

A successful operation returns result code `1` and a generated token ID.

**Note:** A successful database test confirms that the stored procedure works for the tested input. It does not guarantee that the Django endpoint is functioning correctly.

### 10.4 Docker Permission Denied

**Symptoms**

```text
permission denied while trying to connect
to the Docker API at
unix:///var/run/docker.sock
```

**Cause**

The `joph-agent` system account does not have permission to access the Docker socket.

**Resolution**

1. Inspect Docker socket permissions.

   ```bash
   ls -l /var/run/docker.sock
   ```

2. Inspect the agent account.

   ```bash
   id joph-agent
   ```

3. If Docker monitoring is required and the security implications are accepted, grant Docker group membership.

   ```bash
   sudo usermod -aG docker joph-agent
   ```

4. Restart Joph Agent.

   ```bash
   sudo systemctl restart joph-agent
   ```

5. Review the logs.

   ```bash
   sudo journalctl -u joph-agent -n 30 --no-pager
   ```

**Security warning:** Docker group membership can provide root-equivalent access to the host. Use this configuration only for trusted accounts and consider more restricted access mechanisms in production.

### 10.5 JophMonitor Hostname Cannot Be Resolved

**Symptoms**

```text
Could not resolve host
```

**Possible cause**

The Linux device cannot resolve the configured JophMonitor hostname.

**Resolution**

1. Check DNS resolution.

   ```bash
   getent hosts monitor.jophnex.com
   ```

2. Verify HTTPS connectivity.

   ```bash
   curl -v https://monitor.jophnex.com/agent/health/
   ```

3. Correct the DNS configuration if necessary.

For a development server located on the same local network, an internal DNS record or `/etc/hosts` entry may be appropriate.

Example:

```text
192.168.1.228 monitor.jophnex.com
```

The address above is specific to the example development environment. Replace it with the correct internal server address.

**Note:** Preserve the hostname in the HTTPS URL so that TLS certificate validation continues to work.

### 10.6 Agent Service Is Inactive

**Symptoms**

```text
Service Status: inactive
```

**Resolution**

1. Check the service status.

   ```bash
   sudo systemctl status joph-agent --no-pager
   ```

2. Review the latest logs.

   ```bash
   sudo journalctl -u joph-agent -n 50 --no-pager
   ```

3. Identify and resolve the reported error.

4. Restart the service.

   ```bash
   sudo systemctl restart joph-agent
   ```

5. Verify the result.

   ```bash
   joph-agent show
   ```

**Expected result:** The service reports `active`.

### 10.7 Monitoring Reports Are Not Reaching JophMonitor

**Symptoms**

- Reports remain stored locally.
- Recent monitoring information is missing from JophMonitor.
- Report synchronization errors appear in the logs.

**Resolution**

1. Inspect the local database.

   ```bash
   joph-agent check_database
   ```

2. Review agent logs.

   ```bash
   sudo journalctl -u joph-agent -n 100 --no-pager
   ```

3. Verify JophMonitor connectivity.

   ```bash
   curl -i https://monitor.jophnex.com/agent/health/
   ```

4. Inspect incoming requests on JophMonitor.

   ```bash
   docker logs --tail 100 JophMonitor-dev
   ```

5. Verify report processing and database persistence.

**Important:** Do not delete `agent.db` while investigating unsent monitoring reports. Deleting the database may permanently remove locally stored data.

### 10.8 Agent Requires Enrollment After Reinstallation

**Symptoms**

After reinstalling Joph Agent, the application starts without a saved device ID.

**Cause**

The previous installation's persistent data directory was removed.

The directory contains the agent's saved identity and credentials.

**Resolution**

Allow the agent to complete a new enrollment, provided that JophMonitor is reachable and accepts the request.

Verify the result:

```bash
joph-agent show
```

**Prevention**

Preserve the following directory during routine application updates:

```text
/var/lib/joph-agent/
```

Do not remove persistent data unless a complete reset is intended.

---

## 11. Diagnostic Command Reference

| Task | Command |
|---|---|
| Display CLI help | `joph-agent --help` |
| Display agent information | `joph-agent show` |
| Configure JophMonitor | `joph-agent configure` |
| Inspect local reports | `joph-agent check_database` |
| Check service status | `sudo systemctl status joph-agent --no-pager` |
| Start service | `sudo systemctl start joph-agent` |
| Stop service | `sudo systemctl stop joph-agent` |
| Restart service | `sudo systemctl restart joph-agent` |
| Enable startup | `sudo systemctl enable joph-agent` |
| Disable startup | `sudo systemctl disable joph-agent` |
| Display recent logs | `sudo journalctl -u joph-agent -n 50 --no-pager` |
| Follow live logs | `sudo journalctl -u joph-agent -f` |
| Check DNS resolution | `getent hosts monitor.jophnex.com` |
| Test JophMonitor health | `curl -i https://monitor.jophnex.com/agent/health/` |
| Inspect server logs | `docker logs --tail 100 JophMonitor-dev` |

---

## 12. JophMonitor API Integration

Joph Agent communicates with JophMonitor using HTTPS endpoints.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/agent/health/` | Check server availability and API compatibility |
| POST | `/agent/request-token/` | Request a one-time enrollment token |
| POST | `/agent/enroll/` | Register a device |
| POST | `/agent/auth/` | Authenticate an enrolled device |
| POST | `/agent/policy/` | Retrieve monitoring policy |
| POST | `/agent/policy/services/` | Retrieve service monitoring policies |
| POST | `/agent/report/system/` | Submit system reports |
| POST | `/agent/report/containers/` | Submit container reports |

Protected endpoints require valid device authentication.

JophMonitor validates requests and processes the corresponding monitoring operations.

---

## 13. Security

Joph Agent includes several security measures.

### 13.1 HTTPS Communication

Joph Agent communicates with JophMonitor over HTTPS.

HTTPS protects communication against passive interception and provides server authentication when certificate validation succeeds.

### 13.2 Device Authentication

Each enrolled device uses its saved authentication credentials when communicating with protected JophMonitor endpoints.

Enrollment uses a one-time token.

### 13.3 Dedicated Linux Account

Joph Agent runs under a dedicated Linux system account.

This separates the agent process from ordinary interactive user accounts.

### 13.4 Persistent Data Protection

Agent configuration and credentials are stored in the persistent data directory.

Administrators should restrict access to this directory and avoid exposing its contents.

### 13.5 Release Integrity Verification

The installer verifies downloaded release assets using SHA-256 checksums.

Checksum verification detects mismatches between downloaded files and the published checksum values.

For stronger publisher authentication, signed release manifests or another trusted verification mechanism should be considered.

### 13.6 Docker Access

Docker monitoring requires access to the Docker Engine API.

Because Docker socket access can provide elevated host privileges, it should be granted only when necessary.

---

## 14. Agent Operation Sequence

The following diagram summarizes the application lifecycle.

```text
Install Joph Agent
        |
        v
Configure JophMonitor
        |
        v
Start systemd Service
        |
        v
Check Saved Credentials
        |
        +---- Not Enrolled ----+
        |                      |
        |                      v
        |               Request Token
        |                      |
        |                      v
        |                Enroll Device
        |                      |
        |                      v
        |               Save Credentials
        |                      |
        +----------------------+
        |
        v
Authenticate Device
        |
        v
Retrieve Monitoring Policies
        |
        v
Start Monitoring Threads
        |
        v
Collect Monitoring Data
        |
        v
Store and Synchronize Reports
        |
        v
Continue Monitoring
```

---

## 15. Project Information

**Project:** Joph Agent  
**Version:** 1.0.0  
**Programming Language:** Python  
**Supported Platform:** Linux  
**Architecture:** Client–Server  
**Monitoring Server:** JophMonitor  
**Local Database:** SQLite  
**Service Manager:** systemd  
**Distribution:** GitHub Releases

### Repository

https://github.com/Jophmukendi/Joph_Agent

### Release

https://github.com/Jophmukendi/Joph_Agent/releases/tag/v1.0.0

### Monitoring Server

https://monitor.jophnex.com

---

## 16. Conclusion

Joph Agent 1.0.0 provides a centralized approach to Linux infrastructure monitoring.

It combines automated enrollment, device authentication, system and service monitoring, Docker container monitoring, policy synchronization, and local report storage.

The application is distributed through GitHub Releases and managed through a command-line interface and systemd.

Its modular architecture provides a foundation for future monitoring and administration capabilities.

