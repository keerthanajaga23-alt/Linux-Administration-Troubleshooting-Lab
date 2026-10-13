
# Lab 06: DNS Troubleshooting

## 1. Objective

The objective of this lab is to understand how DNS resolution works and practise troubleshooting hostname resolution failures using Linux commands.

This lab focuses on DNS client troubleshooting. Configuring and managing a local DNS server with BIND9 is covered separately in Lab 07.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Focus:** IPv4 DNS troubleshooting
* **Tools:** `cat`, `ping`, `getent`, `nslookup`, `dig`, `ip`

During an earlier session, the WSL environment showed the following configuration:

* **Default gateway:** `172.16.39.254`
* **DNS resolver:** `10.255.255.254`

These values can change. Check the current configuration before using them in a test.

## 3. What Is DNS?

DNS stands for **Domain Name System**.

DNS allows applications and users to access services using domain names instead of having to remember IP addresses.

Example:

```text
google.com
    |
    v
DNS resolver
    |
    v
IPv4 address
```

When a program needs to connect to a hostname, it generally uses the system's configured name-resolution mechanisms to obtain an IP address.

## 4. Important DNS Terms

### DNS resolver

A DNS resolver receives DNS queries and helps obtain the requested DNS information.

### DNS query

A request for information about a domain name, such as its IPv4 address.

### A record

Maps a hostname to an IPv4 address.

Example:

```text
server.lab.test -> 192.168.1.20
```

### AAAA record

Maps a hostname to an IPv6 address.

### DNS server

A server that answers DNS queries. Depending on its role, it may provide authoritative answers for a zone or resolve queries by contacting other DNS servers.

### DNS forwarder

A DNS server to which another DNS server forwards queries when it needs help resolving them.

### DNS timeout

A query does not receive a response within the expected time.

### NXDOMAIN

A DNS response indicating that the requested domain name does not exist.

### SERVFAIL

A DNS response indicating that the server could not successfully complete the query.

## 5. Inspect the DNS Resolver Configuration

Run:

```bash
cat /etc/resolv.conf
```

Example:

```text
nameserver 10.255.255.254
```

The `nameserver` entry identifies a DNS resolver used by the system.

In the WSL lab, `10.255.255.254` was observed as the configured resolver.

**Troubleshooting objective:** Identify which DNS server the system is configured to use before investigating a resolution failure.

Do not edit `/etc/resolv.conf` blindly. WSL may generate this file automatically, and manual changes may not persist.

## 6. Test IPv4 Connectivity

Test connectivity to a public IPv4 address:

```bash
ping -4 -c 4 8.8.8.8
```

This sends four ICMP Echo Requests to the specified IPv4 address.

A successful response indicates that the destination responded to the ICMP test.

However, a failed ping does not always mean the server is unavailable, because ICMP may be blocked.

**Important:** This command tests IP reachability, not DNS resolution.

## 7. Test System Name Resolution

Run:

```bash
getent ahostsv4 google.com
```

This asks the system's name-service mechanisms to return IPv4 address information for `google.com`.

Possible results:

* One or more IP addresses are returned: the lookup succeeded.
* No addresses are returned: the system did not return a usable result through this lookup.

If the lookup fails, inspect `/etc/resolv.conf` and continue with direct DNS queries.

## 8. Test DNS with nslookup

Run:

```bash
nslookup google.com
```

This tests hostname resolution using the DNS configuration available to `nslookup`.

To query a particular resolver, use:

```bash
nslookup google.com 10.255.255.254
```

Replace the example resolver address with the current nameserver shown in `/etc/resolv.conf` when necessary.

Interpret the result:

* **An IP address is returned:** The DNS query succeeded.
* **Connection timed out:** No response arrived within the expected time.
* **NXDOMAIN:** The queried name was reported as nonexistent.
* **SERVFAIL:** The DNS server could not successfully complete the query.

A timeout can result from several causes, including an unreachable resolver, a service problem, a firewall restriction, or a network-path issue.

## 9. Test DNS with dig

Run:

```bash
dig google.com A
```

This requests the IPv4 A record for `google.com`.

To query a specific DNS server:

```bash
dig @10.255.255.254 google.com A
```

The `@` option specifies the DNS server to query directly.

To display only returned IPv4 addresses:

```bash
dig @10.255.255.254 google.com A +short
```

Use the actual configured resolver address when conducting the test.

### Why use dig?

`dig` provides detailed DNS response information, including:

* DNS response status
* Answer section
* Query time
* DNS server queried

The detailed response can help distinguish a successful lookup from a timeout, a negative answer, or a server failure.

## 10. Understand DNS Query Results

### NOERROR

The DNS server successfully processed the query. Check the answer section to determine whether the requested record was returned.

### NXDOMAIN

The queried domain name does not exist according to the DNS response.

Investigate the spelling of the hostname, the domain name, and whether the record should exist.

### SERVFAIL

The server could not successfully complete the query.

Investigate DNS server health, authoritative data, forwarders, and upstream connectivity as applicable.

### Timeout

