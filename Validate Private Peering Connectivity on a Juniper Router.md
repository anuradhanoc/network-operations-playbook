# 4. SOP – Follow the Below Steps

Follow the steps below to validate private peering connectivity on a Juniper router. The procedure includes local route verification, physical and logical interface health checks, interface utilization analysis, ARP neighbor verification, BGP session status validation, and configuration review.

**Note:** The command outputs in this SOP use anonymized identifiers and illustrative placeholders. They preserve realistic Juniper CLI formatting and representative operational values without reproducing the original provider, circuit, router, or configuration details.

## 4.1 Local Interface Address and Route Verification

**Purpose:** Verify that the router's local peering IP address is installed as an active local route and identify the associated logical interface.

**Command:**

```text
User@ISP-CRTR> show route <LOCAL_IP> best active-path

inet.0: <DESTINATIONS> destinations, <ROUTES> routes (<ACTIVE> active, <HOLDDOWN> holddown, <HIDDEN> hidden)
+ = Active Route, - = Last Active, * = Both

<LOCAL_IP>/32 *[Local/0] <ROUTE_AGE>
                       Local via ae<INTERFACE>.0

{master}
```

**Observation:**

- The local peering IP has an active `/32` local route.

- Route type: `Local`.

- Route status: Active.

- Associated logical interface: `ae<INTERFACE>.0`.

**Conclusion:** The local peering IP is installed as an active `/32` local route through `ae<INTERFACE>.0`. This confirms local address recognition and the associated logical interface, but does not independently establish end-to-end connectivity.

## 4.2 Physical and Logical Interface Health Verification

**Purpose:** Verify the operational status, bandwidth, logical interface state, and configured peering addresses on the associated interface.

**Command:**

```text
User@ISP-CRTR> show interfaces ae<INTERFACE>

Physical interface: ae<INTERFACE>, Enabled, Physical link is Up

  Interface index: <INTERFACE_INDEX>, SNMP ifIndex: <SNMP_IFINDEX>

  Description: ##PRIVATE-PEERING<CIRCUIT_DETAILS_REDACTED>##

  Link-level type: Ethernet, MTU: <PHYSICAL_MTU>, Speed: <INTERFACE_SPEED>, BPDU Error: None, MAC-REWRITE Error: None, Loopback: Disabled, Source filtering: Disabled,

  Flow control: Disabled

  Pad to minimum frame size: Disabled

  Minimum links needed: 1, Minimum bandwidth needed: 1bps

  Device flags   : Present Running

  Interface flags: SNMP-Traps Internal: <INTERFACE_FLAGS>

  Current address: <ACTIVE_INTERFACE_MAC>, Hardware address: <HARDWARE_MAC>

  Last flapped   : <LAST_FLAP_TIMESTAMP>

  Input rate     : <INPUT_RATE> bps (<INPUT_PPS> pps)

  Output rate    : <OUTPUT_RATE> bps (<OUTPUT_PPS> pps)

Logical interface ae<INTERFACE>.0 (Index <LOGICAL_INDEX>) (SNMP ifIndex <SNMP_IFINDEX>)

    Flags: Up SNMP-Traps <LOGICAL_INTERFACE_FLAGS> Encapsulation: ENET2

    Statistics     Packets           pps            Bytes             bps

    Bundle:

        Input : <INPUT_PACKETS>    <INPUT_PPS>   <INPUT_BYTES>     <INPUT_BPS>

        Output: <OUTPUT_PACKETS>   <OUTPUT_PPS>  <OUTPUT_BYTES>    <OUTPUT_BPS>

    Adaptive Statistics:

        Adaptive Adjusts: <ADAPTIVE_ADJUSTS>

        Adaptive Scans:   <ADAPTIVE_SCANS>

        Adaptive Updates: <ADAPTIVE_UPDATES>

    Protocol inet, MTU: <IPV4_MTU>

    Max nh cache: <MAX_NH_CACHE>, New hold nh limit: <NEW_HOLD_NH_LIMIT>, Curr nh cnt: <CURRENT_NH_COUNT>, Curr new hold cnt: <CURRENT_NEW_HOLD_COUNT>, NH drop cnt: <NH_DROP_COUNT>

      Flags: <IPV4_FLAGS>

      Addresses, Flags: Is-Preferred Is-Primary

        Destination: <IPV4_SUBNET>/<PREFIX_LENGTH>, Local: <LOCAL_IPV4>, Broadcast: <BROADCAST_IPV4>

    Protocol inet6, MTU: <IPV6_MTU>

    Max nh cache: <MAX_NH_CACHE>, New hold nh limit: <NEW_HOLD_NH_LIMIT>, Curr nh cnt: <CURRENT_NH_COUNT>, Curr new hold cnt: <CURRENT_NEW_HOLD_COUNT>, NH drop cnt: <NH_DROP_COUNT>

      Flags: <IPV6_FLAGS>

      Addresses, Flags: Is-Preferred Is-Primary

        Destination: <IPV6_SUBNET>/<PREFIX_LENGTH>, Local: <LOCAL_IPV6>

      Addresses, Flags: Is-Preferred

        Destination: <LINK_LOCAL_SUBNET>, Local: <LINK_LOCAL_IPV6>

    Protocol multiservice, MTU: <MULTISERVICE_MTU>

{master}

```

