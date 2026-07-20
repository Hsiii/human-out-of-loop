---
name: solve-issue
description: "Convenience alias for $solve-issues. Accepts an optional positive integer amount, defaults to 1, and runs that many eligible issues end-to-end with no hard maximum. Use when the user says \"solve issue\" or invokes $solve-issue with or without an amount."
---

# Solve Issue

Treat the optional argument as the amount, defaulting to `1`. Follow `$solve-issues <amount>` exactly and preserve any explicit sequential or parallel intent from the request.

This is a user-facing alias only. Implementation and review tasks receive their contracts directly and must never invoke this alias.
