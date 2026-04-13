# liblognorm / mmnormalize Rulebase Quick Guide

This is a **comprehensive, clear, and practical guide** for anyone working with `mmnormalize` and liblognorm-based rulebases.

---

## 1. What a Rulebase Is

A **rulebase** is a set of patterns used by `mmnormalize` (rsyslog) or `lognormalizer` (CLI) to extract structured fields from log messages.

Each rule tries to match the **beginning of the message**.  
First matching rule wins.

```text
version=2
rule=:<pattern>
rule=:<pattern>
```

---

## 2. How Matching Works

* Rules are evaluated **top → bottom**
* Matching is **anchored at the start** of the message
* If a rule matches:

  * fields are extracted
  * processing stops
* If no rule matches → fallback rule is used

---

## 3. Basic Syntax

### 3.1 Literal text

Anything outside `%...%` is matched literally.

```text id="r9ec8p"
rule=:ERROR %msg:rest%
```

Matches:

```text id="us26di"
ERROR something failed
```

---

### 3.2 Fields

Fields capture parts of the message:

```text id="iczws3"
%fieldname:type%
```

Example:

```text id="ce5ouy"
rule=:%user:word% logged in
```

---

## 4. Field Types (Most Useful Ones)

### word

Matches a single word (no spaces)

```text id="dt3rde"
%user:word%
```

---

### number

Matches digits

```text id="txs521"
%id:number%
```

---

### char-to:X

Match until character `X` (NOT including it)

```text id="r0topb"
%tag:char-to:/%
```

Example:

```text id="oxrd0i"
CFGMAN/5/... → tag = CFGMAN
```

---

### rest

Match everything remaining

```text id="gfr77f"
%msg:rest%
```

---

### whitespace

Matches spaces/tabs

```text id="8sft5r"
%space:whitespace%
```

Useful to consume unwanted spacing.

---

## 5. Delimiters (IMPORTANT)

Delimiters like `/`, `:`, `(` are:

✔ written literally
❌ NOT fields

Example:

```text id="jb56pw"
rule=:%tag:char-to:/%/%severity:number%/%msg:rest%
```

---

## 6. Special Characters (CRITICAL)

### `%` must be escaped

Because `%` starts a field.

### Use hex:

| Character | Escape |
| --------- | ------ |
| `%`       | `\x25` |
| `/`       | `\x2f` |
| `:`       | `\x3a` |

---

### Example (HPE logs):

```text id="s2q868"
%%10CFGMAN/5/...
```

Becomes:

```text id="jfyf47"
\x25\x2510
```

---

## 7. Practical Example (HPE Logs)

Input:

```text id="xyjcm9"
%%10CFGMAN/5/CFGMAN_CFGCHANGED(l): message...
```

Rule:

```text id="uosv85"
rule=: \x25\x2510%tag:char-to:/%/%severity:number%\x2f%event:char-to::%:%msg:rest%
```

Result:

| Field    | Value                |
| -------- | -------------------- |
| tag      | CFGMAN               |
| severity | 5                    |
| event    | CFGMAN_CFGCHANGED(l) |
| msg      | message...           |

---

## 8. Common Pitfalls (VERY IMPORTANT)

### ❌ 1. Forgetting `%` escaping

```text id="v4duyy"
rule=: %%10...   ← often fails
```

✔ use:

```text id="jnd37f"
\x25\x2510
```

---

### ❌ 2. Trying to “parse” delimiters

Wrong:

```text id="vdcfnd"
%"/"%   ❌ invalid
```

Correct:

```text id="80gyf1"
/.../   ✔ literal
```

---

### ❌ 3. Not consuming all segments

If input is:

```text id="5vj3zs"
A/B/C
```

You must parse:

```text id="yetqsb"
%a:char-to:/%/%b:char-to:/%/%c:rest%
```

---

### ❌ 4. Leading whitespace

If logs start with space:

```text id="gz4fkb"
" %%10..."
```

