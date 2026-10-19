# Build Log

Every step of connecting Okta to AWS, in order: what I did, why, the technical detail, and what I learned.

---

## Step 0 — A business email for the Okta signup (2026-10-19)

**The problem:** Okta's developer signup only accepts a business email, and I only had personal Gmail and Outlook addresses.

**My reasoning:** I own `devinxiong.com`. A business email is just an address on a domain you control, so I could create one for free instead of paying for a mailbox.

**What I did:** Set up **Cloudflare Email Routing** so `devin@devinxiong.com` forwards to my Gmail. Cloudflare added the MX and SPF records, I verified Gmail as the destination, created the routing rule, and confirmed a test email arrived.

**Technical detail:**
- **MX records** tell the internet where to deliver mail for a domain.
- **Email Routing is receive-only.** That's all an account signup needs: the verification link.

**Lesson:** check what you already own before buying something new. Full write-up in the [devinxiong.com build log](https://github.com/devinxiong97/devinxiong.com/blob/main/docs/build-log.md).

---

## Step 1 — Create a free Okta developer org

*In progress.*
