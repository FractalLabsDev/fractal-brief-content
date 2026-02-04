# Device Fingerprinting for Practice Interviews

## The Problem

We've seen users create multiple accounts to abuse free trials and circumvent our subscription model. This costs us revenue and creates unfair advantages for bad actors.

## The Solution

Device fingerprinting identifies returning devices even when users create new accounts with different emails. It works by collecting non-sensitive browser characteristics (screen size, timezone, installed fonts, etc.) to create a unique "fingerprint" for each device.

## Why Fingerprint Pro ($200/month)

- **99.5% accuracy** — industry-leading identification that persists across incognito mode, VPNs, and browser changes
- **Privacy-compliant** — GDPR/CCPA ready, no cookies required
- **Fraud detection built-in** — detects bots, tampering, and suspicious patterns
- **Simple integration** — JavaScript snippet on frontend, webhook on backend

## What We'll Do With It

1. **Detection** — Flag users who register with a device that's already linked to another account
2. **Logging** — Track fraud attempts for analysis
3. **Future blocking** — If abuse becomes significant, we can auto-block repeat offenders

## ROI

If we prevent even 2-3 subscription bypasses per month, the service pays for itself. Given our trial abuse patterns, the actual savings are likely much higher.

## Privacy Note

We're only using this for fraud prevention, not behavioral tracking. The fingerprint data stays in our database and isn't shared with third parties.
