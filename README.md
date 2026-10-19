# Okta → AWS Single Sign-On (IAM Identity Center)

Signing in to AWS through Okta with MFA, with access controlled by Okta groups and accounts created and removed automatically through SCIM. No AWS passwords, no long-lived access keys.

**Status:** 🚧 In progress. Every step is documented as I go in [docs/build-log.md](docs/build-log.md).

---

## The Problem

Managing AWS access with individual IAM users doesn't scale and isn't safe:
- Every person gets another password and often long-lived access keys
- Offboarding means remembering to delete users in AWS separately
- There's no single place to enforce MFA or review who has access

Companies solve this by making their identity provider, like Okta, the single source of truth. I wanted to build that end to end.

---

## What It Does (when finished)

1. **Single sign-on:** sign in to Okta, approve MFA, click the AWS tile, and land in AWS with the right role.
2. **Group-based access:** Okta groups map to AWS permission sets, like admin or read-only.
3. **Automatic provisioning (SCIM):** add someone to an Okta group and their AWS access appears. Remove them and it's gone.
4. **MFA enforced at the front door:** Okta requires MFA before AWS is ever reached.

---

## Architecture

```
User ──▶ Okta (login + MFA) ──SAML──▶ AWS IAM Identity Center ──▶ AWS account (permission set)
              │
              └────────SCIM (users + groups)────────▶ IAM Identity Center
```

| Piece | What it does |
|---|---|
| **Okta** | Identity provider (IdP): who you are, your password, your MFA, your groups |
| **SAML** | The standard that lets Okta vouch for you to AWS, so AWS never sees your password |
| **SCIM** | The standard Okta uses to create, update, and deactivate users and groups in AWS automatically |
| **AWS IAM Identity Center** | AWS's single sign-on service. Receives the SAML login and holds the permission sets |
| **Permission set** | A role template, like `ReadOnlyAccess`, assigned to a group for an AWS account |
| **AWS Organizations** | Required for Identity Center to grant access to AWS accounts. Free |

---

## Plan

| Step | What | Status |
|---|---|---|
| 0 | Get a business email for the Okta signup | ✅ Done |
| 1 | Create a free Okta developer org | ✅ Done |
| 2 | Turn on AWS Organizations and IAM Identity Center | ✅ Done |
| 3 | Add the AWS IAM Identity Center app in Okta | ⏳ Next |
| 4 | Exchange SAML metadata (set up the login trust) | |
| 5 | Turn on SCIM provisioning | |
| 6 | Create Okta groups and push them to AWS | |
| 7 | Create permission sets and assign them to groups | |
| 8 | Enforce MFA, test login, and test offboarding | |

---

## Security Choices

- **A separate Okta org just for this project.** A personal AWS account should never be connected to an employer's identity provider.
- **Least privilege from day one.** Read-only is the default; admin is a separate group.
- **Offboarding is part of the test plan,** not an afterthought. Removing a user in Okta must remove their AWS access.

**Next ideas:** an SCP that blocks member accounts from creating their own Identity Center instances, and a dedicated member account for workloads.

**Cost:** $0. Okta's free developer plan, AWS Organizations, and IAM Identity Center are all free.
