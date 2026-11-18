---
title: "SAMPLE: Example directory page"
slug: sample-directory
description: "A sample directory page that shows the file format. It is a draft and is not published in a normal build."
last_updated: 2026-11-18
draft: true
listings:
  - name: Example Resource (with affiliate link)
    url: https://example.com
    affiliate_url: https://example.com/?ref=sample
    summary: A placeholder listing that uses affiliate_url for the outbound link.
    category: Sample
  - name: Example Resource (plain URL)
    url: https://example.org
    summary: A placeholder listing with no affiliate_url, so the link falls back to url.
    category: Sample
---

This is a **sample** page. It exists so you can see the Markdown/YAML format for a directory page.

It is marked `draft: true`, so a normal `python3 build.py` will skip it. To preview drafts locally, run `python3 serve.py --drafts` or `python3 build.py --drafts`.

Replace this file (or add new files alongside it) once the niche is chosen. Do not publish this sample as a real page.
