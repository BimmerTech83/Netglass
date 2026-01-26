# SonicWALL Template - LLD (Low-Level Discovery) Implementation Summary

## Overview
This template maximizes the use of Zabbix's Low-Level Discovery (LLD) feature to automatically discover and monitor SonicWALL firewall components without manual configuration.

## LLD Discovery Rules Implemented

### 1. **Interface Discovery** (Standard SNMP)
- **Key**: `net.if.discovery`
- **Discovery OID**: `.1.3.6.1.2.1.2.2.1.2` (ifDescr)
- **Filter**: Monitors only physical interfaces (types 6, 131, 135)
- **Discovered Macros**:
  - `{#IFNAME}` - Interface name
  - `{#IFDESCR}` - Interface description
  - `{#IFTYPE}` - Interface type
  - `{#IFINDEX}` - Interface index

**Auto-Discovered Items per Interface**:
- Incoming throughput (bps) with bits conversion
- Outgoing throughput (bps) with bits conversion
- Incoming errors (counter)
- Outgoing errors (counter)
- Administrative status
- Operational status

**Triggers**:
- ✅ High incoming throughput (>1Gbps)
- ✅ High outgoing throughput (>1Gbps)
- ✅ Increasing incoming errors
- ✅ Increasing outgoing errors
- ✅ Operational status down

**Use Case**: Automatically monitors all physical and logical interfaces including VLANs without manual item creation.

---

### 2. **SonicWALL Per-Interface Metrics Discovery**
- **Key**: `sonicwall.if.discovery`
- **Discovery OID**: `.1.3.6.1.4.1.8741.1.3.1.8.1.1.1` (sonicIfIndex)
- **Discovered Macros**:
  - `{#IFINDEX}` - SonicWALL-specific interface index

**Auto-Discovered Items per Interface**:
- Interface usage percentage (SonicWALL-specific metric)
- Interface throughput in Mbps (SonicWALL-specific metric)

**Triggers**:
- ✅ Interface approaching saturation (>900 Mbps)

**Use Case**: Provides SonicWALL vendor-specific performance metrics for detailed per-interface monitoring.

---

### 3. **IPSec VPN Site-to-Site Discovery**
- **Key**: `vpn.ipsec.discovery`
- **Discovery OID**: `.1.3.6.1.4.1.8741.1.3.2.1.1.2` (VPN name)
- **Discovered Macros**:
  - `{#VPNNAME}` - VPN tunnel name
  - `{#VPNSTATUS}` - Tunnel status indicator

**Auto-Discovered Items per VPN**:
- Tunnel status monitoring
- Bytes in (throughput)
- Bytes out (throughput)

**Triggers**:
- ✅ VPN tunnel down/disconnected

**Use Case**: Automatically discovers and monitors all configured IPSec site-to-site VPN tunnels, alerting on failures.

---

### 4. **IPSec Security Association (SA) Discovery**
- **Key**: `ipsec.sa.discovery`
- **Discovery OID**: `.1.3.6.1.4.1.8741.1.3.2.1.1.1.1` (sonicIpsecSaIndex)
- **Discovered Macros**:
  - `{#SAINDEX}` - Security Association index

**Auto-Discovered Items per SA**:
- SA index tracking

**Use Case**: Provides low-level discovery of IPSec Security Associations for detailed encryption session monitoring and troubleshooting.

---

### 5. **VPN SSL/TLS Remote Access Discovery**
- **Key**: `vpn.ssl.discovery`
- **Discovery OID**: `.1.3.6.1.4.1.8741.1.3.3.1.1.2` (SSL VPN name)
- **Discovered Macros**:
  - `{#VPNNAME}` - Remote access VPN name
  - `{#VPNSTATUS}` - Session status

**Auto-Discovered Items per SSL VPN**:
- SSL tunnel status monitoring

**Triggers**:
- ✅ SSL VPN session inactive

**Use Case**: Automatically monitors all remote access SSL/TLS VPN sessions for user connectivity issues.

---

## Non-Discovered Static Items

The following items are collected once per firewall and do not require LLD:

### System Metrics
- `system.uptime` - System uptime
- `system.hostname` - Device hostname
- `system.location` - Device location
- `system.contact` - System contact

### SonicWALL Performance Metrics
- `sonicwall.cpu.fwd` - Data forwarding CPU utilization (with high CPU alert at 85%)
- `sonicwall.cpu.mgmt` - Management CPU utilization (with high CPU alert at 85%)
- `sonicwall.ram.util` - RAM utilization percentage (with alert at 90%)
- `sonicwall.conn.cache` - Active connection cache entries
- `sonicwall.conn.cache.max` - Maximum connection cache capacity
- `sonicwall.nat.translation` - Active NAT translations

---

## Benefits of LLD Implementation

1. **Scalability**: Automatically discovers all interfaces and VPNs - no manual item creation
2. **Maintainability**: Adding new interfaces/VPNs is automatic
3. **Efficiency**: Reduces template update overhead
4. **Consistency**: All similar items use identical monitoring parameters
5. **Flexibility**: Filter rules can be adjusted to include/exclude specific device types
6. **Completeness**: No interfaces or VPNs are missed due to configuration drift

---

## OID Reference

| Component | OID | Type |
|-----------|-----|------|
| Interface Discovery | .1.3.6.1.2.1.2.2.1 (IF-MIB) | Standard SNMP |
| SonicWALL Interface Index | .1.3.6.1.4.1.8741.1.3.1.8.1.1 | Vendor-specific |
| IPSec Tunnel Discovery | .1.3.6.1.4.1.8741.1.3.2.1.1 | Vendor-specific |
| IPSec SA Discovery | .1.3.6.1.4.1.8741.1.3.2.1.1.1.1 | Vendor-specific |
| SSL/TLS VPN Discovery | .1.3.6.1.4.1.8741.1.3.3.1.1 | Vendor-specific |

---

## Deployment Notes

- Ensure SNMP community string includes access to enterprise OIDs (8741 = SonicWALL)
- All MIB files should be loaded in Zabbix for proper OID translation
- Discovery rules run every 1 hour by default - adjust if more frequent updates needed
- Discovered items inherit the same history/trend settings from prototypes
- Preprocessing rules (like bits conversion) apply consistently to all discovered items

---

## Future Enhancement Opportunities

Additional LLD rules that could be implemented:

1. **Firewall Rules/Policies Discovery** - If available via SNMP
2. **User Authentication Sessions** - SSL VPN user count and duration
3. **Threat/Security Events** - If exposed via MIB
4. **Memory Modules** - If CPU/RAM can be broken down by hardware component
5. **Service/Module Status** - AV, IPS, WAF engine statuses

