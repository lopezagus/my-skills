---
name: deluge
description: Write, review, or debug Zoho Deluge code. Use this skill whenever the user asks to write a Deluge function, script, workflow, or custom function for any Zoho app (CRM, Books, Desk, Creator, etc.), review or fix existing Deluge code, understand a Deluge error, or design an integration between Zoho apps. Also trigger when the user mentions invokeurl, Zoho connections, or Deluge-specific syntax. This skill enforces a strict house coding style and prevents hallucination of non-existent Deluge functions.
---

# Deluge Skill

Zoho Deluge is a proprietary scripting language used across all Zoho apps. It is **not JavaScript** — it superficially resembles it but has a completely different standard library, no prototype chain, no `let`/`const`/`var`, no arrow functions, and no `null` coercion semantics. Every hallucinated JS method applied to a Deluge value is a runtime error.

This skill enforces a specific house style and a whitelist of real built-in functions.

---

## Step 1 — Detect Context

Before writing any code, identify which Zoho application(s) are involved:

| Signal in the request | Context |
|---|---|
| `invokeurl`, `zcrm` connection, modules like Quotes, Leads, Contacts, Deals | **Zoho CRM** |
| `zbooks` connection, invoices, bills, estimates, items, vendors | **Zoho Books** |
| Both | **CRM + Books integration** |
| `zdesk` connection, tickets, agents | **Zoho Desk** |
| No app mentioned, standalone logic | **Generic Deluge** |

Once context is identified, read the matching reference file(s):
- **CRM** → `references/zoho-crm-api.md`
- **Books** → `references/zoho-books-api.md`
- **Always** → `references/deluge-builtins.md` (real functions only)
- **Always** → `references/coding-style.md` (house style)

---

## Step 2 — Apply House Style

Read `references/coding-style.md` before writing any code. Key rules at a glance:

- **camelCase** for all variable names. No exceptions.
- **Block comments only** — never inline `//` comments. Use `/* ... */` blocks with section headers.
- **URL construction pattern**: define a base URL constant, then build module-specific and record-specific URLs by concatenation.
- **Execution flags** `logMode` and `logRaw` defined at the top of every function.
- **Return maps** `errorMap` and `successMap` initialized at the top and returned at appropriate exit points.
- Every API call must be followed by an empty-result guard using `.isEmpty()` and `.isNull()`.
- Every `if` block uses braces `{}` even for single-line bodies.

---

## Step 3 — Use Only Real Deluge Functions

Read `references/deluge-builtins.md` before writing any logic. **Do not use any function not listed there.** When in doubt, check the reference — if it is not listed, it does not exist in Deluge.

Common hallucination traps:
- There is no `Array.map()`, `Array.filter()`, `Array.reduce()`. Use `for each` loops.
- There is no `JSON.parse()` or `JSON.stringify()`. Maps and lists are native types.
- There is no `String.split()` with a regex. Use `.toList(delimiter)`.
- There is no `.length` property. Use `.size()`.
- There is no `parseInt()` or `parseFloat()`. Use `.toLong()` or `.toDecimal()`.
- `invokeurl` is a **statement block**, not a function call. Syntax is strict.

---

## Step 4 — Write or Review the Code

When writing new code, always produce the full function body — never snippets without context. Structure: constants block → execution flags → data extraction → processing logic → return.

When reviewing code, check in this order:
1. Does it use any hallucinated functions? (cross-reference builtins)
2. Does it handle empty/null API responses?
3. Does it follow the naming and comment style?
4. Is the `invokeurl` block syntax correct?
5. Are URL variables constructed from the base URL constant?

---

## Reference Files

| File | When to read |
|---|---|
| `references/coding-style.md` | Always — before writing any code |
| `references/deluge-builtins.md` | Always — before writing any logic |
| `references/zoho-crm-api.md` | When the function touches Zoho CRM |
| `references/zoho-books-api.md` | When the function touches Zoho Books |
