# CIPHERVAULT — README

A single-file, client-side password manager. Everything runs in your browser; there's no server, no account, and no API keys involved. This doc covers how to use it safely and what to avoid.

> ⚠ **Experimental / testing build.** Not audited. Don't make this your only copy of any password — keep the encrypted backup (see below) somewhere else too.

---

## Quick Start

1. Open `cipher_vault.html` in a modern browser (Chrome, Edge, or Firefox recommended).
2. First time: choose a master password (8+ characters). This password is **never stored anywhere** — if you forget it, your data cannot be recovered by anyone, including you.
3. That's it — you're in the vault.

**To use it again later**, just reopen the same file in the same browser on the same device. Your data persists automatically between sessions.

---

## What It Stores

- **Logins** — username, password, URL, notes
- **Cards** — number, expiry, CVV, cardholder
- **Secure notes** — free text
- **Recovery codes** — one service, many one-time codes
- **2FA / TOTP** — generates live 6-digit codes from a secret you paste in
- **Passkey log** — a *reference record* only (site + account), not the actual passkey. Real passkeys live in your browser/OS/security key and can't be exported by any tool, including this one — that's by design, not a limitation of this app specifically.

Every item can also have a **folder** (one) and **tags** (many) for organizing.

---

## Security Model (what's actually protecting your data)

- **Key derivation:** Argon2id (memory-hard, OWASP baseline params) for new vaults, with an automatic silent upgrade path for older vaults. If Argon2id can't load (no internet on first use), it falls back to PBKDF2 at 600,000 rounds — still strong, just not the newer algorithm.
- **Encryption:** AES-256-GCM for the vault contents.
- **Where it's stored:** the encrypted blob lives in the browser's app-level storage, tied to your account on this device. It is not readable without your master password — the app itself never has a "backdoor" into it.
- **Auto-lock:** clears the decryption key from memory after inactivity (configurable), and optionally on tab-switch/window-blur too.
- **Clipboard:** copied passwords auto-clear after a configurable delay.

### Honest limitations
- This is a hobby project, not a professionally audited security product.
- Argon2id and the strength meter (zxcvbn) load from a CDN. If you're fully offline the first time you use a feature, it falls back gracefully rather than breaking — but it does mean a working connection is needed at least once.
- Biometric unlock relies on your browser/authenticator supporting the WebAuthn `prf` extension. Not all browsers do yet (Safari doesn't as of writing). If it's not supported, you'll just see no biometric option — nothing breaks, password unlock always works.
- "Memory zeroization" is best-effort. JavaScript can't guarantee true memory erasure (garbage collection can leave copies) — this hardens what it can, but isn't a cryptographic guarantee.

---

## ⚠ Things That Will Actually Break Things

- **Don't open the same vault in two tabs/windows and edit in both.** Storage writes don't merge — the last save wins, and the other tab's changes can be silently lost. Use one tab at a time.
- **Don't clear your browser's site data for this page** unless you've exported a backup first — that's how the vault is stored, and clearing it deletes the vault.
- **Don't lose your master password.** There is no reset, no recovery email, no "forgot password" — that's the whole point of it being real encryption. This is exactly what the **Emergency Kit** (Settings → Emergency Kit) is for: print it, hand-write your password on the paper copy, and store that somewhere physically secure.
- **If you switch browsers or devices, your vault doesn't come with you automatically.** Export an encrypted backup (Settings → Backup) and import it on the new device/browser.

---

## Backups (do this regularly)

Two different export options exist — use the right one:

| | Encrypted Backup | CSV Export |
|---|---|---|
| **Where** | Settings → Backup | Settings → Transfer Passwords (CSV) |
| **Format** | Still encrypted — useless without your master password | Plaintext — anyone who opens the file can read every password |
| **Use for** | Personal backups, moving to a new device | Moving logins into a *different* password manager |
| **What's included** | Everything (logins, cards, notes, recovery codes, 2FA) | Logins only |

**After a CSV import/export, delete the CSV file from your downloads folder.** The app reminds you of this after every CSV import for the same reason.

---

## Settings Reference

- **Auto-lock** — how long the vault stays unlocked with no activity
- **Lock on focus loss** — locks immediately when you switch tabs/windows, not just after the timer
- **Clipboard auto-clear** — how long a copied password stays on the clipboard
- **Biometric unlock** — enable/disable fingerprint/face/security-key unlock (if your browser supports it)
- **Master password** — change it any time; re-encrypts everything under the new password
- **Trash** — deleted items sit here 30 days before permanent removal; restore anything deleted by mistake here

---

## Security Audit Tab

Run from the sidebar. Checks for:
- **Reused passwords** — fully local, no network call
- **Weak passwords** — via zxcvbn (or a local heuristic if that hasn't loaded)
- **Breached passwords** — opt-in button, calls the free HIBP API (`api.pwnedpasswords.com`). Only the first 5 characters of a SHA-1 hash are sent — your actual password never leaves the browser — but it does require internet and is a third-party request, so it's manual, not automatic.

---

## No API Keys, No Accounts

Nothing in this app requires signup, an API key, or a paid service. Every external call is either to a free/keyless public endpoint (HIBP) or a public CDN script (Argon2id, zxcvbn), and both degrade gracefully if unreachable.
