---
name: noma-nextjs-auth
description: >-
  Implements Noma auth in Next.js (App Router), including NextAuth integration,
  token lifecycle, and correct backend logout behavior.
---

# Noma + Next.js Auth

Use this skill for Next.js auth setup, migration, or review.

Always follow `noma-auth-contract` first.

## Approved architectures

- NextAuth with custom credentials provider (recommended)
- Custom auth client (allowed if contract checklist passes)

## Mandatory implementation rules

- Login must exchange credentials with Noma backend and persist tokens safely.
- Logout must call Noma backend revoke before local session clear.
- NextAuth sign-out path must include backend revoke.
- Refresh token rotation must be implemented in callbacks or server route.

## Required logout sequence

1. Read current Noma access token in server-safe context.
2. Call backend logout endpoint with bearer token.
3. If successful, clear NextAuth/local session.
4. If it fails, return actionable error and keep state consistent.

## PR review checklist

- No server secrets exposed via `NEXT_PUBLIC_`.
- Backend logout revoke exists and is used.
- Refresh flow handles expired access token.
- No token leakage in logs.
- Private routes protected in middleware.
- Session duplication behavior documented for team.

## Common failure pattern

Logout only clears local cookie/state, old backend session remains active, and next login creates another active session.

Fix by revoking backend session during logout.

## Deliverables for AI agent

- List of changed files.
- Exact location of backend revoke call.
- Login/refresh/logout sequence summary.
- Manual verification:
  - login -> logout -> login
  - confirm expected active sessions in Noma auth sessions page

## References

- `noma-auth-contract`
- `noma-user-auth`
- `noma-nextjs`
- Canonical templates: [examples.md](examples.md)
- Definition of done and test matrix: [reference.md](reference.md)
