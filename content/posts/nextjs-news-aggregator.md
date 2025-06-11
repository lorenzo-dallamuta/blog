# Why a News Aggregator?

I’ve been wanting to experiment with hybrid rendering in Next.js—mixing server-side rendering (SSR) for SEO-critical pages and client-side rendering (CSR) for dynamic filters. A news aggregator felt like the perfect use case:

- **SSR** for fast-loading, crawlable homepage headlines.  
- **CSR** for real-time search/filtering as users tweak their queries.  

Instead of building a custom backend, I’m using the NewsAPI—a free API with global news data—to focus on the frontend architecture. But first, let’s see what the API actually offers!

---

## First Impressions of NewsAPI

Before writing a single line of code, I wanted to understand what the API does best—so I could design a UI that highlights its strengths rather than fights its limitations. After all, there’s no point in building a fancy "top headlines" dashboard if the API frequently returns empty responses!

NewsAPI provides three key endpoints:

1. `/everything` – Searches all articles (flexible filters, full-text search).  
2. `/top-headlines` – Breaking news by country/category (simpler, but limited).  
3. `/sources` – Lists available publishers (useful for filtering).  

---

### Testing the API with cURL

I started with the API’s "Hello World" example[link]—fetching Apple-related news:  
```bash
curl https://newsapi.org/v2/everything \
  -G -d q=Apple \
  -d from=2025-05-08 \
  -d sortBy=popularity \
  -d apiKey=YOUR_KEY
```

**Result**: 45,000 articles 😅. Clearly, pagination (`pageSize` and `page` params) will be an essential part of the project.  

Next, I tried fetching top headlines in France:  
```bash
curl https://newsapi.org/v2/top-headlines \
  -G -d country=fr \
  -d pageSize=5 \
  -d apiKey=YOUR_KEY
```

**Result**: `{"articles": []}`. Oops—no news on Sundays? This means our app needs fallback logic (e.g., showing older headlines or expanding the search region).  

---

### Key Takeaways from the API

- **`/everything` is the most powerful**  
  - Supports date ranges (`from`/`to`), sorting, and keyword searches.  
  - Perfect for a "search" feature with a lot of filtering options.  

- **`/top-headlines` is hit-or-miss**  
  - Great for a "breaking news" section, but empty responses are possible.  
  - *Solution*: Combine with `/everything` as a fallback.  

- **User experience considerations**  
  - Let users save favorite filters (e.g., "Tech news from Germany").  
  - Cache responses to avoid rate limits and improve performance.  

---

## What’s Next?

In **Part 2**, we’ll:  
1. Bootstrap a Next.js app.  
2. Fetch data from NewsAPI client-side.  
3. Build a basic headline list with pagination.  

By starting with the API, we avoid designing a UI that doesn’t match the data’s reality. Next time, we’ll turn these insights into code!  

**Question for you**: How would you handle empty API responses? Let me know in the comments!  