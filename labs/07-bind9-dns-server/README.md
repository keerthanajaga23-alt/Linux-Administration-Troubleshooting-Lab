
# Lab 07: BIND9 DNS Server Configuration

## 1. Objective

The objective of this lab is to practise basic DNS server administration using BIND9 on Ubuntu running under WSL2.

The lab covers installing and identifying the BIND9 server, defining a DNS zone, configuring DNS records, validating configuration files, testing DNS queries, and investigating a service-control issue.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **DNS software:** BIND9
* **BIND version observed:** 9.18.39
* **DNS zone:** `lab.test`
* **Zone file:** `/etc/bind/db.lab.test`
* **Main local-zone configuration:** `/etc/bind/named.conf.local`

The WSL network configuration may change after a restart. Confirm the current interface addresses before using an IP address in a new test.

## 3. What Is BIND9?

BIND9 is DNS server software used to answer DNS queries and provide authoritative DNS information for configured zones.

### Important terms

* **BIND9:** DNS server software.
* **`named`:** The BIND DNS server daemon.
* **DNS zone:** A portion of the DNS namespace managed as a unit.
* **Zone file:** A file containing DNS records for a zone.
* **Authoritative DNS server:** A server that provides authoritative information for a zone.
* **A record:** Maps a hostname to an IPv4 address.
* **NS record:** Identifies a name server for a DNS zone.
* **SOA record:** Contains administrative information about a DNS zone, including its serial number and timing parameters.
* **RNDC:** Remote Name Daemon Control, a tool used to control a BIND server.

## 4. Identify BIND9 Tools and Version

Check whether the `named` executable is available:

```bash
which named
```

During this lab, the executable was located at:

```text
/usr/sbin/named
```

Check the version:

```bash
named -v
```

The version observed during the session was:

```text
BIND 9.18.39
```

The `named` executable is the DNS server program. Tools such as `dig` and `nslookup` are DNS query clients used to test name resolution.

## 5. Understand the BIND9 Configuration Files

The main configuration directory is:

```text
/etc/bind/
```

Important configuration files include:

| File                                 | Purpose                                            |
| ------------------------------------ | -------------------------------------------------- |
| `/etc/bind/named.conf`               | Main BIND configuration                            |
| `/etc/bind/named.conf.options`       | Global server options                              |
| `/etc/bind/named.conf.local`         | Local zone definitions                             |
| `/etc/bind/named.conf.default-zones` | Default zone definitions                           |
| `/etc/bind/db.lab.test`              | Zone database created for this lab                 |
| `/etc/bind/rndc.key`                 | Authentication key file used by RNDC configuration |

Inspect the local zone configuration:

```bash
sudo cat /etc/bind/named.conf.local
```

In this lab, the file initially contained comments but no custom zone definition.

## 6. Configure the `lab.test` Zone

A local zone definition was added to `/etc/bind/named.conf.local`:

```text
zone "lab.test" {
    type master;
    file "/etc/bind/db.lab.test";
};
```

### Explanation

* `zone "lab.test"` defines the zone name.
* `type master` identifies this server as the primary authoritative server for the zone in this configuration.
* `file` specifies the path of the zone database.

The `.test` domain is intended for testing and documentation rather than use as a public production domain.

## 7. Create the DNS Zone File

The zone file used in the lab was:

```text
/etc/bind/db.lab.test
```

The initial configuration was:

```dns
$TTL 86400
@   IN  SOA ns1.lab.test. admin.lab.test. (
        2026100501
        3600
        1800
        604800
        86400
)

@       IN  NS  ns1.lab.test.
ns1     IN  A   127.0.0.1
server  IN  A   192.168.1.20
www     IN  A   192.168.1.20
```

### Explanation of the records

**TTL**

`$TTL 86400` sets the default time to live for records that do not specify another TTL. The value is in seconds.

**SOA record**

The Start of Authority record contains zone-management information, including the primary name server, administrative contact representation, serial number, and refresh/retry/expiry values.

