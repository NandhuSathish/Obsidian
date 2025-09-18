---
tags:
  - nodejs
---

# 📦 Semantic Versioning (SemVer) in Node.js

Semantic versioning helps us manage package versions clearly using this format:


MAJOR.MINOR.PATCH

````

- **MAJOR** → Breaking changes
- **MINOR** → New features (backward compatible)
- **PATCH** → Bug fixes (backward compatible)

---

## ✅ Version Ranges: `^` vs `~`

### `^` (Caret)
- Allows updates to **minor and patch** versions.
- Avoids breaking changes from new major versions.

**Example:**
```json
"express": "^4.17.1"
````

✔ Accepts `4.17.2`, `4.18.0`
❌ Does not update to `5.0.0`

---

### `~` (Tilde)

-   Allows updates to **patch** versions only.
-   Locks the dependency more tightly.

**Example:**

```json
"express": "~4.17.1"
```

✔ Accepts `4.17.2`, `4.17.5`
❌ Does not update to `4.18.0` or `5.0.0`

---

## ✅ When to Use

-   Use **`^`** when you're fine with feature improvements and fixes.
-   Use **`~`** when you want to allow only small bug fixes.

---

## ✅ Quick Reference Table

| Symbol | Allowed Updates         | Example (`4.17.1`) |
| ------ | ----------------------- | ------------------ |
| `^`    | Minor and patch updates | `>=4.17.1 <5.0.0`  |
| `~`    | Patch updates only      | `>=4.17.1 <4.18.0` |


