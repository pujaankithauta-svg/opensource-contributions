# 🌟 Open Source Contributions

Tracking my contributions to prestigious open source projects — bugs fixed, pull requests submitted, and the impact made.

---

## 📊 Contribution Log

| # | Project | Company | ⭐ Stars | Issue | Bug Description | Fix Summary | PR | Status |
|---|---------|---------|---------|-------|-----------------|-------------|-----|--------|
| 1 | [Textualize/rich](https://github.com/Textualize/rich) | Textualize (USA) | 56k+ | [#3299](https://github.com/Textualize/rich/issues/3299) | `Segment._split_cells` wrong split points for strings mixing 2-cell emoji/CJK chars with 1-cell chars | Replaced heuristic + non-backtracking while-loop with correct O(n) linear scan; wide chars cut in the middle padded with spaces | [PR #4146](https://github.com/Textualize/rich/pull/4146) | 🟡 Open |
| 2 | [openai/openai-python](https://github.com/openai/openai-python) | OpenAI (USA) | 30k+ | [#3201](https://github.com/openai/openai-python/issues/3201) | Streaming `tool_call` deltas with duplicate logical indexes in the first chunk are accumulated incorrectly, producing invalid JSON arguments | Extracted `_accumulate_list()` helper that merges by logical `index` field (not physical position); called on both first-encounter and subsequent merges | [PR #3307](https://github.com/openai/openai-python/pull/3307) | 🟡 Open |
| 3 | [psf/requests](https://github.com/psf/requests) | Python (USA) | 54k+ | [#6102](https://github.com/psf/requests/issues/6102) | `HTTPDigestAuth` fails with non-latin credentials: bytes username renders as `b'...'` repr in Authorization header | Added UTF-8 bytes→str normalization at top of `build_digest_header()` for both hash input and header field | [Fork/Branch](https://github.com/pujaankithauta-svg/requests/tree/fix/digest-auth-non-latin-credentials) | 🟠 Blocked (repo restricts new contributors) |

---

## 📁 Contribution Details

### 1. `openai/openai-python` — Streaming tool_call delta accumulation bug

**Project:** [`openai-python`](https://github.com/openai/openai-python) — Official Python SDK for the OpenAI API. **30k+ ⭐ · OpenAI**  
**File changed:** `src/openai/lib/streaming/_deltas.py`  
**Issue:** [#3201](https://github.com/openai/openai-python/issues/3201)  
**PR:** [#3307](https://github.com/openai/openai-python/pull/3307) ✅ Open  

**Bug:**
When a streaming response's first chunk contains two entries with the same logical `"index"` value (e.g. one for the tool-call header and one for the first argument fragment), both were stored as separate physical list slots. Subsequent chunks merged into physical slot 0 while the duplicate slot retained stale partial data — producing broken argument JSON like:
```json
[
  {"index": 0, "function": {"arguments": "path\": \".\"} "}},
  {"index": 0, "function": {"arguments": " {\""}}
]
```

**Root cause:** The fast path `if key not in acc: acc[key] = delta_value` stored the raw list without merging duplicate logical indexes.

**Fix:**
```python
# New helper merges by logical index, not physical position
def _accumulate_list(acc_list, delta_list, ...):
    for delta_entry in delta_list:
        index = delta_entry["index"]
        for i, existing in enumerate(acc_list):
            if is_dict(existing) and existing.get("index") == index:
                acc_list[i] = accumulate_delta(existing, delta_entry)
                found = True; break
        if not found:
            acc_list.append(delta_entry)
    return acc_list

# Called on BOTH first-encounter and subsequent merges
if key not in acc:
    if is_list(delta_value) and all(is_dict(x) and "index" in x for x in delta_value):
        acc[key] = _accumulate_list([], delta_value, ...)
```

**Outcome:** Duplicate logical indexes in any chunk collapse correctly into a single entry. Arguments accumulate into valid JSON.

---

### 2. `Textualize/rich` — Segment._split_cells wide-char bug

**Project:** [`rich`](https://github.com/Textualize/rich) — Terminal text formatting library. **56k+ ⭐ · Textualize**  
**File changed:** `rich/segment.py`  
**Issue:** [#3299](https://github.com/Textualize/rich/issues/3299)  
**PR:** [#4146](https://github.com/Textualize/rich/pull/4146) ✅ Open  

**Bug:**
```python
Segment._split_cells(Segment("🦊🦊🦊\n\n\n\n\n\n"), 3)
# ❌ Wrong: returned all 3 foxes (6 cells) on the left side
```

**Root cause:** Initial position estimated as `pos = int((cut / cell_length) * len(text))` — wrong ratio for mixed-width strings. While-loop adjusted by ±1 but couldn't backtrack far enough.

**Fix:** O(n) linear scan accumulating cell widths character by character, stopping exactly at `cut`. Wide chars split in the middle padded with spaces on both sides.

**Outcome:** Correct splits for all mixed-width strings. `lru_cache` preserved.

---

### 3. `psf/requests` — HTTPDigestAuth non-latin credentials bug

**Project:** [`requests`](https://github.com/psf/requests) — HTTP library for Python. **54k+ ⭐**  
**File changed:** `src/requests/auth.py`  
**Issue:** [#6102](https://github.com/psf/requests/issues/6102)  
**Status:** Fix committed to fork; PR blocked by repo's contributor-only interaction limits.

**Bug:** `bytes` username/password rendered as Python `repr` (`b'...'`) in Authorization header.  
**Fix:** Decode bytes→str via UTF-8 at the start of `build_digest_header()`.

---

## 🛠️ Languages & Skills Demonstrated

- **Python** — Unicode/wide-char handling, streaming delta accumulation, HTTP auth
- **AI/ML tooling** — OpenAI streaming API internals, tool-call accumulation
- **Open source workflow** — fork → branch → fix → PR with detailed write-ups
- **Code reading** — Understanding large production codebases quickly

---

*Last updated: May 2026 · 3 contributions · 2 open PRs*
