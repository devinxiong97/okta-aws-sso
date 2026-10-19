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

## Step 2 — Turn on AWS Organizations and IAM Identity Center (2026-10-19)

**What I did:** Enabled **IAM Identity Center** as an **organization instance** in **us-east-2 (Ohio)**. That also turned on **AWS Organizations**, which made my account the organization's management account.

**Decisions I made on the setup screen:**

| Choice | What I picked | Why |
|---|---|---|
| Instance type | **Organization instance** | An account instance can't grant access to AWS accounts, only to some applications, so SSO into AWS wouldn't work |
| Region setup | **Single-Region**, not the default Multi-Region | The default encrypts with a customer managed KMS key replicated to a second region, which bills monthly. Single-Region uses an AWS owned key at no cost. Multi-region resilience matters for a company workforce, not a one-person lab |
| Home region | **us-east-2** | Same region as the rest of my AWS work. The home region is effectively permanent |

**What I learned:**
- **Read the defaults before clicking Enable.** The default option would have added a small recurring KMS charge. The warning was on the screen, but easy to miss.
- **Management account best practice:** AWS recommends keeping workloads out of the management account, so a compromised workload can't reach billing or identity controls. My site lives in this account, which is fine for a lab. In a company, workloads go in separate member accounts.
- **Identity source starts as "Identity Center directory,"** meaning AWS stores the users itself. Step 4 switches it to an external identity provider (Okta). That switch is the core of the project: Okta becomes the single source of truth.
- **Account instances can bypass central control.** Member accounts can create their own Identity Center instance unless an SCP prevents it. Added to the roadmap.
- **The AWS access portal URL** (`https://d-xxxxxxxxxx.awsapps.com/start`) is the sign-in page users land on after Okta.

---

## Step 3 — Add the AWS IAM Identity Center app in Okta (2026-10-19)

**What I did:** In the Okta Admin Console, went to **Applications → Browse App Catalog**, found **AWS IAM Identity Center** in the **Okta Integration Network (OIN)**, and added it. The app is **Active**, with no users assigned yet.

**Why the catalog app:** OIN apps are pre-built, vendor-maintained integrations. AWS's already knows the SAML format and SCIM endpoints AWS expects, so there's nothing custom to build or maintain. I skipped two look-alikes:
- **"AWS SSO"** is the old name for the same service.
- **"AWS Account Federation"** is an older method that federates straight into IAM roles in each account. Identity Center is the current, centralized way.

**How the app's tabs map to the plan:**

| Tab | What it controls | Step |
|---|---|---|
| **Sign On** | SAML: the login trust between Okta and AWS | 4 |
| **Provisioning** | SCIM: creating and removing users in AWS automatically | 5 |
| **Assignments** | Which people or groups get the AWS tile | 6 |
| **Push Groups** | Which Okta groups get copied into AWS | 6 |

---

## Step 4 — Exchange SAML metadata (set up the login trust)

*In progress.*
