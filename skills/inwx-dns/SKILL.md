---
name: inwx-dns
description: Manage INWX domains and DNS records via the domrobot XML-RPC API. Use when the user wants to create, update, delete, or query DNS records, transfer domains, register domains, manage contacts, handle DNSSEC, manage certificates, check domain availability or pricing, set up DynDNS, or check account/billing status at INWX. Triggers on "inwx", "dns record", "domain transfer", "nameserver", "dns zone", "wildcard record", "domain status", "domain registration", "dnssec", "dyndns", "ssl certificate".
version: 1.0.0
---

# INWX API — Complete Reference

## Prerequisites

```bash
pip3 install --user --break-system-packages inwx-domrobot 2>/dev/null || pip3 install inwx-domrobot
```

## Authentication

```python
from INWX.Domrobot import ApiClient

api = ApiClient(api_url=ApiClient.API_LIVE_URL)

# Without 2FA
api.login('username', 'password')

# With 2FA (6-digit code from authenticator app — expires in 30 seconds)
api.login('username', 'password', tfa_token='123456')

# With shared secret (auto-generates TOTP, no manual code needed)
api.login('username', 'password', shared_secret='BASE32SECRET')
```

**Important:** 2FA codes expire in 30 seconds. Execute login and all API calls in a single script.

**Credentials:** Always ask the user. Never store in code. Reuse from conversation if previously provided.

**Test environment:** Use `ApiClient.API_OTE_URL` instead of `API_LIVE_URL`.

---

## Account API

```python
# Login
api.call_api('account.login', {'user': 'username', 'pass': 'password'})

# Unlock 2FA
api.call_api('account.unlock', {'tan': '123456'})

# Logout
api.call_api('account.logout', {})

# Account info
api.call_api('account.info', {})

# List sub-accounts
api.call_api('account.list', {})

# Create sub-account
api.call_api('account.create', {
    'user': 'subuser', 'pass': 'password', 'email': 'sub@example.com',
    'firstname': 'John', 'lastname': 'Doe',
})

# Update account
api.call_api('account.update', {'email': 'new@example.com'})

# Change password
api.call_api('account.changepassword', {'oldpass': 'old', 'newpass': 'new'})

# Delete sub-account
api.call_api('account.delete', {'user': 'subuser'})

# Check if username exists
api.call_api('account.check', {'user': 'testuser'})

# Role management
api.call_api('account.addrole', {'user': 'subuser', 'role': 'DOMAIN_ADMIN'})
api.call_api('account.removerole', {'user': 'subuser', 'role': 'DOMAIN_ADMIN'})
api.call_api('account.getroles', {'user': 'subuser'})
```

---

## Accounting API

```python
# Account balance
api.call_api('accounting.accountBalance', {})

# Credit log (deposits/charges)
api.call_api('accounting.log', {})

# Specific credit log entry
api.call_api('accounting.creditLogById', {'id': 12345})

# Locked funds (pending transactions)
api.call_api('accounting.lockedFunds', {})

# Get invoice
api.call_api('accounting.getInvoice', {'invoiceId': 12345})

# List invoices
api.call_api('accounting.listInvoices', {})

# Preview invoice
api.call_api('accounting.getPreviewInvoice', {})

# Get receipt
api.call_api('accounting.getReceipt', {'receiptId': 12345})

# Get statement
api.call_api('accounting.getstatement', {'from': '2024-01-01', 'to': '2024-12-31'})

# Refund
api.call_api('accounting.refund', {'transactionId': 12345})
```

---

## Domain API

### Registration & Lifecycle

```python
# Check availability
res = api.call_api('domain.check', {'domain': 'example.com'})
# avail: 1 = available, 0 = taken

# Check multiple domains
res = api.call_api('domain.check', {'domain': ['example.com', 'example.de', 'example.app']})

# Register domain
api.call_api('domain.create', {
    'domain': 'example.com',
    'period': '1Y',           # 1Y, 2Y, ... 10Y
    'ns': ['ns.inwx.de', 'ns2.inwx.de', 'ns3.inwx.eu'],
    'registrant': CONTACT_ID,
    'admin': CONTACT_ID,
    'tech': CONTACT_ID,
    'billing': CONTACT_ID,
})

# Renew domain
api.call_api('domain.renew', {'domain': 'example.com', 'period': '1Y'})

# Delete domain (schedule for deletion at expiry)
api.call_api('domain.delete', {'domain': 'example.com'})

# Restore deleted domain (redemption period)
api.call_api('domain.restore', {'domain': 'example.com'})
```

