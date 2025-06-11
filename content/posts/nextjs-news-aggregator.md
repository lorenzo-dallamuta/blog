---
title: "Building a News Aggregator with Next.js and NewsAPI – Part 1: Exploring the API"
date: 2025-06-08
draft: false
---

## Why a News Aggregator?

I've been wanting to experiment with **hybrid rendering in Next.js**—mixing server-side rendering (SSR) for SEO-critical pages and client-side rendering (CSR) for dynamic filters. A news aggregator felt like the perfect use case:

- **SSR** for fast-loading, crawlable homepage headlines.
- **CSR** for real-time search/filtering as users tweak their queries.

Instead of building a custom backend, I'm using the [NewsAPI](https://newsapi.org/)—a free API with global news data—to focus on the frontend architecture. But first, let's see what the API actually offers!

## First Impressions of NewsAPI

Before writing a single line of code, I wanted to **understand what the API does best**—so I could design a UI that highlights its strengths rather than fights its limitations. After all, there's no point in building a fancy "top headlines" dashboard if the API frequently returns empty responses!

NewsAPI provides three key endpoints:
1. **`/everything`** – Searches all articles (flexible filters, full-text search).
2. **`/top-headlines`** – Breaking news by country/category (simpler, but limited).
3. **`/sources`** – Lists available publishers (useful for filtering).

## Testing the API with cURL

I started with the API's ["Hello World" example](https://newsapi.org/docs/get-started)—fetching Apple-related news:

```bash
curl https://newsapi.org/v2/everything \
  -G -d q=Apple \
  -d from=2025-05-08 \
  -d sortBy=popularity \
  -d apiKey=YOUR_KEY