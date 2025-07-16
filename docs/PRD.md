# Product Requirements Document: DefenderMonitor

## Overview
DefenderMonitor is a lightweight Windows service written in Go. It
periodically checks that Microsoft Defender is healthy and running
on Windows 11 workstations. If Defender is found to be stopped or in
an error state, the service should alert IT staff so they can take
corrective action.

## Goals
- Detect when Microsoft Defender is not running or unhealthy on a
  Windows 11 workstation.
- Support thousands of clients running simultaneously.
- Send status reports to a centralized monitoring solution so IT staff
  can track the health of all clients.
- Handle system reboots gracefully by delaying checks until both
  Windows Defender and the network are fully available.

## Non‑Goals
- Full antivirus management. DefenderMonitor only verifies Defender is
  running and can report problems; it does not update or manage
  Defender itself.
- Real‑time or deep health metrics. The monitor only needs to know
  whether the service is running and optionally fetch minimal status
  (like threat definition updates).

## Functional Requirements
1. **Windows Service**
   - The monitor runs as a service on each workstation.
   - It must start automatically with Windows but allow a configurable
     delayed start so that Defender has time to initialize after
     reboot.
2. **Periodic Health Check**
   - At a configurable interval (default: every 5 minutes), the
     service checks Microsoft Defender’s status via the Windows
     SecurityCenter or relevant APIs.
   - If Defender is stopped or not healthy, the service logs this and
     marks the workstation as degraded.
3. **Alerting**
   - The service sends an alert to IT when a workstation is degraded.
   - Alerting should support sending to a centralized system such as
     an HTTP endpoint, syslog, or message queue. The exact protocol may
     be configurable.
4. **Centralized Monitoring**
   - Workstations periodically send a “heartbeat” status report even
     when healthy. This allows a remote monitoring console to see which
     workstations are online and the status of Defender.
   - Heartbeats should include machine identifier, timestamp, and the
     last known Defender state.
5. **Reliability**
   - The service should retry failed network transmissions to avoid
     losing alerts when connectivity is intermittent.
   - Logging should be local and remote; if remote upload fails, keep
     a history locally for troubleshooting.

## Technical Requirements
- **Language**: Go (Golang) to keep the service small and easy to
  maintain.
- **Platform**: Windows 11 x64. Should also work on Windows 10 where
  possible.
- **Service Manager**: Use the native Windows service framework via Go
  libraries such as `golang.org/x/sys/windows/svc`.
- **API Usage**: Interrogate Microsoft Defender via the Windows
  Security Center API or Windows Management Instrumentation (WMI) to
  check service status and last update time.
- **Configuration**: Provide a config file or registry‐based settings
  for interval, alerting endpoint, and log paths.
- **Security**: Communications to the monitoring endpoint must be
  authenticated (for example using a shared key or certificate) and
  support TLS.

## User Experience
- The service itself has no user interface.
- Logs should be accessible locally for administrative review.
- For simplicity, the initial implementation can write plain text logs
  to a configurable directory and send JSON payloads to the central
  monitor.

## Milestones
1. **Prototype**
   - Basic service that checks Defender status locally and logs result.
2. **Centralized Reporting**
   - Add the heartbeat and alerting features with a simple HTTP
     endpoint for testing.
3. **Configuration and Reliability**
   - Implement robust configuration loading and retries for networking
     failures.
4. **Packaging**
   - Provide instructions and scripts to install the service on
     workstations using standard Windows tools.

