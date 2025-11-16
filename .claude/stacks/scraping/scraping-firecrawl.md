# Firecrawl Web Scraping Patterns

## Overview

Firecrawl is a web scraping API that handles JavaScript rendering, anti-bot protection, and returns clean, structured data ready for LLM consumption. It's designed specifically for AI applications that need to extract content from websites.

**Key Features:**
- JavaScript rendering with Playwright
- Bypass anti-bot protections
- Returns clean markdown or structured data
- Automatic sitemap crawling
- Rate limiting and retry logic built-in
- Screenshot capture
- Batch processing

**Official Website:** https://firecrawl.dev

---

## Installation and Setup

**📚 Official Documentation**: [Firecrawl - Documentation](https://docs.firecrawl.dev)

Follow the official documentation for the latest installation instructions and API setup guides.

### Quick Reference

The general setup process involves:
1. Sign up at [firecrawl.dev](https://firecrawl.dev)
2. Get your API key from the dashboard
3. Install the Firecrawl SDK for your language
4. Configure environment variables
5. Initialize the Firecrawl client

**Note**: SDK installation commands and API methods may change. Always refer to the official docs above for the current setup method.

### Environment Variables

```bash
# .env.local
FIRECRAWL_API_KEY=fc-xxxxxxxxxxxxx
```

---

## Basic Usage

### Initialize Client

**lib/firecrawl.ts**
```typescript
import FirecrawlApp from '@mendable/firecrawl-js';

export const firecrawl = new FirecrawlApp({
  apiKey: process.env.FIRECRAWL_API_KEY!
});
```

### Scrape Single Page

```typescript
import { firecrawl } from '@/lib/firecrawl';

export async function scrapePage(url: string) {
  const result = await firecrawl.scrapeUrl(url);

  return {
    markdown: result.markdown, // Clean markdown content
    html: result.html, // Original HTML
    metadata: result.metadata // Title, description, etc.
  };
}

// Usage
const data = await scrapePage('https://example.com/article');
console.log(data.markdown);
```

---

## Scraping Modes

### 1. Markdown Mode (Default)

Best for articles, blog posts, documentation.

```typescript
export async function scrapeAsMarkdown(url: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown']
  });

  return result.markdown;
}
```

**Example Output:**
```markdown
# Article Title

This is the main content of the article...

## Section 1

Content here...
```

### 2. Structured Data Extraction

Extract specific fields using a schema.

```typescript
import { z } from 'zod';

const ArticleSchema = z.object({
  title: z.string(),
  author: z.string(),
  publishDate: z.string(),
  content: z.string(),
  tags: z.array(z.string())
});

export async function scrapeArticle(url: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['extract'],
    extract: {
      schema: ArticleSchema.shape,
      systemPrompt: 'Extract article information from the page'
    }
  });

  return ArticleSchema.parse(result.extract);
}
```

### 3. HTML Mode

Get raw HTML (with JavaScript rendered).

```typescript
export async function scrapeHTML(url: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['html']
  });

  return result.html;
}
```

### 4. Screenshot Mode

Capture visual representation of the page.

```typescript
export async function captureScreenshot(url: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['screenshot']
  });

  // Returns base64 encoded image
  return result.screenshot;
}

// Save to file
import fs from 'fs';

const screenshot = await captureScreenshot('https://example.com');
const buffer = Buffer.from(screenshot, 'base64');
fs.writeFileSync('screenshot.png', buffer);
```

---

## Crawling Multiple Pages

### Crawl Entire Website

```typescript
export async function crawlWebsite(url: string) {
  const crawlResult = await firecrawl.crawlUrl(url, {
    limit: 100, // Max pages to crawl
    scrapeOptions: {
      formats: ['markdown']
    }
  });

  // Returns array of scraped pages
  return crawlResult.data.map(page => ({
    url: page.url,
    markdown: page.markdown,
    metadata: page.metadata
  }));
}

// Usage
const pages = await crawlWebsite('https://example.com');
console.log(`Crawled ${pages.length} pages`);
```

### Crawl with Filters

```typescript
export async function crawlBlogPosts(baseUrl: string) {
  const crawlResult = await firecrawl.crawlUrl(baseUrl, {
    limit: 50,
    // Only crawl URLs matching this pattern
    includePaths: ['/blog/*', '/posts/*'],
    // Exclude certain paths
    excludePaths: ['/blog/archive/*', '*.pdf'],
    scrapeOptions: {
      formats: ['markdown', 'extract'],
      extract: {
        schema: {
          title: 'string',
          author: 'string',
          date: 'string',
          content: 'string'
        }
      }
    }
  });

  return crawlResult.data;
}
```

### Crawl with Depth Limit

```typescript
export async function crawlWithDepth(url: string, maxDepth: number = 2) {
  const crawlResult = await firecrawl.crawlUrl(url, {
    maxDepth, // How many levels deep to crawl
    limit: 100,
    scrapeOptions: {
      formats: ['markdown']
    }
  });

  return crawlResult.data;
}
```

---

## Advanced Options

### Wait for Selectors

Useful for pages with dynamic content.

```typescript
export async function scrapeWithWait(url: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown'],
    waitFor: 5000, // Wait 5 seconds for content to load
    // Or wait for specific selector
    // waitFor: '.article-content'
  });

  return result.markdown;
}
```

### Custom Headers and Cookies

```typescript
export async function scrapeWithAuth(url: string, token: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown'],
    headers: {
      'Authorization': `Bearer ${token}`,
      'User-Agent': 'MyApp/1.0'
    }
  });

  return result.markdown;
}
```

### Remove Unwanted Elements

```typescript
export async function scrapeClean(url: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown'],
    removeSelectors: [
      'nav',
      'footer',
      '.advertisement',
      '#comments',
      '.sidebar'
    ]
  });

  return result.markdown;
}
```

### Extract Links

```typescript
export async function extractLinks(url: string) {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['links']
  });

  return result.links; // Array of all links found on the page
}
```

---

## Batch Processing

### Scrape Multiple URLs

```typescript
export async function batchScrape(urls: string[]) {
  const results = await Promise.allSettled(
    urls.map(url =>
      firecrawl.scrapeUrl(url, {
        formats: ['markdown']
      })
    )
  );

  return results.map((result, index) => {
    if (result.status === 'fulfilled') {
      return {
        url: urls[index],
        success: true,
        markdown: result.value.markdown
      };
    } else {
      return {
        url: urls[index],
        success: false,
        error: result.reason.message
      };
    }
  });
}
```

### Rate-Limited Batch Scraping

```typescript
import pLimit from 'p-limit';

export async function batchScrapeRateLimited(
  urls: string[],
  concurrency: number = 3
) {
  const limit = pLimit(concurrency);

  const results = await Promise.all(
    urls.map(url =>
      limit(async () => {
        try {
          const result = await firecrawl.scrapeUrl(url, {
            formats: ['markdown']
          });

          return {
            url,
            success: true,
            markdown: result.markdown
          };
        } catch (error: any) {
          return {
            url,
            success: false,
            error: error.message
          };
        }
      })
    )
  );

  return results;
}
```

---

## Common Use Cases

### 1. Scrape Documentation

```typescript
interface DocPage {
  title: string;
  content: string;
  url: string;
  section: string;
}

export async function scrapeDocumentation(docsUrl: string): Promise<DocPage[]> {
  const crawlResult = await firecrawl.crawlUrl(docsUrl, {
    limit: 200,
    includePaths: ['/docs/*'],
    scrapeOptions: {
      formats: ['extract'],
      extract: {
        schema: {
          title: 'string',
          content: 'string',
          section: 'string'
        }
      }
    }
  });

  return crawlResult.data.map(page => ({
    title: page.extract.title,
    content: page.extract.content,
    url: page.url,
    section: page.extract.section
  }));
}
```

### 2. Monitor Competitor Prices

```typescript
interface ProductPrice {
  name: string;
  price: number;
  currency: string;
  inStock: boolean;
  url: string;
}

export async function scrapeCompetitorPrices(
  productUrls: string[]
): Promise<ProductPrice[]> {
  const results = await batchScrapeRateLimited(productUrls, 5);

  return results
    .filter(r => r.success)
    .map(r => {
      const result = r as any;

      return {
        name: result.metadata.title,
        price: extractPrice(result.markdown),
        currency: 'USD',
        inStock: !result.markdown.includes('Out of Stock'),
        url: r.url
      };
    });
}

function extractPrice(markdown: string): number {
  const priceMatch = markdown.match(/\$(\d+\.?\d*)/);
  return priceMatch ? parseFloat(priceMatch[1]) : 0;
}
```

### 3. Content Aggregation

```typescript
interface Article {
  title: string;
  summary: string;
  author: string;
  publishDate: string;
  url: string;
  content: string;
}

export async function aggregateNews(sources: string[]): Promise<Article[]> {
  const allArticles: Article[] = [];

  for (const source of sources) {
    const crawlResult = await firecrawl.crawlUrl(source, {
      limit: 20,
      includePaths: ['/article/*', '/news/*'],
      scrapeOptions: {
        formats: ['extract'],
        extract: {
          schema: {
            title: 'string',
            author: 'string',
            publishDate: 'string',
            summary: 'string',
            content: 'string'
          }
        }
      }
    });

    const articles = crawlResult.data.map(page => ({
      ...page.extract,
      url: page.url
    }));

    allArticles.push(...articles);
  }

  return allArticles;
}
```

### 4. SEO Analysis

```typescript
interface SEOData {
  url: string;
  title: string;
  description: string;
  h1Tags: string[];
  imageCount: number;
  wordCount: number;
  internalLinks: number;
  externalLinks: number;
}

export async function analyzeSEO(url: string): Promise<SEOData> {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown', 'html']
  });

  const markdown = result.markdown || '';
  const html = result.html || '';

  const h1Matches = html.match(/<h1[^>]*>(.*?)<\/h1>/gi) || [];
  const h1Tags = h1Matches.map(tag => tag.replace(/<\/?h1[^>]*>/gi, ''));

  const imageCount = (html.match(/<img/gi) || []).length;
  const wordCount = markdown.split(/\s+/).length;

  const internalLinks = (html.match(new RegExp(`href="(${url}[^"]*)"`, 'gi')) || []).length;
  const externalLinks = (html.match(/href="http/gi) || []).length - internalLinks;

  return {
    url,
    title: result.metadata?.title || '',
    description: result.metadata?.description || '',
    h1Tags,
    imageCount,
    wordCount,
    internalLinks,
    externalLinks
  };
}
```

### 5. Lead Generation

```typescript
interface LeadInfo {
  companyName: string;
  email?: string;
  phone?: string;
  address?: string;
  description: string;
}

export async function extractLeadInfo(url: string): Promise<LeadInfo | null> {
  try {
    const result = await firecrawl.scrapeUrl(url, {
      formats: ['extract'],
      extract: {
        schema: {
          companyName: 'string',
          email: 'string?',
          phone: 'string?',
          address: 'string?',
          description: 'string'
        },
        systemPrompt: 'Extract company contact information and description from the page'
      }
    });

    return result.extract as LeadInfo;
  } catch (error) {
    console.error(`Failed to extract lead info from ${url}:`, error);
    return null;
  }
}
```

---

## Integration with AI

### RAG (Retrieval-Augmented Generation)

```typescript
import { openrouter } from '@/lib/openrouter';

export async function answerFromWebsite(
  websiteUrl: string,
  question: string
): Promise<string> {
  // 1. Scrape website content
  const crawlResult = await firecrawl.crawlUrl(websiteUrl, {
    limit: 50,
    scrapeOptions: {
      formats: ['markdown']
    }
  });

  // 2. Combine all page content
  const context = crawlResult.data
    .map(page => `URL: ${page.url}\n\n${page.markdown}`)
    .join('\n\n---\n\n');

  // 3. Send to LLM with context
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      {
        role: 'system',
        content: 'Answer questions based on the provided website content. Include source URLs when referencing specific information.'
      },
      {
        role: 'user',
        content: `Website Content:\n${context}\n\nQuestion: ${question}`
      }
    ]
  });

  return completion.choices[0].message.content || '';
}
```

### Content Summarization

```typescript
export async function summarizeWebpage(url: string): Promise<string> {
  // Scrape the page
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown']
  });

  // Summarize with AI
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3-haiku',
    messages: [
      {
        role: 'user',
        content: `Summarize this article in 3-5 bullet points:\n\n${result.markdown}`
      }
    ]
  });

  return completion.choices[0].message.content || '';
}
```

### Competitive Analysis

```typescript
export async function compareCompetitors(urls: string[]): Promise<string> {
  // Scrape all competitor websites
  const results = await batchScrapeRateLimited(urls, 3);

  const competitorData = results
    .filter(r => r.success)
    .map(r => `Company: ${new URL(r.url).hostname}\n\n${r.markdown}`)
    .join('\n\n---\n\n');

  // Analyze with AI
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      {
        role: 'user',
        content: `Analyze these competitor websites and provide:
1. Key differences in offerings
2. Pricing comparison
3. Unique features of each
4. Market positioning

Competitor Data:
${competitorData}`
      }
    ]
  });

  return completion.choices[0].message.content || '';
}
```

---

## Error Handling

### Retry Logic

```typescript
export async function scrapeWithRetry(
  url: string,
  maxRetries: number = 3
): Promise<string | null> {
  let lastError: Error | null = null;

  for (let i = 0; i < maxRetries; i++) {
    try {
      const result = await firecrawl.scrapeUrl(url, {
        formats: ['markdown']
      });

      return result.markdown || '';
    } catch (error: any) {
      lastError = error;
      console.error(`Attempt ${i + 1} failed:`, error.message);

      // Wait before retrying (exponential backoff)
      if (i < maxRetries - 1) {
        await new Promise(resolve =>
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      }
    }
  }

  console.error(`Failed after ${maxRetries} attempts:`, lastError);
  return null;
}
```

### Validate Results

```typescript
export async function scrapeSafe(url: string): Promise<{
  success: boolean;
  markdown?: string;
  error?: string;
}> {
  try {
    const result = await firecrawl.scrapeUrl(url, {
      formats: ['markdown']
    });

    // Validate result
    if (!result.markdown || result.markdown.length < 100) {
      return {
        success: false,
        error: 'Insufficient content extracted'
      };
    }

    return {
      success: true,
      markdown: result.markdown
    };
  } catch (error: any) {
    return {
      success: false,
      error: error.message || 'Unknown error occurred'
    };
  }
}
```

---

## Caching

### Cache Scraped Content

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!
});

export async function scrapeCached(
  url: string,
  ttl: number = 3600 // 1 hour
): Promise<string> {
  const cacheKey = `scrape:${url}`;

  // Check cache
  const cached = await redis.get<string>(cacheKey);
  if (cached) {
    console.log('Cache hit');
    return cached;
  }

  // Scrape if not cached
  console.log('Cache miss - scraping');
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown']
  });

  // Store in cache
  await redis.setex(cacheKey, ttl, result.markdown || '');

  return result.markdown || '';
}
```

---

## Monitoring and Logging

### Track Scraping Metrics

```typescript
interface ScrapeLog {
  url: string;
  success: boolean;
  duration: number;
  contentLength: number;
  error?: string;
  timestamp: Date;
}

export async function scrapeWithLogging(url: string): Promise<string | null> {
  const startTime = Date.now();

  try {
    const result = await firecrawl.scrapeUrl(url, {
      formats: ['markdown']
    });

    const duration = Date.now() - startTime;

    await db.scrapeLogs.create({
      data: {
        url,
        success: true,
        duration,
        contentLength: result.markdown?.length || 0,
        timestamp: new Date()
      }
    });

    return result.markdown || '';
  } catch (error: any) {
    const duration = Date.now() - startTime;

    await db.scrapeLogs.create({
      data: {
        url,
        success: false,
        duration,
        contentLength: 0,
        error: error.message,
        timestamp: new Date()
      }
    });

    return null;
  }
}

// Get analytics
export async function getScrapeAnalytics(since: Date) {
  const logs = await db.scrapeLogs.findMany({
    where: {
      timestamp: { gte: since }
    }
  });

  return {
    totalScrapes: logs.length,
    successRate: logs.filter(l => l.success).length / logs.length,
    avgDuration: logs.reduce((sum, l) => sum + l.duration, 0) / logs.length,
    avgContentLength: logs.reduce((sum, l) => sum + l.contentLength, 0) / logs.length
  };
}
```

---

## Best Practices

### 1. Respect robots.txt

```typescript
import { robotsParser } from 'robots-txt-parser';

async function canScrape(url: string): Promise<boolean> {
  const urlObj = new URL(url);
  const robotsUrl = `${urlObj.protocol}//${urlObj.host}/robots.txt`;

  try {
    const response = await fetch(robotsUrl);
    const robotsTxt = await response.text();

    const parser = robotsParser(robotsUrl, robotsTxt);
    return parser.isAllowed(url, 'MyBot');
  } catch {
    // If no robots.txt, assume allowed
    return true;
  }
}

