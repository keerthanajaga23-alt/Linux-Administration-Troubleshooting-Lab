
# Lab 05: Linux Network Troubleshooting

## 1. Objective

Practice inspecting Linux network interfaces, identifying IPv4 addresses and default gateways, checking routing information, testing connectivity, inspecting listening ports, and troubleshooting basic network problems.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Focus:** IPv4 networking
* **Tools:** `ip`, `ping`, `ss`, `getent`, `dig`, `nslookup`, `curl`

The WSL network configuration can change when WSL restarts. Always check the current configuration instead of assuming that an earlier IP address or gateway is still valid.

## 3. Inspect Network Interfaces

Run:

```bash
ip addr
```

To inspect the primary interface:

```bash
ip addr show eth0
```

**Explanation:**

* `ip` is a Linux networking administration tool.
* `addr` displays interface addresses.
* `eth0` is a common interface name in WSL Linux environments.
* An IPv4 address identifies the host on an IP network.

During an earlier session, the WSL IPv4 address was reported as `172.16.36.24`. Verify the current address before using it in any test.

## 4. Inspect the Routing Table

Run:

```bash
ip route
```

This displays the Linux routing table, including the default route when configured.

Example:

```text
default via 172.16.39.254 dev eth0
```

This means traffic for destinations without a more specific matching route is sent through the next-hop gateway `172.16.39.254` using `eth0`.

**Important terms:**

* **Routing table:** Rules that determine where IP traffic is sent.
* **Default gateway:** The next-hop router used when no more specific route matches.
* **Next hop:** The next network device to which traffic is sent.

Your earlier WSL session showed the gateway `172.16.39.254`. Confirm it with `ip route` each time you need it.

## 5. Understand IPv4 Addresses and Subnets

An IPv4 address contains 32 bits.

Example:

```text
192.168.1.20/24
```

The `/24` prefix means the first 24 bits identify the network. Its dotted-decimal subnet mask is:

```text
255.255.255.0
```

For this example, the network is `192.168.1.0/24`, and addresses in the same subnet can generally communicate directly at the IP layer, subject to the relevant interface, routing, firewall, and network configuration.

**Key terms:**

* **IP address:** Identifies a network interface at the IP layer.
* **Subnet mask/prefix:** Indicates which bits represent the network prefix.
* **Network address:** Identifies the IP subnet.
* **Host address:** Identifies an interface within the subnet.

Use the values shown by your actual network configuration during troubleshooting rather than applying this example to every network.

## 6. Test IPv4 Connectivity

Test a destination IP using IPv4:

```bash
ping -4 -c 4 8.8.8.8
```

**Explanation:**

* `ping` sends ICMP Echo Requests and checks for replies.
* `-4` selects IPv4.
* `-c 4` sends four requests.

A reply indicates that the destination responded to the ICMP test. A failed ping does not conclusively prove that a host or service is unavailable, because ICMP may be blocked or ignored.

## 7. Compare IP Connectivity and Name Resolution

First test an IP address:

```bash
ping -4 -c 4 8.8.8.8
```

Then test a hostname:

```bash
getent ahostsv4 google.com
```

**Explanation:**

* The first command tests IP reachability using ICMP.
* The second asks the system's name-service mechanisms for IPv4 addresses associated with `google.com`.

If IP connectivity works but normal hostname resolution fails, DNS is one possible area to investigate. Also consider routing, firewall rules, proxy settings, and other relevant network configuration as indicated by the evidence.

## 8. Check DNS Resolver Configuration

Inspect the resolver configuration:

```bash
cat /etc/resolv.conf
```

Look for a line similar to:

```text
nameserver 10.255.255.254
```

A `nameserver` entry specifies a DNS resolver address used by the system's resolver configuration.

During an earlier WSL session, this address was `10.255.255.254`. Verify the current value before using it in tests.

Do not edit WSL-generated resolver configuration blindly. First determine how WSL is providing DNS and whether a configuration change would persist.

## 9. Test DNS Using `nslookup` and `dig`

Test name resolution with the configured resolver:

```bash
nslookup google.com
```

Query a specific DNS server:

```bash
nslookup google.com 10.255.255.254
```

Use `dig` to request an IPv4 A record:

```bash
dig google.com A
```

Query a specific DNS server directly:

```bash
dig @10.255.255.254 google.com A
```

