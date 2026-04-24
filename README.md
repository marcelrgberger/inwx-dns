# inwx-dns

A Claude Code plugin for managing domains and DNS records via the [INWX](https://www.inwx.de) domrobot XML-RPC API.

## Installation

```bash
# Add marketplace
claude plugins marketplace add marcelrgberger/inwx-dns

# Install plugin
claude plugins install inwx-dns
```

Requires Python 3 and the `inwx-domrobot` package (installed automatically when needed).

## What It Does

This plugin gives Claude Code full access to the INWX registrar API. Ask Claude to manage your domains and DNS in natural language.

### Domain Management

- **Registration** — register new domains across 1000+ TLDs
- **Transfer** — transfer domains from other registrars with auth code handling
- **Renewal** — renew domains, change renewal mode (auto-renew, auto-expire)
- **Status** — check domain status, expiry dates, delegation, transfer locks
- **WHOIS** — look up domain ownership information
- **Pricing** — check domain prices, promotions, and TLD rules
- **Trade** — change domain registrant (owner change)

### DNS Record Management

- **Full CRUD** — create, read, update, delete DNS records
- **All record types** — A, AAAA, CNAME, MX, TXT, SRV, CAA, TLSA
- **Wildcard records** — set up wildcard DNS for Kubernetes, multi-tenant apps
- **URL redirects** — 301/302 redirects and frame forwarding
- **Zone import/export** — BIND format zone file import and export
- **Bulk operations** — manage records across multiple domains

### Additional Features

- **Contact management** — create and manage PERSON, ORG, and ROLE contacts
- **DNSSEC** — sign/unsign zones, manage DNSKEY records
- **DynDNS** — create and manage dynamic DNS accounts
- **SSL certificates** — order, renew, and manage SSL/TLS certificates
- **Nameserver sets** — reusable nameserver configurations
- **Accounting** — check balance, view invoices, transaction history

## Usage Examples

```
> Transfer example.com from GoDaddy to INWX
> Set up wildcard DNS for *.app.example.com pointing to 1.2.3.4
> Create A record for api.example.com
> Check which domains are expiring in the next 30 days
> List all DNS records for example.com
> Set up MX records for iCloud Mail on example.de
> Add DNSSEC to example.com
> Create a 301 redirect from example.de to example.com
> Check if coolstartup.app is available
> What does a .app domain cost?
```

## Authentication

The plugin asks for your INWX credentials when needed. If 2FA is enabled, you'll be asked for a one-time code from your authenticator app.

Credentials are never stored — they're only used within the current conversation.

## Components

| Component | Type | Description |
|-----------|------|-------------|
| `inwx-dns` | Skill | Complete INWX API reference with code examples |
| `inwx-manager` | Agent | Specialized subagent for domain and DNS operations |

## API Coverage

| API Group | Methods | Description |
|-----------|---------|-------------|
| Account | 13 | Login, logout, 2FA, sub-accounts, roles |
| Accounting | 11 | Balance, invoices, statements, refunds |
| Domain | 26 | Registration, transfer, renewal, pricing, WHOIS |
| Nameserver | 13 | Zones, records, import/export, bulk operations |
| Contact | 7 | PERSON, ORG, ROLE contact management |
| DNSSEC | 7 | Key management, zone signing |
| DynDNS | 8 | Dynamic DNS accounts |
| Certificate | 13 | SSL/TLS certificate lifecycle |
| Hosting | 9 | Web hosting management |
| Nameserver Set | 5 | Reusable NS configurations |

## Requirements

- Python 3.8+
- `inwx-domrobot` package (auto-installed)
- INWX account ([inwx.de](https://www.inwx.de))

## License

MIT
