# HTML Study Guide & Dashboard Generator

## Template category

**Category:** Interactive Educational & Reference Document

* This template generates a fully self-contained, downloadable HTML file designed to serve as a comprehensive study guide, operational roadmap, or information dashboard for any complex topic.

---

## prompt

`Act as an expert front-end developer and educational content designer. Generate a single, fully self-contained HTML file that acts as a comprehensive, highly structured study guide and resource dashboard for [INSERT TOPIC/EXAM HERE]. The file must be fully responsive, beautifully styled with embedded CSS, use clean visual hierarchy, and include absolutely all relevant information, step-by-step roadmaps, core definitions, and verifiable external resource links within the document. Do not use placeholders, do not summarize sections, and ensure absolutely zero markdown or text leaks outside the HTML code block. Every single HTML tag must be opened and closed with flawless syntactic precision to ensure the file renders perfectly without truncating.`

**Prompt details:**

* **Dynamic Variable:** Replace `[INSERT TOPIC/EXAM HERE]` with your specific subject (e.g., "B1 Polish Exam", "AWS Certified Solutions Architect", "JavaScript Data Structures").
* **Design Ethos:** Modern, minimalist dashboard style utilizing clean borders, subtle background tints, and bold typographical weights for rapid readability.

**Instructions:**

* **Incorporate All Context:** Pack the HTML with every piece of analysis, roadmap data, tips, and lists previously discussed or requested.
* **Embedded Styling:** Place a clean, modern CSS reset and style suite directly inside a `<style>` block in the HTML `<head>`.
* **Structural Semantics:** Organize data logically using semantic HTML5 tags (`<header>`, `<main>`, `<section>`, `<article>`, `<table>`).
* **Functional Interactivity:** (Optional) Add a simple, vanilla JavaScript search filter or checkbox system inside a `<script>` block if the dashboard requires interactive task tracking.
* **Actionable Links:** Hardcode valid, absolute URLs for all reference sites, registration portals, and downloadable material sources using target="_blank".

**Constraints:**

* **Strict Self-Containment:** No external CSS files, no external JS frameworks, and no external images (use clean SVG code or CSS shapes if icons are needed).
* **Zero Truncation:** Do not use ellipses `...`, shorthand code blocks, or comments like `<!-- rest of code goes here -->`. Write out every single line completely.
* **Flawless Tag Nesting:** Ensure strict execution rules for code rendering—every parent container tag must explicitly close to prevent browser rendering drops or cut-offs.
* **Strict Output Isolation:** Deliver the final asset exclusively inside a single markdown code block marked as HTML. No intro or outro text outside the block.

**Required Output:**

* A single, error-free `<!DOCTYPE html>` structure containing `<head>`, `<style>`, `<body>`, script elements, layout grids, tables, and typography nodes, immediately ready to be saved locally with an `.html` extension.

---

**Template Name:** The Zero-Truncation Self-Contained Dashboard Generator

---

**Is it worth it?**

Yes! This template:

* Prevents the AI from taking lazy shortcuts or cutting off long HTML strings halfway through generation.
* Guarantees the resulting file works completely offline as a standalone tool on your computer or phone.
* Enforces semantic web standards so the text is instantly scannable, cleanly spaced, and easy on the eyes during long study sessions.

If you are ready to build another custom resource page, let me know the exact topic or exam details you want to target next, and I can generate the raw code using this framework!
