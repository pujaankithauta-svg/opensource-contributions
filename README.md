# 🌟 Open Source Contributions

Tracking my contributions to prestigious open source projects — bugs fixed, pull requests submitted, and the impact made.

---

## 📊 Contribution Log

| # | Project | ⭐ Stars | Issue | Bug Description | Fix Summary | PR / Commit | Status |
|---|---------|---------|-------|-----------------|-------------|-------------|--------|
| 1 | [Textualize/rich](https://github.com/Textualize/rich) | 56k+ | [#3299](https://github.com/Textualize/rich/issues/3299) | `Segment._split_cells` produces wrong split points for strings mixing 2-cell wide characters (emoji, CJK) with 1-cell characters. The heuristic initial position `int((cut/cell_length)*len(text))` is inaccurate and the while-loop cannot backtrack to recover. | Replaced heuristic + non-backtracking while-loop with a correct O(n) linear scan over characters, accumulating cell widths until the cut point is reached. Wide chars cut in the middle are padded with spaces on both sides. | [PR #4146](https://github.com/Textualize/rich/pull/4146) | 🟡 Open |
| 2 | [psf/requests](https://github.com/psf/requests) | 54k+ | [#6102](https://github.com/psf/requests/issues/6102) | `HTTPDigestAuth` fails with non-latin credentials: when `username`/`password` is passed as `bytes`, the Authorization header contains the Python `repr` of the bytes object (e.g. `b'Ond\xc5\x99ej'`) instead of the decoded string. | Added normalization at the start of `build_digest_header()` to decode any `bytes` username/password to `str` via UTF-8 before use in both the A1 hash computation and the `username=` header field. | [Fork + Branch](https://github.com/pujaankithauta-svg/requests/tree/fix/digest-auth-non-latin-credentials) · [Commit](https://github.com/pujaankithauta-svg/requests/commit/52bae90316113988b2e3d563d7f8ccad31a242e4) | 🟠 PR Blocked (repo has contributor-only interaction limits) |

---

## 📁 Contribution Details

### 1. `Textualize/rich` — Segment._split_cells wide-char bug

**Project:** [`rich`](https://github.com/Textualize/rich) — Terminal text formatting library. 56k+ ⭐  
**File changed:** `rich/segment.py`  
**Issue:** [#3299](https://github.com/Textualize/rich/issues/3299)  
**PR:** [#4146](https://github.com/Textualize/rich/pull/4146)  

**Bug:**
```python
s = Segment("🦊🦊🦊\n\n\n\n\n\n")
Segment._split_cells(s, 3)
# ❌ Wrong: returned all 3 foxes (6 cells) on the left side
```

**Root cause:** The initial character position was estimated as:
```python
pos = int((cut / cell_length) * len(text))
```
This ratio is wrong when the string mixes 2-cell wide chars with 1-cell chars. The while-loop adjusted by `±1` but couldn't backtrack far enough.

**Fix:** Linear O(n) scan — walk each character, accumulate cell widths, stop at `cut`:
```python
cell_pos = 0
for pos, char in enumerate(text):
    char_width = cell_size(char)
    if cell_pos + char_width > cut:
        # cut falls in a wide char — pad both sides with space
        return (_Segment(text[:pos] + " ", ...), _Segment(" " + text[pos+1:], ...))
    cell_pos += char_width
    if cell_pos == cut:
        return (_Segment(text[:pos+1], ...), _Segment(text[pos+1:], ...))
```

**Outcome:** Correct splits for all mixed-width strings. `lru_cache` preserved.

---

### 2. `psf/requests` — HTTPDigestAuth non-latin credentials bug

**Project:** [`requests`](https://github.com/psf/requests) — HTTP library for Python. 54k+ ⭐  
**File changed:** `src/requests/auth.py`  
**Issue:** [#6102](https://github.com/psf/requests/issues/6102)  
**Fork:** [pujaankithauta-svg/requests](https://github.com/pujaankithauta-svg/requests/tree/fix/digest-auth-non-latin-credentials)  

**Bug:**
```python
auth = HTTPDigestAuth("Ondřej".encode("utf-8"), "heslíčko")
# ❌ Header: Digest username="b'Ond\xc5\x99ej'", ...
```

**Root cause:** `self.username` (bytes) was interpolated directly into f-strings in both the A1 hash computation and the Authorization header value.

**Fix:** Decode bytes to str at the top of `build_digest_header()`:
```python
_username = self.username.decode("utf-8") if isinstance(self.username, bytes) else self.username
_password = self.password.decode("utf-8") if isinstance(self.password, bytes) else self.password
A1 = f"{_username}:{realm}:{_password}"
# and use _username in the header too
```

**Outcome:** Correct UTF-8 username in both the hash and the Authorization header. PR was blocked by psf/requests interaction restrictions (contributor-only), fix ready to submit.

---

## 🛠️ Languages & Skills Demonstrated

- Python — bug analysis, string encoding, Unicode/wide-char handling
- Open source workflow — fork → branch → fix → PR
- Reading and understanding large production codebases

---

*Last updated: May 2026*