**Observation:**

- Physical interface status: `Up`.

- Logical interface status: `Up`.

- Interface speed: `<INTERFACE_SPEED>`.

- Active interface MAC: `<ACTIVE_INTERFACE_MAC>`.

- Hardware MAC: `<HARDWARE_MAC>`.

- IPv4 local address: `<LOCAL_IPV4>`.

- IPv4 connected subnet: `<IPV4_SUBNET>/<PREFIX_LENGTH>`.

- IPv6 addressing is configured on the logical interface.

- Input and output traffic counters are available.

- Last interface flap: `<LAST_FLAP_TIMESTAMP>`.

**Conclusion:** The physical and logical interfaces are operational, with IPv4 and IPv6 addressing configured. Interface traffic counters are available, and the last flap timestamp is recorded. Further analysis of interface errors, drops, and historical utilization is required to rule out intermittent issues or congestion.

## 4.3 ARP Neighbor Verification

**Purpose:** Verify that the remote IPv4 peering address resolves to a neighbor MAC address on the expected logical interface.

**Command:**

```text
User@ISP-CRTR> show arp interface ae<INTERFACE>.0

MAC Address          Address       Name              Interface          Flags

<PEER_MAC>           <PEER_IP>     <PEER_HOSTNAME>    ae<INTERFACE>.0    <ARP_FLAGS>

{master}

```

**Observation:**

- Remote peer IP: `<PEER_IP>`.

- Peer MAC address: `<PEER_MAC>`.

- ARP entry status: `<ARP_FLAGS>`.

- Associated logical interface: `ae<INTERFACE>.0`.

**Conclusion:** The router has learned the remote peer's MAC address through ARP on `ae<INTERFACE>.0`, indicating Layer 2 neighbor resolution. This does not independently confirm end-to-end connectivity or successful traffic forwarding.

## 4.4 BGP Session Status Verification

**Purpose:** Verify whether the private peering BGP session is established and review the neighbor's operational status.

**Command:**

```text
User@ISP-CRTR> show bgp summary | match <PEER_IP>

<PEER_IP>        <PEER_AS>      <INPUT_MESSAGES>      <OUTPUT_MESSAGES>       <QUEUE>     <PREFIXES> <UPTIME> Establ

{master}

```

**Observation:**

- BGP neighbor IP: `<PEER_IP>`.

- Remote AS: `<PEER_AS>`.

- BGP session state: `Establ`.

- Session uptime: `<BGP_UPTIME>`.

- Input and output message counters are populated.

- Prefix count is displayed in the BGP summary.

**Conclusion:** The BGP session is established between the local router and the remote peer, confirming that the BGP adjacency is operational. However, this does not independently confirm destination reachability, correct route advertisement, or successful end-to-end traffic forwarding.

## 4.5 BGP Neighbor Configuration Verification

**Purpose:** Verify the BGP neighbor configuration, including routing policies, address family, prefix limits, and remote AS.

**Command:**