### Transfer

```python
# Initiate inbound transfer
api.call_api('domain.transfer', {
    'domain': 'example.com',
    'authCode': 'AUTH-CODE',
    'ns': ['ns.inwx.de', 'ns2.inwx.de', 'ns3.inwx.eu'],
    'registrant': CONTACT_ID,
    'admin': CONTACT_ID,
    'tech': CONTACT_ID,
    'billing': CONTACT_ID,
})

# Cancel pending transfer
api.call_api('domain.transfercancel', {'domain': 'example.com'})

# Approve outbound transfer
api.call_api('domain.transferOut', {'domain': 'example.com'})

# Push domain to another INWX account
api.call_api('domain.push', {'domain': 'example.com', 'target': 'other-user'})
```

**Transfer notes:**
- `.de` domains: tech and billing contacts must be PERSON type (not ORG)
- `clientTransferProhibited`: transfer lock must be removed at current registrar
- Transfer costs one year of registration (extends expiry by one year)
- `.de` transfers are near-instant; gTLDs (.com, .app) take up to 5 days

### Domain Info & Management

```python
# Domain info
res = api.call_api('domain.info', {'domain': 'example.com'})
# Keys: status, exDate, renewalMode, noDelegation, registrant, admin, tech, billing, ns, authCode

# List all domains
res = api.call_api('domain.list', {})
# Optional filters: status, renewalMode, domain (pattern)

# Update domain
api.call_api('domain.update', {
    'domain': 'example.com',
    'renewalMode': 'AUTOEXPIRE',  # AUTORENEW, AUTOEXPIRE, AUTODELETE
    'ns': ['ns.inwx.de', 'ns2.inwx.de', 'ns3.inwx.eu'],
    'transferLock': True,
})

# Trade (change registrant for gTLDs)
api.call_api('domain.trade', {
    'domain': 'example.com',
    'registrant': NEW_CONTACT_ID,
})

# WHOIS lookup
api.call_api('domain.whois', {'domain': 'example.com'})

# Domain log (history)
api.call_api('domain.log', {'domain': 'example.com'})

# Domain stats
api.call_api('domain.stats', {})

# Set/remove client hold
api.call_api('domain.setClientHold', {'domain': 'example.com'})
api.call_api('domain.removeClientHold', {'domain': 'example.com'})
```

### Pricing

```python
# Get price for specific domain
api.call_api('domain.getdomainprice', {'domain': 'example.com'})

# Get all domain prices
api.call_api('domain.getalldomainprices', {})

# Get pricing rules
api.call_api('domain.getRules', {'tld': 'com'})

# Price changes
api.call_api('domain.priceChanges', {})

# Current promotions
api.call_api('domain.getPromos', {})

# TLD groups
api.call_api('domain.getTldGroups', {})

# Extra data rules (TLD-specific requirements)
api.call_api('domain.getextradatarules', {'tld': 'de'})
```

---

## Nameserver (DNS) API

### Zone Management

```python
# Create zone
api.call_api('nameserver.create', {
    'domain': 'example.com',
    'type': 'MASTER',          # MASTER or SLAVE
    'ns': ['ns.inwx.de', 'ns2.inwx.de', 'ns3.inwx.eu'],
})

# Zone info (lists all records)
res = api.call_api('nameserver.info', {'domain': 'example.com'})
for r in res['resData']['record']:
    print(r['id'], r['type'], r['name'], r['content'], r['ttl'])

# List all zones
res = api.call_api('nameserver.list', {})

# Update zone
api.call_api('nameserver.update', {
    'domain': 'example.com',
    'type': 'MASTER',
})

# Delete zone
api.call_api('nameserver.delete', {'domain': 'example.com'})

# Check zone
api.call_api('nameserver.check', {'domain': 'example.com'})

# Export zone (BIND format)
api.call_api('nameserver.export', {'domain': 'example.com'})

# Import zone (BIND format)
api.call_api('nameserver.import', {
    'domain': 'example.com',
    'content': '$ORIGIN example.com.\n@ IN A 1.2.3.4',
})
```

### Record Management

