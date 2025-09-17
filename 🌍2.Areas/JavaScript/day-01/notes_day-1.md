![alt text](image.png)

# Loading and Executing Scripts in HTML

This note explains how different placements and attributes of the `<script>` tag affect the process of downloading HTML, downloading scripts, and executing scripts in a web page.

## Key Phases

-   **Download HTML & Build DOM** (🟠 Orange)
-   **Download Script** (🟠 Pink)
-   **Execute Script** (🟢 Green)

---

## 1. `<script>` tag in `<head>`

-   The browser starts by downloading the HTML and building the DOM.
-   When it encounters the `<script>` tag, it **pauses the HTML parsing**, downloads the script, and executes it.
-   After execution, it resumes building the DOM.

**Flow:**
🟠 Download HTML & Build DOM → 🟠 Pause → 🟠 Resume
🟠 → 🟡 Download Script → 🟢 Execute Script

yaml
Copy code

---

## 2. `<script>` tag at the end of `<body>`

-   The browser builds the DOM fully first.
-   After that, it downloads the script and executes it.

**Flow:**
🟠 Download HTML & Build DOM → 🟡 Download Script → 🟢 Execute Script

yaml
Copy code

---

## 3. `<script>` tag with `async`

-   The browser starts building the DOM.
-   Meanwhile, the script starts downloading asynchronously.
-   Once the script is downloaded, it interrupts the DOM building, executes the script, and then resumes.

**Flow:**
🟠 Download HTML & Build DOM → 🟡 Download Script (async) → 🟢 Execute Script → 🟠 Resume

yaml
Copy code

---

## 4. `<script>` tag with `defer`

-   The browser downloads the HTML and builds the DOM without interruptions.
-   The script is downloaded in parallel but **executes only after the DOM is fully built**.

**Flow:**
🟠 Download HTML & Build DOM → 🟡 Download Script → 🟠 DOM Complete → 🟢 Execute Script

pgsql
Copy code

---

## Summary Table

| Script Placement / Attribute | DOM Parsing                                 | Script Download          | Script Execution                        |
| ---------------------------- | ------------------------------------------- | ------------------------ | --------------------------------------- |
| In `<head>`                  | Paused during script download and execution | Immediately on encounter | Immediately after download              |
| End of `<body>`              | Complete before script download             | After DOM is ready       | Immediately after download              |
| With `async`                 | Pauses when script is ready                 | Parallel download        | Immediately after download, pausing DOM |
| With `defer`                 | Continues while downloading                 | Parallel download        | After DOM is fully built                |

---

## Notes

-   Use `defer` if the script depends on the DOM but doesn’t need to block page load.
-   Use `async` if the script can run anytime and is not dependent on DOM structure.
-   Placing scripts at the end of `<body>` is a simple way to avoid blocking DOM parsing.
-   Inline scripts in `<head>` block rendering and delay page load.
