# BIND9 DNS Server Hardening

This project contains a hardened BIND9 configuration designed to reduce the attack surface of a DNS server and prevent common DNS-related attacks.

## Configuration Files

### `named.conf`

This is the main BIND9 configuration file.

Its purpose is simply to load the rest of the configuration files used by the server:

- `named.conf.options`
- `named.conf.local`
- `named.conf.default-zones`
- `named.conf.logging`

The security configuration itself is mainly located in `named.conf.options`.

---

### `named.conf.local`

This file defines our local DNS zone:

```text
grupo5.local
```

It specifies the zone database file used by BIND9 and also disables:

- Zone transfers
- Dynamic DNS updates

This prevents unauthorized users from downloading or modifying the DNS zone.

---

### `db.grupo5.local`

This is the DNS database for the `grupo5.local` zone.

It contains the DNS records used in the project, for example:

- `ns1.grupo5.local`
- `www.grupo5.local`
- `mail.grupo5.local`
- `database.grupo5.local`

Each hostname is associated with an IP address.

---

### `named.conf.logging`

This file configures BIND9 logging.

It separates the logs into different files:

- `bind.log` → general BIND9 activity
- `security.log` → security-related events and zone transfers
- `queries.log` → DNS queries received by the server

Logging makes it easier to monitor the DNS server and detect suspicious activity.

---

# `named.conf.options`

This is the main file used for the BIND9 security hardening.

Most of the security protections implemented in this project are configured here.

## 1. Trusted Clients ACL

```conf
acl "trusted" {
    127.0.0.1;
    192.168.56.0/24;
};
```

An ACL (**Access Control List**) defines which IP addresses or networks are trusted.

In our configuration:

- `127.0.0.1` represents the DNS server itself.
- `192.168.56.0/24` represents our trusted local network.

This ACL is later used to restrict access to DNS recursion and cached information.

---

## 2. Version Hiding

```conf
version none;
hostname none;
server-id none;
```

These options prevent the DNS server from revealing information about itself.

Without this protection, an attacker could identify the BIND version and search for known vulnerabilities affecting that specific version.

This reduces information disclosure during reconnaissance.

---

## 3. Recursion Control

```conf
recursion yes;

allow-recursion { trusted; };
allow-query-cache { trusted; };
allow-query { any; };
```

DNS recursion is enabled, but only trusted clients are allowed to use it.

This prevents the DNS server from becoming an **open resolver**.

An open resolver can be abused in:

- DNS amplification attacks
- DNS reflection attacks
- Cache snooping

Normal DNS queries are still allowed, but recursive queries and cached information are restricted to the trusted network.

---

## 4. Zone Transfer Protection

```conf
allow-transfer { none; };
```

Zone transfers are disabled globally.

A zone transfer allows another DNS server to request a complete copy of a DNS zone.

If this was publicly available, an attacker could obtain information about internal hosts and services.

Disabling zone transfers prevents this type of information disclosure.

---

## 5. Response Rate Limiting

```conf
rate-limit {
    responses-per-second 5;
    window 5;

    exempt-clients {
        127.0.0.1;
        ::1;
    };
};
```

Response Rate Limiting (**RRL**) limits how many similar DNS responses the server sends in a short period of time.

This helps reduce the effectiveness of DNS reflection and amplification attacks.

In our configuration, the server limits repeated responses while excluding localhost from the restriction.

---

## 6. DNSSEC Validation

```conf
dnssec-validation auto;
```

DNSSEC validation allows BIND9 to verify cryptographic signatures included in DNS responses.

This helps detect forged or modified DNS information.

It provides additional protection against attacks such as DNS cache poisoning.

---

## 7. Minimal Responses

```conf
minimal-responses yes;
```

This makes BIND9 send only the necessary information in DNS responses.

Removing unnecessary information:

- Reduces response size
- Reduces information disclosure
- Reduces the amplification factor of DNS attacks

---

## 8. Cache Limits

```conf
max-cache-size 100M;
max-cache-ttl 3600;
max-ncache-ttl 3600;
```

These options limit the resources used by the DNS cache.

The maximum cache size is limited to:

```text
100 MB
```

Cached DNS records are also limited to a maximum lifetime of:

```text
3600 seconds
```

This helps prevent excessive memory consumption and limits how long potentially incorrect cached information can remain stored.

---

# Security Checklist

The following security measures were applied in this project:

- [ ] Restrict recursive DNS queries to trusted clients
- [ ] Use an ACL for trusted clients
- [ ] Disable DNS zone transfers
- [ ] Disable dynamic DNS updates
- [ ] Hide the BIND version, hostname and server ID
- [ ] Enable Response Rate Limiting
- [ ] Enable DNSSEC validation
- [ ] Use minimal DNS responses
- [ ] Limit DNS cache size and TTL
- [ ] Disable unused IPv6 listening
- [ ] Enable query and security logging
- [ ] Define a local test DNS zone
- [ ] Test DNS resolution with `dig`
- [ ] Test that restricted operations are refused

## Final Result

The hardened configuration reduces the amount of information exposed by the DNS server and restricts operations that could be abused by an attacker.

The main differences compared with a default BIND9 configuration are the restriction of recursion, blocked zone transfers, hidden server information, rate limiting and improved logging.
