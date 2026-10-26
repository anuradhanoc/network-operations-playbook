# SOP-NET-001: Reverse Path Connectivity Investigation

## 1. Case Details

- **Category:** L2 Network Troubleshooting
- **Issue Type:** Connectivity / Reverse Path Investigation
- **Device Platform:** Juniper JunOS
- **Status:** Connectivity successful during the provided tests; reverse-path investigation pending.
- **Objective:** Validate destination reachability, verify the directly connected route, and request source/destination IP details and reverse-path utilization checks from the remote team.

## 2. Issue Description

The reporting team was asked to provide the correct destination IP (D IP), source IP (S IP), and reverse-path details corresponding to the problematic time.

The local team does not have administrative control over the reverse path and requested the remote team to check utilization on its side.

## 3. Troubleshooting Summary

Three checks were performed:

1. Ping the remote destination using the specified source IP.
2. Verify the route to the local peer IP.
3. Ping the local peer using the same source IP to confirm local link reachability.

All packets were received successfully in both ping tests. However, these results do not establish whether a reverse-path issue occurred during the reported problem window.

---

## 4. SOP - Follow the Below Steps
Follow the steps below to troubleshoot destination connectivity and validate the directly connected peer. The procedure includes source-to-destination ping testing, route verification, and peer connectivity testing to identify potential network issues.

> **Note:** The command outputs in this SOP use anonymized identifiers and illustrative placeholders. They demonstrate the troubleshooting procedure and do not reproduce exact live-device outputs.

### 4.1 Destination Connectivity Test
**Purpose:** Verify reachability to the destination from the specified source address.

```text
User@ISP-CRTR> ping <DST_IP> source <SRC_IP> rapid count 100
PING DST_IP (DST_IP): 56 data bytes
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
--- DST_IP ping statistics ---
100 packets transmitted, 100 packets received, 0% packet loss
round-trip min/avg/max/stddev = <MIN_RTT>/<AVG_RTT>/<MAX_RTT>/<STDDEV> ms
```

**Observation:**
- Destination reachability was successful during the test.
- Packets transmitted: 100
- Packets received: 100
- Packet loss: 0%
- Average RTT: <AVG_RTT> ms
- Maximum RTT: <MAX_RTT> ms

**Conclusion:** The destination was reachable during the test. The relatively high RTT warrants comparison against the expected latency for this path, but does not independently prove congestion or a reverse-path fault.

### 4.2 Route Verification
**Purpose:** Verify the route to the local peer address and identify the outgoing interface.

```text
{master}
User@ISP-CRTR> show route <PEER_IP>

inet.0: <DESTINATIONS> destinations, <ROUTES> routes (<ACTIVE> active, <HOLDDOWN> holddown, <HIDDEN> hidden)
Restart Complete
+ = Active Route, - = Last Active, * = Both

<PEER_IP>/31 *[Direct/0] <ROUTE_AGE>
> via ae<INTERFACE>.0

{master}
```

**Observation:**
- The `PEER_IP/31` route was active and directly connected.
- Status: Active
- Route type: Direct
- Prefix length: `/31`
- Outgoing interface: `ae<INTERFACE>.0`

**Conclusion:** The example route is represented as directly connected and active. In this sanitized output, the route and interface are illustrative placeholders mapped from the original log.

### 4.3 Local Peer Reachability Test
**Purpose:** Verify connectivity between the current router and the directly connected peer.

```text
User@ISP-CRTR> ping <PEER_IP> source <SRC_IP> rapid count 200
PING PEER_IP (PEER_IP): 56 data bytes
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
--- PEER_IP ping statistics ---
200 packets transmitted, 200 packets received, 0% packet loss
round-trip min/avg/max/stddev = <MIN_RTT>/<AVG_RTT>/<MAX_RTT>/<STDDEV> ms

{master}
User@ISP-CRTR>
```

**Observation:**
- The local peer responded to all 200 packets.
- Packets received: 200
- Packet loss: 0%
- Average RTT: <AVG_RTT> ms
- Maximum RTT: <MAX_RTT> ms

**Conclusion:** The local peer was reachable with no packet loss during this test.
---

## 5. Root Cause Analysis

**Confirmed root cause:** Not established from the supplied logs.

The evidence confirms successful connectivity during the two ping tests and an active, directly connected route to the local peer.

The evidence does not establish:
- Whether packet loss occurred at the reported problematic time.
- Whether congestion occurred on the forward or reverse path.
- Whether the remote network experienced interface utilization issues.
- Whether asymmetric routing or another path-specific issue contributed to the reported symptoms.

## 6. Recommended Next Steps

1. Obtain the correct source IP and destination IP for the affected traffic.
2. Request the exact date, time, and timezone of the reported issue.
3. Request reverse-path details for the same time window.
4. Check local interface utilization, errors, drops, and congestion counters.
5. Ask the remote team to check utilization and packet drops on its side.
6. Repeat testing during the problem window, if possible.
7. Compare monitoring data and latency results before drawing a final conclusion.

## Customer Communication Template

Hi Team,

Kindly share the correct destination IP (D IP) and source IP (S IP). If you are experiencing an issue with the reverse path, please share the relevant reverse-path details and the exact timestamp of the issue.

We do not have administrative control over the reverse path. Please check interface utilization, congestion, and packet drops from your end during the reported time window.


## Final Assessment

The available tests show successful destination connectivity and local peer reachability with 0% packet loss.

Further investigation requires the correct traffic endpoints, the problematic timestamp, reverse-path information, and utilization data from the relevant network segments.

**Status:** Pending additional evidence from the reporting and remote teams.


