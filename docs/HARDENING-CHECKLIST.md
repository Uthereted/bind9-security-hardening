# BIND9 Hardening Checklist

## 1. System

- [ ] Keep BIND9 updated
- [ ] Run BIND using a dedicated user
- [ ] Apply least privilege
- [ ] Configure firewall rules

## 2. DNS Configuration

- [ ] Restrict recursion
- [ ] Restrict zone transfers
- [ ] Hide BIND version
- [ ] Disable unnecessary features
- [ ] Configure authoritative and recursive roles appropriately

## 3. Network Security

- [ ] Restrict TCP/UDP port 53
- [ ] Allow queries only from required networks
- [ ] Prevent open resolver configuration

## 4. DNS Security

- [ ] Consider DNSSEC
- [ ] Protect zone transfers
- [ ] Protect dynamic updates

## 5. Monitoring

- [ ] Enable appropriate logging
- [ ] Monitor unusual DNS traffic
- [ ] Monitor failed queries and transfers
