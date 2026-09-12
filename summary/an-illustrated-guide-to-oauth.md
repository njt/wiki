---
title: "An Illustrated Guide to OAuth"
author: "Aditya Bhargava"
source: "https://www.ducktyped.org/p/an-illustrated-guide-to-oauth"
publication: "DuckTyped (Substack)"
date: 2025-08-25
fetched: 2026-05-14
topics:
  - security-and-sandboxing
---

# An Illustrated Guide to OAuth

**Author:** Aditya Bhargava
**Publication:** DuckTyped (Substack)
**Date:** August 25, 2025

---

## Introduction

OAuth emerged in 2007 when Twitter needed a solution allowing third-party applications to post tweets on users' behalf. The fundamental problem it solved was enabling secure delegation without requiring users to share passwords with untrusted applications.

The naive approach of having users provide their credentials directly to third-party services creates serious security vulnerabilities. Even trustworthy apps might mishandle password storage, exposing users to theft. Similarly, API keys are insufficiently granular for user-specific authorization.

The core innovation of OAuth is the **access token** -- essentially an API key scoped to a specific user. This allows applications to perform actions or access data on a user's behalf without ever handling login credentials.

## How OAuth Works

The guide uses YNAB (a personal finance application) connecting to Chase Bank as its primary example. YNAB helps users track spending and categorize finances without requiring direct access to banking passwords.

### OAuth Flow - User Experience

From a user's viewpoint, the process appears straightforward:

1. YNAB redirects the user to Chase
2. User authenticates with Chase credentials
3. Chase displays a consent screen asking which accounts YNAB should access
4. User selects their checking account and confirms
5. Chase redirects back to YNAB, which now has access

This seemingly simple experience masks complex security mechanisms operating in the background.

### The Critical Security Foundation: Authorization Codes vs. Access Tokens

A fundamental security principle distinguishes OAuth from simpler approaches: **never transmit access tokens through URLs**.

URLs persist in browser history and server logs, creating exposure vectors. Instead, OAuth uses a two-stage process:

**Front-channel communication:** Chase redirects the user back to YNAB with an authorization code embedded in the URL. This authorization code is not itself sensitive.

**Back-channel communication:** YNAB's backend server exchanges this authorization code for an access token through a secure HTTPS POST request. The client secret accompanies this exchange, ensuring only legitimate applications can redeem authorization codes.

This separation ensures access tokens remain encrypted and invisible to users, browsers, and potential attackers.

## OAuth Terminology

- **Resource Owner:** The user whose data is being accessed
- **OAuth Client (or OAuth App):** The third-party application seeking access
- **Authorization Server:** The service where users authenticate and grant permissions (Chase, in the YNAB example)
- **Resource Server:** The API serving protected data (often the same entity as the authorization server)
- **Scopes:** Specific permissions users grant -- defining what data an access token can access

## OAuth Flow - Developer Implementation

### Application Registration

Before initiating OAuth flows, developers must register their applications with the authorization server. This registration requires:

**Required Information:**
- Application name (displayed to users during consent)
- Redirect URI(s) (where users return after authorization)

**Credentials Received:**
- Client ID: A public identifier for API requests
- Client secret: A sensitive credential for backend authentication

The redirect URI registration prevents hijacking attacks. By whitelisting valid redirect domains during registration, authorization servers block malicious redirects to attacker-controlled sites.

### Step-by-Step Implementation

**Step 1 - Initial Redirect to Authorization Server**

The OAuth client redirects users to the authorization server's OAuth endpoint with URL parameters:

- Client ID (publicly known)
- Redirect URI (validated during registration)
- Response type (typically "code" for authorization code flow)
- Requested scopes (defining data access permissions)

The authorization server validates the client ID and confirms the redirect URI matches registered values before displaying any user interface.

**Step 2 - User Consent and Authorization Code**

After user authentication and scope selection, the authorization server redirects to the provided redirect URI with an authorization code appended: `ynab.com/oauth-callback?authorization_code=xyz`

**Step 3 - Backend Token Exchange**

YNAB's backend server exchanges the authorization code for an access token by making a secure HTTPS POST request to Chase's authorization server, including:
- Authorization code (from the redirect)
- Client secret (proof of legitimate application)

Chase validates that the client secret matches the client ID and verifies the authorization code hasn't expired. Upon validation, Chase responds with the access token.

## Front-Channel vs. Back-Channel Communication

OAuth distinguishes between communication types based on visibility:

**Front-channel (GET requests):** Parameters visible in URLs; anyone can observe them. Includes user redirects and authorization code delivery.

**Back-channel (POST requests):** Data encrypted in request bodies; invisible to users and browsers. Used for sensitive exchanges like trading authorization codes for access tokens.

This distinction exists because some applications (single-page applications without backends, mobile apps) cannot safely store client secrets. These use **PKCE** ("pixie"), an alternative authentication mechanism that doesn't require exposing secrets. While less secure than backend-based flows, PKCE enables OAuth for applications lacking dedicated backend infrastructure.

## Security Design Decisions

The OAuth specification reflects numerous hard-learned security lessons. Each component addresses potential exploits:

- **Authorization codes instead of access tokens in URLs:** Prevents token exposure in browser history
- **Client secrets:** Prevents unauthorized redemption of authorization codes
- **Redirect URI validation:** Blocks authorization code interception and redirection attacks
- **Scope limitations:** Restricts token capabilities to requested permissions only
- **Token expiration:** Limits exposure window if tokens are compromised

This layered approach explains OAuth's apparent complexity -- each feature directly addresses specific attack vectors.

## OAuth Variations and Extensions

Various legitimate implementations exist:

**Implicit Flow:** Older approach returning access tokens directly in redirects (less secure, now discouraged)

**Authorization Code Flow with PKCE:** Recommended for browsers and mobile applications

**Token Refresh:** OAuth tokens expire and require refresh tokens for renewal without re-authentication

**OpenID Connect (OIDC):** A layer above OAuth that returns user identity information alongside authorization

**"Sign in with" workflows:** Use OIDC for authentication rather than pure authorization

This diversity explains why OAuth documentation appears inconsistent -- different architectures require different flows.

## Conclusion

OAuth's apparent complexity reflects deliberate security design rather than poor specification. Each component addresses specific vulnerabilities discovered through real-world attacks. Understanding these security principles -- authorization codes, client secrets, back-channel communication, and scope limitation -- reveals why OAuth has become the industry standard for delegation without credential sharing.