The address after `@` identifies the DNS server that should receive the query.

Use the actual current resolver address from `/etc/resolv.conf` rather than assuming the example address is still correct.

## 10. Inspect Listening Ports

Run:

```bash
sudo ss -lntup
```

This displays listening TCP and UDP sockets and, when available, the associated process information.

To inspect HTTP port 80:

```bash
sudo ss -lntp | grep ':80'
```

To inspect SSH port 22:

```bash
sudo ss -lntp | grep ':22'
```

To inspect DNS port 53:

```bash
sudo ss -lntup | grep ':53'
```

**Key terms:**

* **Port:** A transport-layer identifier used by applications.
* **Listening socket:** A socket waiting for incoming traffic.
* **TCP:** A connection-oriented transport protocol.
* **UDP:** A connectionless transport protocol.

A listening port alone does not prove that an application is responding correctly. Follow up with an appropriate application-level test.

## 11. Test HTTP Connectivity

Test the local Nginx endpoint:

```bash
curl -I http://localhost
```

`curl` makes an HTTP request. The `-I` option requests response headers.

This is different from `ping`:

* `ping` tests IP reachability through ICMP.
* `curl` tests application-level HTTP behaviour.

An HTTP response helps verify that an HTTP server is responding locally, although the response status must still be interpreted.

## 12. Troubleshooting Scenario: IP Works but Hostname Fails

### Scenario

An IP connectivity test succeeds, but a hostname cannot be resolved.

### Investigation

1. Check the current resolver configuration:

   ```bash
   cat /etc/resolv.conf
   ```

2. Test normal hostname resolution:

   ```bash
   getent ahostsv4 google.com
   ```

3. Send a DNS query to the configured resolver:

   ```bash
   nslookup google.com
   ```

4. If appropriate, query the identified resolver directly:

   ```bash
   dig @<DNS-IP> google.com A
   ```

   Replace `<DNS-IP>` with the actual resolver address.

5. If the query fails, investigate resolver reachability, DNS service availability, UDP/TCP port 53, firewall rules, and resolver configuration as appropriate.

6. Repeat the name-resolution test after making a justified correction.

## 13. Troubleshooting Scenario: Destination Is Unreachable

### Scenario

A host cannot reach a destination IP address.

### Investigation

Check interface configuration:

```bash
ip addr
```

Check the routing table:

```bash
ip route
```

Test the intended destination:

```bash
ping -4 -c 4 <DESTINATION-IP>
```

Replace `<DESTINATION-IP>` with the destination address.

Then investigate the evidence for an incorrect address or prefix, missing route, unreachable gateway, firewall restriction, or unavailable destination. Do not assume every failed ping is caused by the default gateway.

## 14. Troubleshooting Scenario: Service Is Not Reachable

### Scenario

A website or remote service cannot be accessed.

Check whether the expected service port is listening:

```bash
sudo ss -lntp
```

For a local HTTP service, test:

```bash
curl -I http://localhost
```

For a service running on a remote host, check name resolution, the destination IP, routing, the intended port, firewall or security controls, and the relevant service status and logs.

Use the appropriate service-specific tests instead of relying on a single connectivity command.

## 15. Key Commands Reference

| Command                    | Purpose                                     |
| -------------------------- | ------------------------------------------- |
| `ip addr`                  | Display network interfaces and IP addresses |
| `ip route`                 | Display the routing table                   |
| `ping -4 -c 4 IP`          | Test IPv4 reachability using ICMP           |
| `cat /etc/resolv.conf`     | Inspect resolver configuration              |
| `getent ahostsv4 HOSTNAME` | Check system IPv4 name resolution           |
| `nslookup HOSTNAME`        | Query DNS                                   |
| `dig HOSTNAME A`           | Query an IPv4 A record                      |
| `ss -lntp`                 | Inspect listening TCP sockets               |
| `ss -lntup`                | Inspect listening TCP and UDP sockets       |
| `curl -I URL`              | Request HTTP response headers               |


## 16. Learning Outcomes

This lab provides practice in:

* Inspecting Linux IPv4 network configuration.
* Understanding subnet prefixes and default gateways.
* Checking routes and basic IP reachability.
* Differentiating network connectivity from DNS resolution.
* Inspecting listening ports.
* Testing HTTP behaviour.
* Following an evidence-based network troubleshooting process.
