---
layout: default
title: "Building a TTS Tool for My Friend in One Hour"
date: 2025-08-15
categories: [tech]
tags: [tts, microsoft, ai, text to speech, azure]
---

# Building a TTS Tool for My Friend in One Hour

<img src="{{ site.baseurl }}/assets/images/2025/08/20250815.png" alt="tts" class="img-center">

## The Ask

My friend asked if there was a service that could read academic papers aloud - not like NotebookLM which creates podcast-style summaries, but something that would actually read the original text. She wanted to listen to papers like audiobooks when her eyes got tired.

I didn't know of such a service, but since I'm familiar with Microsoft Azure Language Services, I offered to help: "Send me the paper and I'll make mp3 for you."

## The Reality Check

I thought this would be simple:
1. Extract text from PDF using Claude/ChatGPT/Grok  
2. Run it through Azure TTS (Text to Speech)
3. Done!

Wrong. Academic PDFs are messy. Extract text and you get dozens of co-author names, chart numbers, table data, footnotes - everything my friend didn't want to hear.

I tried asking different AIs to extract only title, abstract, and main content:
- Claude Sonnet: Failed
- Grok: Failed with errors  
- Claude Opus: Success

Turns out extracting clean content from academic PDFs is harder than expected.

## From Manual to Automated

I generated the MP3 using Azure TTS and made a tutorial video. But a week later, my friend was still hunting for paid PDF-to-speech services.

That's when I realized: I might not know the perfect PDF-to-text AI, but I can definitely build a basic TTS web app. 

So I coded one with Claude Sonnet, built it in Next.js, deployed on Vercel, and shared it.

This is why I love being able to code!!!

---

## 📌 Navigation
- [← All Posts](/posts)
- [🏠 Home](/)
<!-- - [📧 Contact](/contact) -->

{% if page.previous %}
**Previous:** [{{ page.previous.title }}]({{ page.previous.url }})
{% endif %}

{% if page.next %}
**Next:** [{{ page.next.title }}]({{ page.next.url }})
{% endif %}
