# 04 — Licensing

This page explains how commercial licensing works for Flink Manager and how to activate your
licence after installation.

> 🌐 **To purchase or request a licence, visit the company website:
> [https://vinteros-labs.com/](https://vinteros-labs.com/)**

## Overview

- Flink Manager is a **commercial** product by **Vinteros Labs** (Luxembourg).
- It is **not open source and not free** — a paid licence is required.
- This repository **does not contain or distribute the software or licence keys**; it is the
  user documentation.

## Obtaining a licence

1. Open **[vinteros-labs.com](https://vinteros-labs.com/)**.
2. Locate the **Flink Manager** product page.
3. For a quick evaluation, get a **free trial licence** directly at
   **[vinteros-labs.com/products/flink-manager/license/](https://vinteros-labs.com/products/flink-manager/license/)** —
   the website issues a testing licence instantly, and Vinteros Labs keeps it renewed
   automatically.
4. For a production licence, follow the enquiry / purchase flow (form or contact details on
   the site); Vinteros Labs will issue your licence as a **licence token** (a compact, signed
   token).

## Activating the licence

Licences are managed in the console under **Configuration → License** (requires the
`LICENSE_MANAGE` permission, granted to the `admin` role by default).

1. Install and start Flink Manager ([02 — Installation](02-installation.md)).
2. Log in and open **Configuration → License**.
3. Paste the licence token into the **License token** field and click **Install license**.
4. The page shows the licence status, who it was issued to, the expiry date and the days
   remaining.

### Licence statuses

| Status | Meaning |
|---|---|
| **Valid** | Licence installed and within its validity period |
| **Expiring soon** | Valid, but expiring within **30 days** (a warning is shown) |
| **Expired** | Validity period has ended — protected operations are locked |
| **Missing** | No licence installed — protected operations are locked |
| **Invalid** | Token malformed, tampered or unsupported |

> Installing an **already-expired** licence is possible but immediately locks the application
> until a valid licence is installed; the console asks for explicit confirmation first.

### What happens without a valid licence

Protected operations are rejected until a valid licence is installed. The console surfaces a
licence warning banner; administrators can still reach the License page to install or renew a
licence.

### Renewal

Paste the new token on the same page — installing a new licence replaces the previous one.

## Support

For licence questions, billing, renewals, or evaluation licences, contact Vinteros Labs via the
website: [https://vinteros-labs.com/](https://vinteros-labs.com/) — or create a ticket there
directly.

For product issues (bugs, unexpected behaviour), see
[06 — Releases → Reporting issues](06-releases.md#reporting-issues): open a GitHub issue or a
ticket on the official website.

## More resources

- [Online documentation](https://vinteros-labs.com/resources/docs/)
- [Changelog](https://vinteros-labs.com/resources/changelog/)
- [Blog](https://vinteros-labs.com/resources/blog/)

---

- [Back to top](../README.md)