You must handle it:

```text id="bd8rm0"
%space:whitespace%\x25\x2510...
```

---

### ❌ 5. Rule never matches

Symptoms:

* variables remain unset
* fallback rule used

Cause:

* mismatch at start of message

---

## 9. Debugging Tips (lognormalizer vs rsyslog/mmnormalize)

### Use lognormalizer (CLI)

```bash
lognormalizer -H -U -r rules.rb < test.log
```

This is the fastest way to test rule matching and field extraction.

---

### ⚠️ IMPORTANT: Differences vs rsyslog (mmnormalize)

`lognormalizer` and `mmnormalize` use the same liblognorm engine, but the **input they process is different**.

---

### 1. Input data differences

#### lognormalizer (CLI)

* Processes the **full raw log line**
* Example:

```text
<189>Apr 13 17:43:06 2026 host %%10CFGMAN/5/...
```

---

#### rsyslog + mmnormalize

* Processes **only the message part (`$msg`) by default**
* Example:

```text
 %%10CFGMAN/5/...
```

✔ syslog header is already parsed and removed

---

### 2. Practical impact

A rule that works in `lognormalizer` may fail in rsyslog because:

* the **prefix is missing**
* matching no longer starts at the same position

---

### Example

#### Rule (designed for full syslog line):

```text
rule=:<%pri:number%>%month:word% %day:number% ...
```

✔ Works in `lognormalizer`
❌ Fails in rsyslog (header already stripped)

---

#### Correct rule for rsyslog:

```text
rule=: \x25\x2510%tag:char-to:/%/%severity:number%\x2f%event:char-to::%:%msg:rest%
```

---

### 3. Leading whitespace issue

In rsyslog, `$msg` often begins with a space:

```text
" %%10CFGMAN/..."
```

So rules must handle it:

```text
%space:whitespace%\x25\x2510...
```

---

### 4. Forcing rsyslog to use raw message

You can change behavior:

```rsyslog
action(type="mmnormalize" ruleBase="rules.rb" useRawMsg="on")
```

Now mmnormalize sees:

```text
<189>Apr 13 ... %%10CFGMAN/...
```

✔ Matches lognormalizer behavior
⚠ Requires different rules

---

### 5. Key takeaway

| Tool                         | Input        |
| ---------------------------- | ------------ |
| lognormalizer                | full raw log |
| mmnormalize (default)        | `$msg` only  |
| mmnormalize (useRawMsg="on") | full raw log |

---

### 6. Debugging strategy

✔ Start with `lognormalizer` to validate parsing
✔ Then adapt rules for rsyslog input (`$msg`)
✔ Watch for:

* missing prefixes
* leading whitespace
* different match position

---

### 7. Common symptom of mismatch

* Works in CLI
* Fails in rsyslog
* Fields remain unset

👉 Almost always caused by **input mismatch**

---

## 10. Fallback Rule (ALWAYS include)

```text id="mcxr8y"
rule=:%msg:rest%
```

Prevents unparsed logs.

---

## 11. Best Practices

✔ Start simple, then refine
✔ Match structure, not content
✔ Always include fallback
✔ Escape `%` reliably
✔ Model separators (`/`, `:`) explicitly
✔ Keep rules minimal and readable

---

## 12. Minimal Template

```text id="m7wak5"
version=2

rule=: \x25\x2510%tag:char-to:/%/%severity:number%\x2f%event:char-to::%:%msg:rest%
rule=:%msg:rest%
```

---

## Final Takeaway

Think of liblognorm as:

> **“Split the line step by step using known separators.”**

Not regex. Not fuzzy matching.
Just deterministic parsing from left to right.

---

This guide focuses on what actually works in real rsyslog deployments, especially with structured logs like HPE, Cisco, and similar network devices.

**Credit:**
This guide was created by [rsyslog Assistant](https://github.com/rsyslog/rsyslog). It's designed to help users understand and use `mmnormalize` effectively.
