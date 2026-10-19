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

## Step 1 — Create a free Okta developer org (2026-10-19)

**What I did:** Signed up for Okta's free **Integrator** plan with `devin@devinxiong.com`, activated it from the email (which arrived through the forwarding from Step 0), and set up **Okta Verify** for MFA. My org is `devinxiong-integrator-5984483`.

**What I noticed:**
- **An Okta "org" is a private Okta instance** where I'm the super admin, the same thing a company runs, just empty.
- **Two different sides:** the **Admin Console** (managing users, groups, apps, and policies) and the **end-user dashboard** (where people click app tiles to sign in). I'll configure in one and test from the other.
- **The landing page is for app developers.** It pushes "Create your first Okta app," which is for building a new integration. I don't need that. AWS already has a ready-made app in Okta's catalog (the Okta Integration Network), which I'll use in Step 3.
- **MFA was required before I could get in.** Okta is secure by default, and that's the same behavior I'll enforce for AWS in Step 8.

**Why a separate org:** a personal AWS account should never be connected to an employer's identity provider. A separate org keeps the blast radius at zero.

---

## Step 2 — Turn on AWS Organizations and IAM Identity Center

*In progress.*