```python
# Create record
api.call_api('nameserver.createRecord', {
    'domain': 'example.com',
    'type': 'A',              # A, AAAA, CNAME, MX, TXT, SRV, CAA, TLSA, URL
    'name': 'www',            # subdomain, '@' for root, '*' for wildcard
    'content': '1.2.3.4',
    'ttl': 300,
})

# MX record (with priority)
api.call_api('nameserver.createRecord', {
    'domain': 'example.com',
    'type': 'MX',
    'name': '@',
    'content': 'mx1.example.com',
    'ttl': 3600,
    'prio': 10,
})

# SRV record
api.call_api('nameserver.createRecord', {
    'domain': 'example.com',
    'type': 'SRV',
    'name': '_sip._tcp',
    'content': '0 5060 sip.example.com',
    'ttl': 3600,
    'prio': 10,
})

# CAA record
api.call_api('nameserver.createRecord', {
    'domain': 'example.com',
    'type': 'CAA',
    'name': '@',
    'content': '0 issue "letsencrypt.org"',
    'ttl': 3600,
})

# URL redirect (301 permanent)
api.call_api('nameserver.createRecord', {
    'domain': 'example.com',
    'type': 'URL',
    'name': '@',
    'content': 'https://target.com',
    'ttl': 300,
    'urlRedirectType': 'HEADER301',  # HEADER301, HEADER302, FRAME
})

# Update record
api.call_api('nameserver.updateRecord', {
    'id': RECORD_ID,
    'content': '5.6.7.8',
    'ttl': 600,
})

# Delete record
api.call_api('nameserver.deleteRecord', {'id': RECORD_ID})
```

### Bulk Record Operations

```python
# Delete all records of a type
res = api.call_api('nameserver.info', {'domain': 'example.com'})
for r in res['resData']['record']:
    if r['type'] == 'A' and '*.app' in r['name']:
        api.call_api('nameserver.deleteRecord', {'id': r['id']})
```

---

## Nameserver Set API

```python
# Create a reusable nameserver set
api.call_api('nameserverset.create', {
    'name': 'default-ns',
    'ns': ['ns.inwx.de', 'ns2.inwx.de', 'ns3.inwx.eu'],
})

# List sets
api.call_api('nameserverset.list', {})

# Info
api.call_api('nameserverset.info', {'id': SET_ID})

# Update set
api.call_api('nameserverset.update', {'id': SET_ID, 'ns': ['ns.inwx.de', 'ns2.inwx.de']})

# Delete set
api.call_api('nameserverset.delete', {'id': SET_ID})
```

---

## Contact API

```python
# List contacts
res = api.call_api('contact.list', {})
for c in res['resData']['contact']:
    print(c['id'], c['type'], c['name'], c.get('org'), c['email'])

# Contact info
api.call_api('contact.info', {'id': CONTACT_ID})

# Create PERSON contact
api.call_api('contact.create', {
    'type': 'PERSON',
    'name': 'John Doe',
    'street': 'Main St 1',
    'city': 'Berlin',
    'pc': '10115',
    'cc': 'DE',             # ISO 3166 country code
    'voice': '+49.301234567',
    'email': 'john@example.com',
})

# Create ORG contact
api.call_api('contact.create', {
    'type': 'ORG',
    'name': 'John Doe',
    'org': 'ACME Corp',
    'street': 'Main St 1',
    'city': 'Berlin',
    'pc': '10115',
    'cc': 'DE',
    'voice': '+49.301234567',
    'email': 'info@acme.com',
})

# Update contact
api.call_api('contact.update', {'id': CONTACT_ID, 'email': 'new@example.com'})

# Delete contact
api.call_api('contact.delete', {'id': CONTACT_ID})

# Check contact (validate handle)
api.call_api('contact.check', {'id': CONTACT_ID})
```

---

## DNSSEC API

```python
# List DNSSEC keys for domain
api.call_api('dnssec.list', {'domain': 'example.com'})

# Info about specific key
api.call_api('dnssec.info', {'id': KEY_ID})

# Create DNSSEC key
api.call_api('dnssec.create', {
    'domain': 'example.com',
    'flags': 257,          # 256=ZSK, 257=KSK
    'protocol': 3,
    'algorithm': 13,       # 13=ECDSAP256SHA256
    'publicKey': 'BASE64KEY...',
})

# Update DNSSEC key
api.call_api('dnssec.update', {'id': KEY_ID, 'publicKey': 'NEWBASE64KEY...'})

# Delete DNSSEC key
api.call_api('dnssec.delete', {'id': KEY_ID})

# Sign zone
api.call_api('dnssec.sign', {'domain': 'example.com'})

# Unsign zone
api.call_api('dnssec.unsign', {'domain': 'example.com'})
```