```text
User@ISP-CRTR> show configuration | display set | match <PEER_IP>

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> description <PEER_DESCRIPTION>

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> import <IMPORT_POLICY_1>

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> import <IMPORT_POLICY_2>

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> family inet unicast prefix-limit maximum <PREFIX_LIMIT>

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> family inet unicast prefix-limit teardown <TEARDOWN_THRESHOLD>

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> export <EXPORT_POLICY>

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> remove-private all

set protocols bgp group <BGP_GROUP> neighbor <PEER_IP> peer-as <PEER_AS>

{master}

```

**Observation:**

- The neighbor is configured under `<BGP_GROUP>`.

- Import policies are configured for received routes.

- An export policy is configured for outbound routes.

- A prefix limit and teardown threshold are configured.

- Private AS removal is enabled for outbound BGP processing.

- The configured peer AS can be compared with the remote AS shown in the BGP summary.

**Conclusion:** The configuration output identifies the BGP neighbor's policies, prefix limits, and remote AS settings. Verify that the configured peer AS matches the BGP summary and review the import/export policies and received/advertised routes to confirm that routing is functioning as intended.

**---**

# 5. Interface Utilization Analysis

**Purpose:** Assess the traffic rates on the peering interface and determine whether the reported utilization indicates potential congestion.

**Command:**
```text
User@ISP-CRTR> show interfaces ae<INTERFACE>

Physical interface: ae<INTERFACE>, Enabled, Physical link is Up
  Speed: 100Gbps
  Input rate     : <INPUT_RATE> bps (<INPUT_PPS> pps)
  Output rate    : <OUTPUT_RATE> bps (<OUTPUT_PPS> pps)

{master}
```

**Observation:**

- Illustrative interface capacity: 100 Gbps.
- Input traffic rate: approximately 17.45 Gbps (17.45% utilization).
- Output traffic rate: approximately 9.47 Gbps (9.47% utilization).
- These are illustrative values for the SOP and are not a reproduction of exact live-device output.
- Historical utilization, interface errors, and queue drops should be reviewed during the reported problem period.

**Conclusion:** The illustrative rates do not indicate aggregate interface saturation at the snapshot time: input is approximately 17.45% and output approximately 9.47% of a 100 Gbps capacity. A single snapshot cannot confirm or rule out congestion during the reported issue. Historical interface utilization, queue drops, and error counters should be analyzed for the affected time window.

**Note:** Low aggregate interface utilization does not rule out congestion on individual queues or traffic classes.

**---**

# 6. Root Cause Analysis

**Confirmed root cause:** Not established from the supplied logs.

The supplied examples show the local interface, ARP neighbor resolution, and BGP session as operational. The illustrative interface rates are approximately 17.45% input and 9.47% output against a 100 Gbps capacity; these figures are examples, not verified measurements from the incident.

**Conclusion:** No specific root cause is confirmed. Further investigation requires destination connectivity tests, historical utilization and error checks, and remote-side validation of the reverse path during the reported issue.

## 6.1 Recommended Additional Checks

**A. Verify destination route**

```text
User@ISP-CRTR> show route <DST_IP> exact

```

Confirm the active route, next hop, and outgoing interface.

**B. Verify BGP neighbor**

```text
User@ISP-CRTR> show bgp neighbor <PEER_IP>

```

Check session state and route exchange.

**C. Check interface errors and drops**

```text
User@ISP-CRTR> show interfaces ae<INTERFACE> extensive

```

Review errors, queue drops, and historical utilization.

**D. Test destination connectivity**

```text
User@ISP-CRTR> ping <DST_IP> source <SRC_IP> rapid count 100

```

Record packet loss and RTT during the reported issue.

**E. Request remote-side validation**

Ask the provider to check utilization, errors, routing, and reverse-path behavior for the same timeframe. Share source IP, destination IP, timestamp, and test results.

**Important:** Local peer reachability and an established BGP session do not guarantee end-to-end connectivity.

## 7. Final Assessment

The supplied examples show the local interface, ARP resolution, and BGP session as operational. The illustrative interface utilization does not indicate aggregate saturation at the sample time.

**Limitation:** The available logs do not confirm destination reachability during the incident, historical packet loss, congestion, or reverse-path health. Therefore, the root cause remains unconfirmed.

**Recommended next action:** Collect destination-specific test results, historical interface statistics, and remote-provider reverse-path validation for the reported timeframe.