The query did not receive a response within the expected time.

Investigate the configured resolver, network path, firewall rules, DNS service availability, and port 53.

### Connection refused

The destination actively refused the connection, or no service was accepting it at the targeted address and port.

Confirm the destination IP and port, the service state, and which addresses the DNS server is listening on.

## 11. Troubleshooting Scenario: IP Works but Domain Name Fails

### Scenario

A user reports that an IP connectivity test works, but the user cannot access a website by its domain name.

### Investigation

First test the IP address:

```bash
ping -4 -c 4 8.8.8.8
```

Inspect the resolver configuration:

```bash
cat /etc/resolv.conf
```

Test system name resolution:

```bash
getent ahostsv4 google.com
```

Test a DNS query:

```bash
nslookup google.com
```

Then query the configured resolver explicitly:

```bash
dig @<DNS-IP> google.com A
```

Replace `<DNS-IP>` with the actual resolver address.

### Troubleshooting workflow

1. Confirm the reported symptom.
2. Test IP connectivity.
3. Identify the configured DNS resolver.
4. Test normal hostname resolution.
5. Query the resolver directly.
6. Investigate the failure indicated by the response.
7. Apply an appropriate correction.
8. Repeat the DNS lookup to verify the outcome.

Do not conclude that DNS is the cause based on one failed ping or one failed application request. Compare the available test results.

## 12. Troubleshooting Scenario: DNS Server Times Out

### Scenario

`nslookup` reports that no DNS servers could be reached.

### Investigation

Check the configured resolver:

```bash
cat /etc/resolv.conf
```

Check the route:

```bash
ip route
```

Test the resolver address using ICMP if appropriate:

```bash
ping -4 -c 4 <DNS-IP>
```

Then send an actual DNS query:

```bash
dig @<DNS-IP> google.com A
```

Replace `<DNS-IP>` with the configured resolver address.

If the query times out, investigate:

* Whether the resolver address is correct.
* Whether the resolver is reachable through the current network.
* Whether the DNS service is available.
* Whether a firewall or network control blocks DNS traffic.
* Whether the resolver's upstream dependencies are functioning.

**Important:** A successful ping does not prove that DNS port 53 is working. An ICMP test and a DNS query test different things.

## 13. Troubleshooting Scenario: The DNS Server Returns an Unexpected Address

### Scenario

A hostname resolves successfully, but it returns an address different from the one expected.

### Investigation

Query the A record:

```bash
dig @<DNS-IP> server.lab.test A
```

Inspect the answer section.

Check:

* Whether the expected hostname was queried.
* Whether the correct DNS server was used.
* Whether the DNS record contains the intended address.
* Whether a cached response or another resolver is involved.
* Whether the authoritative zone has been updated correctly.

For a locally managed zone, validate the BIND9 configuration and zone data as described in Lab 07.

## 14. Local DNS Troubleshooting Practice

The separate BIND9 lab created a local zone named `lab.test` with these intended mappings:

```text
server.lab.test -> 192.168.1.20
www.lab.test    -> 192.168.1.20
```

The following direct query was successfully tested against the WSL IPv4 address `172.16.36.24`:

```bash
dig @172.16.36.24 server.lab.test +short
```

The observed result was:

```text
192.168.1.20
```

This confirms that the BIND9 server returned the expected A-record address for that query at the time of the test.

A query to `127.0.0.1` returned `connection refused`, while BIND was listening on `172.16.36.24:53`. This demonstrated that the DNS server's listening address matters: a successful query to one address does not guarantee that the same service accepts queries on every local address.

The BIND9 RNDC reload issue remained unresolved at the end of that session. See Lab 07 for the separate server-administration details.

## 15. Key Commands Reference

| Command                         | Purpose                            |
| ------------------------------- | ---------------------------------- |
| `cat /etc/resolv.conf`          | Inspect DNS resolver configuration |
| `ip route`                      | Inspect the routing table          |
| `ping -4 -c 4 IP`               | Test IPv4 connectivity using ICMP  |
| `getent ahostsv4 HOSTNAME`      | Test system IPv4 name resolution   |
| `nslookup HOSTNAME`             | Test DNS resolution                |
| `nslookup HOSTNAME DNS-IP`      | Query a specific DNS resolver      |
| `dig HOSTNAME A`                | Query an IPv4 A record             |
| `dig @DNS-IP HOSTNAME A`        | Query a specific DNS server        |
| `dig @DNS-IP HOSTNAME A +short` | Show a concise DNS answer          |

Replace placeholder values such as `HOSTNAME`, `DNS-IP`, and `IP` with the appropriate real values.

## 18. Learning Outcomes

This lab provides practice in:

* Understanding DNS resolution.
* Inspecting DNS resolver configuration.
* Distinguishing IP connectivity from name resolution.
* Querying DNS using `getent`, `nslookup`, and `dig`.
* Interpreting common DNS response outcomes.
* Troubleshooting DNS server reachability and unexpected DNS answers.
* Recording test evidence and distinguishing confirmed findings from unresolved issues.