**NS record**

```dns
@ IN NS ns1.lab.test.
```

This declares `ns1.lab.test.` as a name server for the zone.

**A records**

```dns
server IN A 192.168.1.20
www    IN A 192.168.1.20
```

These records represent the following mappings:

```text
server.lab.test -> 192.168.1.20
www.lab.test    -> 192.168.1.20
```

### Note about the loopback record

```dns
ns1 IN A 127.0.0.1
```

`127.0.0.1` is the loopback address of the local machine. It is suitable only when the intended use is local to that machine; remote clients cannot use it to reach this DNS server. A deployable network configuration should use an appropriate reachable server address.

## 8. Validate BIND Configuration

Before applying changes, validate the configuration syntax:

```bash
sudo named-checkconf
```

A successful check normally produces no output.

Validate the `lab.test` zone file:

```bash
sudo named-checkzone lab.test /etc/bind/db.lab.test
```

A successful zone check reports `OK`.

Validate the configuration and attempt to load its configured zones:

```bash
sudo named-checkconf -z
```

### Result observed during the lab

The output included:

```text
zone lab.test/IN: loaded serial 2026100501
zone localhost/IN: loaded serial 2
zone 127.in-addr.arpa/IN: loaded serial 1
zone 0.in-addr.arpa/IN: loaded serial 1
zone 255.in-addr.arpa/IN: loaded serial 1
```

This confirmed that the `lab.test` zone was successfully loaded by the configuration validation tool at that point.

Configuration validation checks syntax and zone loading; it does not by itself prove that the server is reachable from every client or network.

## 9. Check the BIND9 Service

Check the service status:

```bash
sudo systemctl status named --no-pager
```

Check the active state:

```bash
sudo systemctl show -p ActiveState -p SubState -p MainPID named
```

During troubleshooting, the service was reported as active.

The BIND server executable was also found listening on the WSL IPv4 address:

```text
172.16.36.24:53
```

This is an address observed during the earlier session, not a permanent address for WSL.

## 10. Check DNS Listening Ports

Run:

```bash
sudo ss -lntup | grep ':53'
```

This displays matching TCP and UDP listening sockets for DNS port 53.

The output during the lab included entries resembling:

```text
172.16.36.24:53 ... users:(("named",pid=203,...))
```

This indicated that `named` was listening on port 53 at that address during the test.

### Important distinction

A DNS server can listen on one local address but not another.

A service listening on `172.16.36.24:53` does not automatically mean it will accept queries sent to `127.0.0.1:53`.

Always test the exact address that a client is using.

## 11. Test the Local DNS Records

During the lab, the following query was run:

```bash
dig @172.16.36.24 server.lab.test +short
```

The observed result was:

```text
192.168.1.20
```

This confirmed that the BIND server answered that query with the expected IPv4 address at the time of testing.

To test the second record, run:

```bash
dig @172.16.36.24 www.lab.test +short
```

The expected result from the configured record is:

```text
192.168.1.20
```

Run this command again and record its actual result before marking the second record as verified.

A further test using `nslookup` is:

```bash
nslookup server.lab.test 172.16.36.24
```

This explicitly specifies which DNS server to query.

## 12. Troubleshooting Scenario: Connection Refused on Loopback

### Symptom

A query to the loopback address returned:

```text
communications error to 127.0.0.1#53: connection refused
```

### Investigation

First inspect the listening sockets:

```bash
sudo ss -lntup | grep ':53'
```

Then inspect the loopback interface:

```bash
ip addr show lo
```

During the lab, the loopback interface displayed:

```text
127.0.0.1/8
10.255.255.254/32
```

The output also showed BIND listening on `172.16.36.24:53`.

### Finding

The local query failed even though a query directed to `172.16.36.24` succeeded.

The relevant troubleshooting lesson is to check the exact IP address on which the service is listening and compare it with the address used by the client.

Do not assume that a running service accepts connections on every interface.

