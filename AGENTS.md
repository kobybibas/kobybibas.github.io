This repository powers my [GitHub Pages](https://pages.github.com/) site, built with Hugo, where I summarize recent machine learning papers.

The site is organized around posts. Each post corresponds to a single paper and lives inside its own folder under `content/posts/`.

---

## 📂 Repository Structure

```
content/
  posts/
    YYYYMMDD_paper_short_title/
      summary.md      # main summary file
      (optional) figs/ or assets/ for images
```

* **`YYYYMMDD_paper_short_title/`**
  Folder name encodes the date and a short identifier for the paper.
  Example: `20250825_adding_error_bars_to_evals/`

* **`summary.md`**
  Contains the summary of the paper, with front matter metadata at the top.
  This file is rendered as the main blog post.

---

## 📝 Post Format

Each `summary.md` must start with a **YAML front matter** block:

```yaml
---
title: "[Summary] Adding Error Bars to Evals: A Statistical Approach to Language Model Evaluations"
date: 2025-08-20
tags:
- Language Model Evaluation
- Error Bars
- Statistical Analysis
- Uncertainty Quantification
draft: false
---
```

### Front Matter Fields

* **`title`** – Human-readable title. Start with `[Summary]` for consistency.
* **`date`** – Publication date of the post (not necessarily paper date).
* **`tags`** – List of topics, methods, or keywords.
* **`draft`** – Set to `false` to publish, `true` to keep hidden.

---

## ✍️ Writing Guidelines

* **Clarity first**: Summaries should be understandable by ML researchers who may not be experts in the exact subfield.
* **Accuracy**: Ensure all statements are faithful to the original paper. If in doubt, re-check the source.
* **Structure** (recommended):

  1. **Problem** – What issue the paper addresses.
  2. **Approach** – Core methods or contributions.
  3. **Results** – Key findings.
  4. **Takeaways** – Why it matters.

---

## 🚀 Adding a New Paper

1. Create a new folder under `content/posts/` with format:

   ```
   YYYYMMDD_paper_short_title
   ```
2. Inside it, create `summary.md` with the correct front matter.
3. Commit and push — Hugo will rebuild the site automatically.

---

## 🔍 Notes for Agents / Contributors

* **Consistency**: Use `[Summary]` in titles for all posts.
* **Tagging**: Choose 3–5 relevant tags (methods, domains, or concepts).
* **Tone**: Keep it professional, concise, and reader-friendly.
* **External Links**: If linking to the paper, prefer arXiv or official publisher link.