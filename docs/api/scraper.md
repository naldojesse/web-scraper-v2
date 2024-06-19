### api/scraper.md

## Scraper Module

### Components
- **scraper.js**: Base scraper class that provides common functionality for all scrapers.
- **indeedScraper.js**: Implements scraping logic specific to Indeed.
- **linkedinScraper.js**: Implements scraping logic specific to LinkedIn.
- **glassdoorScraper.js**: Implements scraping logic specific to Glassdoor.

### Example Usage (scraper.js)
```javascript
class Scraper {
  constructor(config) {
    this.config = config;
  }

  async init() {
    // Initialize the scraper with necessary configurations
  }

  async scrape() {
    // Implement scraping logic
  }

  async close() {
    // Clean up resources
  }
}

module.exports = Scraper;

### Example Usage (indeedScraper.js)

```javascript
const Scraper = require('./scraper');
const puppeteer = require('puppeteer');

class IndeedScraper extends Scraper {
  constructor(config) {
    super(config);
  }

  async scrape() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.goto(this.config.url);

    // Implement Indeed-specific scraping logic
    const jobListings = await page.evaluate(() => {
      const listings = [];
      document.querySelectorAll('.jobsearch-SerpJobCard').forEach(jobCard => {
        const title = jobCard.querySelector('.title a').innerText;
        const company = jobCard.querySelector('.company').innerText;
        listings.push({ title, company });
      });
      return listings;
    });

    await browser.close();
    return jobListings;
  }
}

module.exports = IndeedScraper;