## 13. Troubleshooting Scenario: Invalid DNS Record

### Objective

Practise detecting and correcting an invalid IPv4 address in the DNS zone file.

An invalid address was introduced during the exercise:

```dns
www IN A 192.168.1.999
```

The address `192.168.1.999` is invalid because an IPv4 octet must be in the range 0–255.

The intended record was:

```dns
www IN A 192.168.1.20
```

After restoring the correct value, validate the zone:

```bash
sudo named-checkzone lab.test /etc/bind/db.lab.test
```

The check subsequently returned `OK`.

### Troubleshooting workflow

1. Identify the incorrect record.
2. Correct the IPv4 address.
3. Validate the zone file.
4. Validate the overall BIND configuration.
5. Apply the change using the supported service-control method.
6. Query the record again to verify the result.

**Verification note:** The zone validation passed after restoration. Record a new successful DNS query after applying the restored file before claiming that the full failure-and-recovery cycle has been verified.

## 14. Troubleshooting Scenario: RNDC Reload Failure

### Symptom

The command:

```bash
sudo systemctl reload named
```

failed with a message indicating that `named.service` could not be reloaded.

The service unit was inspected using:

```bash
sudo systemctl show named -p ExecStart -p ExecReload
```

The output showed that the reload command invoked `rndc reload`.

The following command was also tested:

```bash
sudo rndc status
```

It returned:

```text
rndc: connect failed: 127.0.0.1#953: timed out
```

Inspection showed that `named` was listening on TCP port 953 on `127.0.0.1`.

The RNDC key file was present at:

```text
/etc/bind/rndc.key
```

An explicit-key test also timed out:

```bash
sudo rndc -k /etc/bind/rndc.key -s 127.0.0.1 status
```

### Current finding

The `lab.test` zone passed validation, and a direct DNS query to `172.16.36.24` returned the expected address for `server.lab.test`.

However, the RNDC control request timed out. The exact cause of this control-channel issue was not established during the session.

Therefore, the RNDC reload issue remains **unresolved**.

### Further investigation

When continuing this lab:

1. Capture the full service status and journal messages at the time of failure.
2. Inspect the active BIND control configuration and the `rndc` client configuration.
3. Verify that the daemon and client use compatible control-channel settings and authentication.
4. Test a supported reload method after validating the configuration.
5. Confirm that the zone data was actually reloaded by querying the server again.

Do not claim that RNDC or the service reload is fixed until the test succeeds.

## 15. Key Commands Reference

| Command                     | Purpose                                          |
| --------------------------- | ------------------------------------------------ |
| `named -v`                  | Display the BIND version                         |
| `which named`               | Locate the `named` executable                    |
| `named-checkconf`           | Validate BIND configuration syntax               |
| `named-checkconf -z`        | Validate configuration and attempt to load zones |
| `named-checkzone ZONE FILE` | Validate a zone file                             |
| `systemctl status named`    | Inspect the service status                       |
| `systemctl show`            | Display selected service properties              |
| `ss -lntup`                 | Inspect listening TCP and UDP sockets            |
| `ip addr show lo`           | Inspect the loopback interface                   |
| `dig @DNS-IP HOSTNAME A`    | Query a particular DNS server                    |
| `nslookup HOSTNAME DNS-IP`  | Test a hostname using a specific DNS server      |
| `rndc status`               | Query the BIND control interface                 |
| `journalctl -u named`       | Inspect BIND service logs                        |

Replace placeholders with the appropriate values from your environment.

Learning Outcomes

This lab provides practical exposure to:

* Basic BIND9 DNS server configuration.
* Authoritative DNS zones and resource records.
* SOA, NS, and A records.
* DNS configuration and zone validation.
* DNS query testing using `dig` and `nslookup`.
* Listening-address and service-control troubleshooting.
* Distinguishing DNS query functionality from the RNDC control channel.
* Recording verified findings and unresolved issues accurately.
