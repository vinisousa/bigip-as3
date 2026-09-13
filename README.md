# bigip-as3

AS3 (F5 BIG-IP Application Services 3) declarations, version-controlled and deployed via GitHub Actions.

## Declarations

| File | Purpose |
| --- | --- |
| `declarations/concavo.local-dns-express.json` | Configures the BIG-IP as the authoritative nameserver for zone `concavo.local` using **DNS Express**, pulling the zone (AXFR) from the back-end authoritative server `192.168.0.1`. |

### concavo.local (DNS Express)

- `DNS_Nameserver` **concavo_backend_ns** → `192.168.0.1:53` (the back-end / "forward lookup" authoritative server the BIG-IP transfers the zone from).
- `DNS_Zone` **concavo.local** → DNS Express enabled, transfers from `concavo_backend_ns`, accepts NOTIFY from `192.168.0.1`.

Once deployed, the BIG-IP answers authoritatively for `concavo.local` from its DNS Express engine. A DNS listener/virtual server with a DNS profile (`dnsExpressEnabled: true`) must exist on the BIG-IP to serve client queries.

## Deploying

Deployment is manual via **Actions → Deploy AS3 to BIG-IP → Run workflow**. It POSTs the chosen declaration to `https://<BIGIP_HOST>/mgmt/shared/appsvcs/declare`.

### Required repository secrets

Set these under **Settings → Secrets and variables → Actions** (or with the `gh` CLI):

| Secret | Value |
| --- | --- |
| `BIGIP_HOST` | `192.168.200.201` |
| `BIGIP_USER` | BIG-IP admin username |
| `BIGIP_PASS` | BIG-IP admin password |

> Secrets are never stored in the repo. The workflow reads them from the encrypted Actions secret store at runtime.

### Deploying manually (without Actions)

```bash
curl -sk -u "$BIGIP_USER:$BIGIP_PASS" \
  -H "Content-Type: application/json" \
  -X POST "https://192.168.200.201/mgmt/shared/appsvcs/declare" \
  -d @declarations/concavo.local-dns-express.json
```

## Requirements

- BIG-IP with the AS3 extension installed (schemaVersion `3.30.0`+).
- BIG-IP DNS (GTM) provisioned for DNS Express.
