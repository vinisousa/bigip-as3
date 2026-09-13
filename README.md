# bigip-as3

AS3 (F5 BIG-IP Application Services 3) declarations, version-controlled and deployed via GitHub Actions.

## Declarations

| File | Purpose |
| --- | --- |
| `declarations/dns-services.json` | BIG-IP DNS services for `concavo.local`, `example.com`, and internet forwarding, plus the DNS listener. |

### dns-services.json

Tenant `Sample_DNS_01`, application `DNS_App`:

- **`concavo.local` (authoritative, local)** — a DNS Cache *local zone* (type `static`). Authoritative locally, no AXFR/zone transfer. Currently has no records — add A/AAAA entries to its `records` array as needed.
- **`example.com` (authoritative, local)** — a DNS Cache *local zone* (type `static`) with `cafe.example.com → 192.168.100.32`.
- **Internet forwarding** — DNS Cache (type `resolver`) forward zone `.` → `192.168.0.1:53` for all other queries.
- **Listener** — UDP and TCP virtual servers on `192.168.100.32:53` bound to `dns_profile` (which references the cache).

Query resolution order: local zones (`concavo.local`, `example.com`) → forward everything else to `192.168.0.1`.

## Deploying

Manual via **Actions → Deploy AS3 to BIG-IP → Run workflow** (default path `declarations/dns-services.json`). POSTs to `https://<BIGIP_HOST>/mgmt/shared/appsvcs/declare`.

### Required repository secrets

| Secret | Value |
| --- | --- |
| `BIGIP_HOST` | `192.168.200.201` |
| `BIGIP_USER` | BIG-IP admin username |
| `BIGIP_PASS` | BIG-IP admin password |

> Secrets live only in the encrypted Actions secret store, never in the repo.

### Deploying manually (without Actions)

```bash
curl -sk -u "$BIGIP_USER:$BIGIP_PASS" \
  -H "Content-Type: application/json" \
  -X POST "https://192.168.200.201/mgmt/shared/appsvcs/declare" \
  -d @declarations/dns-services.json
```

## Requirements

- BIG-IP with the AS3 extension installed (schemaVersion `3.30.0`+).
- BIG-IP DNS (GTM) provisioned (DNS caching, DNS listeners).
