# Project Design Document: go/usefultools

## 1. Project Overview
**Name:** go/usefultools
**Purpose:** A centralized, highly secure internal Google Site hosting a suite of utility web apps (converters, formatters, and diff checkers) for Googlers (SWEs, PMs, Analysts).
**Core Value Proposition:** 100% client-side processing. Zero server-side dependencies. Confidential internal data never leaves the browser memory, ensuring strict compliance with internal data privacy policies.

---

## 2. Architecture & Technical Constraints

All tools MUST adhere to the following architectural rules:
*   **Single-File Architecture:** Each tool must be entirely contained within a single `index.html` file (HTML, CSS, and JS combined).
*   **Hosting:** Code is directly embedded into Google Sites using the "Embed Code" `iframe` sandbox.
*   **No Build Step:** Use Vanilla JavaScript (ES6+) only. No React, Vue, or build-dependent frameworks.
*   **External Dependencies:** Must use trusted CDNs (e.g., cdnjs) loaded via `<script>` tags in the `<head>`. Code cannot exceed Google Sites' embed character limits, so large libraries must be imported, not inline.
*   **Main Thread Protection:** Any computationally heavy task (parsing large CSVs, diffing large texts, image manipulation) MUST be offloaded to a Web Worker to prevent UI freezing.
    *   *Worker Implementation:* Because we are limited to a single file, Web Workers must be implemented using **Inline Blob Workers** (creating a Worker from a string).
*   **Clipboard API:** Always provide a fallback `document.execCommand('copy')` as modern `navigator.clipboard` APIs frequently fail inside cross-origin `iframe` sandboxes.

---

## 3. Design System & UI/UX Guidelines

*   **Typography:** Google Sans (wght: 500) for Headers and Buttons. Roboto (wght: 400, 500) for body text. Roboto Mono for code/data inputs.
*   **Icons:** Google Material Symbols Outlined.
*   **Animations:** Tactile, spring-like CSS transitions on buttons (`cubic-bezier(0.4, 0, 0.2, 1)`). "Clear" actions should trigger a brief background flash animation on input fields to provide visual feedback.
*   **Layout:** Responsive CSS Grid (`1fr 1fr` on desktop, stacking to `1fr` on mobile). 

### Base CSS Template (Root Variables)
```css
:root {
  --bg-color: #f8f9fa;
  --border-color: #dadce0;
  --primary-blue: #1a73e8;
  --primary-hover: #174ea6;
  --success-bg: #e6f4ea;
  --success-text: #137333;
  --danger-bg: #fce8e6;
  --danger-text: #c5221f;
  --text-main: #202124;
  --text-secondary: #5f6368;
}
```

---

## 4. Current Tool Arsenal (Completed)

1.  **CSV to JSON Converter**
    *   *Core Engine:* Papa Parse (via CDN).
    *   *Features:* Web Worker enabled, handles escaped quotes, strips empty lines.
2.  **JSON to CSV Converter**
    *   *Core Engine:* Papa Parse (via CDN).
    *   *Features:* Custom recursive object flattening function (handles nested JSON arrays and objects), strict syntax validation.
3.  **Text Diff Checker**
    *   *Core Engine:* jsdiff (via CDN).
    *   *Features:* Inline Blob Worker for heavy diffing, word-level granular highlighting, text swap functionality.

---

## 5. Agent Instructions for Generating New Tools

When asked to generate a *new* tool for this project, the AI Agent MUST:

1.  **Use the Base Template:** Start with the standard HTML boilerplate, embedding the Google Fonts, Material Symbols, and standard `:root` CSS variables.
2.  **Implement the Standard Toolbar:** All tools must have a uniform bottom toolbar layout featuring:
    *   Primary Action Button (Blue).
    *   Secondary Action Buttons (Copy, Download).
    *   A "Clear" button that resets inputs and triggers the `.flash-clear` CSS animation.
3.  **Handle State Changes Visually:** Action buttons must swap icons (e.g., to a checkmark) and turn green (`--success-bg`) for 2 seconds upon successful completion (Copy/Download).
4.  **Enforce Asynchronous Processing:** If the tool parses data (JSON, CSV, Base64, Image Data), immediately disable the primary button, inject a CSS `.spinner`, and defer the processing using `requestAnimationFrame` or an Inline Web Worker to ensure the browser paints the loading state.
5.  **Fail Gracefully:** Catch all errors (e.g., malformed JSON, invalid image formats) and display them in red (`--danger-text`) inside the output box. Do not rely on `console.log` for error reporting to the user.
6.  **Provide Google Sites Page Copy:** Along with the code, the agent MUST ALWAYS output the text copy to be pasted into Google Sites below the embed. This copy feeds internal SEO and must strictly follow this 4-part structure formatted as markdown blockquotes:
    *   **Section 1: The Quick Pitch:** (Google Sites `Text Box`). A punchy headline (e.g., "Transform spreadsheet data into developer-ready code instantly") followed by a 2-sentence description of the tool.
    *   **Section 2: Key Features:** (Google Sites `3 Image/Text columns`). 3 bullet points starting with an emoji on each headline (the 2nd bullet headline MUST ALWAYS be `🔒 **100% Secure:**` highlighting client-side local data processing).
    *   **Section 3: Step-by-Step Guide:** (Google Sites `Collapsible Group`). A numbered list of instructions on how to use the tool.
    *   **Section 4: The Security Guarantee:** (Google Sites `Text Box`). A privacy notice explicitly stating that data processing is 100% client-side and secure for internal data.