export async function scrapeSafely(url: string) {
  const allowed = await canScrape(url);

  if (!allowed) {
    throw new Error('Scraping not allowed by robots.txt');
  }

  return await firecrawl.scrapeUrl(url, {
    formats: ['markdown']
  });
}
```

### 2. Rate Limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
});

export async function scrapeRateLimited(url: string, userId: string) {
  const { success } = await ratelimit.limit(userId);

  if (!success) {
    throw new Error('Rate limit exceeded');
  }

  return await firecrawl.scrapeUrl(url, {
    formats: ['markdown']
  });
}
```

### 3. Content Deduplication

```typescript
import crypto from 'crypto';

function generateContentHash(content: string): string {
  return crypto.createHash('md5').update(content).digest('hex');
}

export async function scrapeUnique(url: string): Promise<{
  content: string;
  isDuplicate: boolean;
}> {
  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown']
  });

  const contentHash = generateContentHash(result.markdown || '');

  // Check if we've seen this content before
  const existing = await db.scrapedContent.findUnique({
    where: { contentHash }
  });

  if (existing) {
    return {
      content: result.markdown || '',
      isDuplicate: true
    };
  }

  // Store hash
  await db.scrapedContent.create({
    data: {
      url,
      contentHash,
      scrapedAt: new Date()
    }
  });

  return {
    content: result.markdown || '',
    isDuplicate: false
  };
}
```

### 4. Schedule Regular Scraping

```typescript
// Using cron or background job
import { CronJob } from 'cron';

export function scheduleScrapingJob(urls: string[]) {
  // Run every day at 2 AM
  const job = new CronJob('0 2 * * *', async () => {
    console.log('Starting scheduled scraping job');

    for (const url of urls) {
      try {
        const result = await scrapeCached(url, 86400); // Cache for 24 hours
        await processScrapedContent(url, result);
      } catch (error) {
        console.error(`Failed to scrape ${url}:`, error);
      }
    }

    console.log('Scraping job completed');
  });

  job.start();
}
```

---

## Resources

- **Firecrawl Dashboard:** https://firecrawl.dev/dashboard
- **API Documentation:** https://docs.firecrawl.dev
- **GitHub:** https://github.com/mendableai/firecrawl
- **Pricing:** https://firecrawl.dev/pricing
- **Discord Community:** https://discord.gg/firecrawl