---

## DynDNS API

```python
# Create DynDNS account
api.call_api('dyndns.create', {
    'domain': 'example.com',
    'name': 'home',           # subdomain
    'type': 'A',
    'content': '1.2.3.4',
    'user': 'dynuser',
    'pass': 'dynpass',
})

# List DynDNS accounts
api.call_api('dyndns.list', {})

# Info
api.call_api('dyndns.info', {'id': DYNDNS_ID})

# Update
api.call_api('dyndns.update', {'id': DYNDNS_ID, 'content': '5.6.7.8'})

# Delete
api.call_api('dyndns.delete', {'id': DYNDNS_ID})

# Enable/disable
api.call_api('dyndns.enable', {'id': DYNDNS_ID})
api.call_api('dyndns.disable', {'id': DYNDNS_ID})
```

**DynDNS update URL:** `https://dyndns.inwx.com/nic/update?myip=NEW_IP`

---

## Certificate (SSL/TLS) API

```python
# List certificates
api.call_api('certificate.list', {})

# Certificate info
api.call_api('certificate.info', {'id': CERT_ID})

# Create certificate order
api.call_api('certificate.create', {
    'product': 'COMODO_POSITIVE_SSL',
    'csr': '-----BEGIN CERTIFICATE REQUEST-----...',
    'period': '1Y',
    'approverEmail': 'admin@example.com',
})

# Renew certificate
api.call_api('certificate.renew', {'id': CERT_ID, 'period': '1Y'})

# Delete/cancel certificate
api.call_api('certificate.delete', {'id': CERT_ID})

# Reissue certificate
api.call_api('certificate.reissue', {'id': CERT_ID, 'csr': '...'})

# Resend approver email
api.call_api('certificate.resendApproverEmail', {'id': CERT_ID})

# Get certificate products/prices
api.call_api('certificate.getProducts', {})
```

---

## Hosting API

```python
# List hosting accounts
api.call_api('hosting.list', {})

# Hosting info
api.call_api('hosting.info', {'id': HOSTING_ID})

# Create hosting
api.call_api('hosting.create', {
    'product': 'HOSTING_BASIC',
    'domain': 'example.com',
})

# Update hosting
api.call_api('hosting.update', {'id': HOSTING_ID, 'product': 'HOSTING_PRO'})

# Delete hosting
api.call_api('hosting.delete', {'id': HOSTING_ID})
```

---

## Message API

```python
# List messages/notifications
api.call_api('message.list', {})

# Mark message as read
api.call_api('message.update', {'id': MSG_ID, 'read': True})
```

---

## Wildcard DNS Records

INWX supports wildcard records. `*` matches all subdomains at that level:

- `*.example.com` matches `foo.example.com` but NOT `bar.foo.example.com`
- `*.sub.example.com` matches `foo.sub.example.com`

For multi-level wildcards (e.g., Kubernetes environments):

```python
wildcards = ['*.app', '*.dev.app', '*.test.app', '*.prod.app']
for name in wildcards:
    api.call_api('nameserver.createRecord', {
        'domain': 'example.com',
        'type': 'A',
        'name': name,
        'content': INGRESS_IP,
        'ttl': 300,
    })
```

---

## Response Codes

| Code | Meaning |
|------|---------|
| 1000 | Success |
| 1001 | Success, action pending |
| 2001 | Internal server error |
| 2002 | Command use error (e.g., invalid 2FA) |
| 2003 | Required parameter missing |
| 2004 | Parameter value range error |
| 2005 | Parameter value syntax error |
| 2104 | Billing failure (insufficient credit) |
| 2200 | Authentication error |
| 2201 | Authorization error |
| 2302 | Object already exists |
| 2303 | Object does not exist |
| 2304 | Object status prohibits operation |
| 2306 | Parameter value policy error |

---

## Best Practices

1. **Always logout** after API calls: `api.logout()`
2. **Batch operations** in a single login session to minimize 2FA prompts
3. **Check response codes** before reporting success
4. **Use INWX nameservers**: `ns.inwx.de`, `ns2.inwx.de`, `ns3.inwx.eu`
5. **Activate delegation** after domain transfer via `domain.update` with NS records
6. **Prefer wildcards** over individual A records when subdomains share the same target
7. **Handle billing failures** by informing the user to top up their INWX credit
8. **.de domains** require PERSON contacts for tech and billing (not ORG)
