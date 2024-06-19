Job-Automator-V2 Blueprint

Overview
Job-Automator-V2 is a highly configurable and extensible web scraping application designed to handle multiple websites and adapt to frequent updates. This Blueprint will guide you through building the application, focusing on modular components, robust error handling, session management, and anti-bot mechanisms.

Project Structure


job-automator-v2/
│
├── src/
│   ├── main/
│   │   ├── app.js
│   │   ├── config.js
│   │   ├── scraperFactory.js
│   │   ├── antiBotFactory.js
│   │   ├── dataHandler.js
│   │   ├── logger.js
│   │   ├── rateLimiter.js
│   │   ├── monitor.js
│   │   ├── auth/
│   │   │   ├── loginHandler.js
│   │   │   ├── cookieManager.js
│   │   │   ├── sessionManager.js
│   │   ├── selectors/
│   │   │   ├── heuristicFinder.js
│   │   │   ├── seleniumFinder.js
│   │   │   ├── manualInput.js
│   │   ├── formHandlers/
│   │   │   ├── jobApplicationHandler.js
│   │   │   ├── genericFormHandler.js
│   │   │   ├── notionIntegration.js
│   │   ├── navigation/
│   │   │   ├── navigationHandler.js
│   │   ├── helpers/
│   │   │   ├── htmlToPdf.js
│   │
│   ├── scrapers/
│   │   ├── scraper.js
│   │   ├── indeedScraper.js
│   │   ├── linkedinScraper.js
│   │   ├── glassdoorScraper.js
│   │
│   ├── anti-bot/
│   │   ├── antiBot.js
│   │   ├── flareSolverr.js
│   │   ├── puppeteerExtra.js
│   │   ├── puppeteerRealBrowser.js
│   │   ├── puppeteerCfClearance.js
│   │   ├── undetectedChromedriver.js
│   │   ├── curlImpersonate.js
│   │   ├── userAgents.js
│   │   ├── proxies.js
│   │   ├── warmupScrapers.js
│   │
│   ├── logs/
│   │   ├── error/
│   │   │   ├── error.log
│   │   ├── combined/
│   │   │   ├── combined.log
│   │   ├── screenshots/
│   │
│   └── tests/
│       ├── scrapers/
│       │   ├── test_indeedScraper.js
│       │   ├── test_linkedinScraper.js
│       │   ├── test_glassdoorScraper.js
│       ├── anti-bot/
│       │   ├── test_antiBotMechanisms.js
│       ├── auth/
│       │   ├── test_auth.js
│       ├── selectors/
│       │   ├── test_selectors.js
│       ├── forms/
│       │   ├── test_forms.js
│       ├── navigation/
│       │   ├── test_navigation.js
│
├── config/
│   ├── config.json
│   ├── sampleConfig.json
│   ├── indeedConfig.json
│   ├── linkedinConfig.json
│   ├── glassdoorConfig.json
│
├── .env
├── package.json
└── README.md
Configuration
Global Configuration (config/config.json)
json
Copy code
{
  "scraper": "puppeteer",
  "antiBotMechanism": "puppeteerExtra",
  "rateLimit": 10,
  "monitoring": {
    "enabled": true,
    "loggingLevel": "info"
  },
  "websites": [
    {
      "name": "Indeed",
      "configFile": "indeedConfig.json"
    },
    {
      "name": "LinkedIn",
      "configFile": "linkedinConfig.json"
    },
    {
      "name": "Glassdoor",
      "configFile": "glassdoorConfig.json"
    }
  ]
}

Environment Variables (.env)
makefile
Copy code
NOTION_API_KEY=your_notion_api_key
NOTION_DATABASE_ID=your_database_id
RATE_LIMIT=10
SCRAPER_TYPE=puppeteer
ANTIBOT_MECHANISM=puppeteerExtra
LOGGING_LEVEL=info
Main Components
app.js
Entry point of the application. Initializes the configuration and starts the scraping process.

javascript
Copy code
const config = require('./config');
const scraperFactory = require('./scraperFactory');
const logger = require('./logger');
const monitor = require('./monitor');

async function main() {
  try {
    const scraper = scraperFactory.createScraper(config.scraper);
    await scraper.scrape();
  } catch (error) {
    logger.error('Error in main application: ', error);
  }
}

main();
config.js
Manages global and website-specific settings.

javascript
Copy code
const fs = require('fs');

function loadConfig(file) {
  return JSON.parse(fs.readFileSync(file));
}

const config = loadConfig('./config/config.json');

module.exports = config;
scraperFactory.js
Factory to select and initialize the appropriate scraper.

javascript
Copy code
const IndeedScraper = require('../scrapers/indeedScraper');
const LinkedInScraper = require('../scrapers/linkedinScraper');
const GlassdoorScraper = require('../scrapers/glassdoorScraper');

function createScraper(scraperType) {
  switch (scraperType) {
    case 'indeed':
      return new IndeedScraper();
    case 'linkedin':
      return new LinkedInScraper();
    case 'glassdoor':
      return new GlassdoorScraper();
    default:
      throw new Error(`Unknown scraper type: ${scraperType}`);
  }
}

module.exports = { createScraper };
antiBotFactory.js
Factory to initialize and configure the appropriate anti-bot mechanism.

javascript
Copy code
const puppeteerExtra = require('./puppeteerExtra');
const flareSolverr = require('./flareSolverr');

function createAntiBotMechanism(type) {
  switch (type) {
    case 'puppeteerExtra':
      return puppeteerExtra;
    case 'flareSolverr':
      return flareSolverr;
    default:
      throw new Error(`Unknown anti-bot mechanism: ${type}`);
  }
}

module.exports = { createAntiBotMechanism };
logger.js
Uses the debug library for logging.

javascript
Copy code
const debug = require('debug');

const logger = {
  info: debug('info'),
  error: debug('error'),
  debug: debug('debug')
};

module.exports = logger;
monitor.js
Uses PM2 for monitoring.

javascript
Copy code
const pm2 = require('pm2');

function startMonitoring() {
  pm2.connect(function(err) {
    if (err) {
      console.error(err);
      process.exit(2);
    }

    pm2.start({
      script: 'src/main/app.js',
      name: 'job-automator'
    }, function(err, apps) {
      pm2.disconnect();
      if (err) throw err;
    });
  });
}

module.exports = startMonitoring;
Authentication and Session Management
loginHandler.js
Handles the login processes for different websites.

javascript
Copy code
const puppeteer = require('puppeteer');

async function login(url, username, password) {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto(url);

  await page.type('input[name="username"]', username);
  await page.type('input[name="password"]', password);
  await page.click('button[type="submit"]');

  await page.waitForNavigation();

  const cookies = await page.cookies();
  await browser.close();
  return cookies;
}

module.exports = login;
cookieManager.js
Manages cookies for session persistence.

javascript
Copy code
const fs = require('fs');

function saveCookies(cookies, filepath) {
  fs.writeFileSync(filepath, JSON.stringify(cookies));
}

function loadCookies(filepath) {
  const cookies = fs.readFileSync(filepath);
  return JSON.parse(cookies);
}

module.exports = { saveCookies, loadCookies };
sessionManager.js
Ensures sessions are maintained across different scraping tasks.

javascript
Copy code
const puppeteer = require('puppeteer');
const { loadCookies } = require('./cookieManager');

async function restoreSession(url, cookieFilePath) {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  const cookies = loadCookies(cookieFilePath);
  await page.setCookie(...cookies);
  await page.goto(url);

  await browser.close();
}

module.exports = restoreSession;

Selectors

heuristicFinder.js

Implements heuristic-based selector detection.

javascript
Copy code
const { JSDOM } = require('jsdom');

function findSelectors(htmlContent) {
  const dom = new JSDOM(htmlContent);
  const document = dom.window.document;
  const username = document.querySelector('input[type="text"], input[type="email"]');
  const password = document.querySelector('input[type="password"]');
  const submit = document.querySelector('input[type="submit"], button[type="submit"]');
  return { username, password, submit };
}

module.exports = findSelectors;
seleniumFinder.js
Uses Selenium for selector finding.

javascript
Copy code
const { Builder, By } = require('selenium-webdriver');

async function findSelectors(url) {
  let driver = await new Builder().forBrowser('chrome').build();
  try {
    await driver.get(url);
    const username = await driver.findElement(By.name('username')).catch(() => driver.findElement(By.xpath("//input[@type='text' or @type='email']")));
    const password = await driver.findElement(By.name('password')).catch(() => driver.findElement(By.xpath("//input[@type='password']")));
    const submit = await driver.findElement(By.name('login')).catch(() => driver.findElement(By.xpath("//input[@type='submit' or @type='button']")));
    return { username, password, submit };
  } finally {
    await driver.quit();
  }
}

module.exports = findSelectors;
manualInput.js
Allows manual selector input.

javascript
Copy code
function manualSelectorInput() {
  const readline = require('readline').createInterface({
    input: process.stdin,
    output: process.stdout
  });

  return new Promise((resolve) => {
    readline.question("Enter the CSS/XPath selector for the username field: ", (usernameSelector) => {
      readline.question("Enter the CSS/XPath selector for the password field: ", (passwordSelector) => {
        readline.question("Enter the CSS/XPath selector for the submit button: ", (submitSelector) => {
          resolve({ usernameSelector, passwordSelector, submitSelector });
          readline.close();
        });
      });
    });
  });
}

module.exports = manualSelectorInput;
Form Handlers
jobApplicationHandler.js
Handles job application submissions.

javascript
Copy code
const puppeteer = require('puppeteer');
const { addJobToNotion } = require('./notionIntegration');
const databaseId = process.env.NOTION_DATABASE_ID;

async function handleJobApplication(jobDetails) {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto(jobDetails.url);

  await page.type('input[name="username"]', jobDetails.username);
  await page.type('input[name="password"]', jobDetails.password);
  await page.click('button[type="submit"]');
  await page.waitForNavigation();

  await addJobToNotion(databaseId, {
    title: jobDetails.title,
    company: jobDetails.company,
    status: 'Applied',
    url: jobDetails.url,
    appliedDate: new Date().toISOString(),
  });

  await browser.close();
  console.log('Job application submitted and tracked in Notion');
}

module.exports = handleJobApplication;
genericFormHandler.js
Handles generic form submissions.

javascript
Copy code
async function handleGenericForm(page, formDetails) {
  for (const field of formDetails.fields) {
    await page.type(field.selector, field.value);
  }
  await page.click(formDetails.submitButton);
  await page.waitForNavigation();
}

module.exports = handleGenericForm;
notionIntegration.js
Handles Notion API interactions.

javascript
Copy code
const { Client } = require('@notionhq/client');

const notion = new Client({ auth: process.env.NOTION_API_KEY });

async function addJobToNotion(databaseId, jobDetails) {
    try {
        const response = await notion.pages.create({
            parent: { database_id: databaseId },
            properties: {
                Title: {
                    title: [
                        {
                            text: {
                                content: jobDetails.title,
                            },
                        },
                    ],
                },
                Company: {
                    rich_text: [
                        {
                            text: {
                                content: jobDetails.company,
                            },
                        },
                    ],
                },
                Status: {
                    select: {
                        name: jobDetails.status,
                    },
                },
                Link: {
                    url: jobDetails.url,
                },
                AppliedDate: {
                    date: {
                        start: jobDetails.appliedDate,
                    },
                },
            },
        });
        console.log('Job added to Notion:', response);
    } catch (error) {
        console.error('Error adding job to Notion:', error);
    }
}

module.exports = { addJobToNotion };
Navigation
navigationHandler.js
Handles navigation tasks.

javascript
Copy code
async function navigateToJobPage(page, jobUrl) {
  await page.goto(jobUrl);
}

module.exports = navigateToJobPage;
Helpers
htmlToPdf.js
Converts HTML to PDF for uploads.

javascript
Copy code
const puppeteer = require('puppeteer');

async function convertHtmlToPdf(htmlContent, outputPath) {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.setContent(htmlContent);
  await page.pdf({ path: outputPath, format: 'A4' });
  await browser.close();
}

module.exports = convertHtmlToPdf;
Scrapers
scraper.js
Base scraper class.

javascript
Copy code
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
indeedScraper.js
Implements scraping logic specific to Indeed.

javascript
Copy code
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
linkedinScraper.js
Implements scraping logic specific to LinkedIn.

javascript
Copy code
const Scraper = require('./scraper');
const puppeteer = require('puppeteer');

class LinkedInScraper extends Scraper {
  constructor(config) {
    super(config);
  }

  async scrape() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.goto(this.config.url);

    // Implement LinkedIn-specific scraping logic
    const jobListings = await page.evaluate(() => {
      const listings = [];
      document.querySelectorAll('.job-card-container').forEach(jobCard => {
        const title = jobCard.querySelector('.job-card-list__title').innerText;
        const company = jobCard.querySelector('.job-card-container__company-name').innerText;
        listings.push({ title, company });
      });
      return listings;
    });

    await browser.close();
    return jobListings;
  }
}

module.exports = LinkedInScraper;
glassdoorScraper.js
Implements scraping logic specific to Glassdoor.

javascript
Copy code
const Scraper = require('./scraper');
const puppeteer = require('puppeteer');

class GlassdoorScraper extends Scraper {
  constructor(config) {
    super(config);
  }

  async scrape() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.goto(this.config.url);

    // Implement Glassdoor-specific scraping logic
    const jobListings = await page.evaluate(() => {
      const listings = [];
      document.querySelectorAll('.jl').forEach(jobCard => {
        const title = jobCard.querySelector('.jobTitle span').innerText;
        const company = jobCard.querySelector('.jobEmpolyerName').innerText;
        listings.push({ title, company });
      });
      return listings;
    });

    await browser.close();
    return jobListings;
  }
}

module.exports = GlassdoorScraper;
Anti-Bot Mechanisms
antiBot.js
Base anti-bot class.

javascript
Copy code
class AntiBot {
  constructor(config) {
    this.config = config;
  }

  async init() {
    // Initialize anti-bot mechanism with necessary configurations
  }

  async solveChallenge() {
    // Implement anti-bot challenge solving logic
  }
}

module.exports = AntiBot;
puppeteerExtra.js
Integration with puppeteer-extra.

javascript
Copy code
const puppeteer = require('puppeteer-extra');
const StealthPlugin = require('puppeteer-extra-plugin-stealth');

puppeteer.use(StealthPlugin());

async function createBrowser() {
  const browser = await puppeteer.launch({ headless: true });
  return browser;
}

module.exports = createBrowser;
flareSolverr.js
Integration with FlareSolverr.

javascript
Copy code
const axios = require('axios');

async function solveChallenge(url) {
  const response = await axios.post('http://localhost:8191/v1', {
    cmd: 'request.get',
    url: url
  });
  return response.data;
}

module.exports = solveChallenge;
Logging and Monitoring
logger.js
Uses the debug library for logging.

javascript
Copy code
const debug = require('debug');

const logger = {
  info: debug('info'),
  error: debug('error'),
  debug: debug('debug')
};

module.exports = logger;
monitor.js
Uses PM2 for monitoring.

javascript
Copy code
const pm2 = require('pm2');

function startMonitoring() {
  pm2.connect(function(err) {
    if (err) {
      console.error(err);
      process.exit(2);
    }

    pm2.start({
      script: 'src/main/app.js',
      name: 'job-automator'
    }, function(err, apps) {
      pm2.disconnect();
      if (err) throw err;
    });
  });
}

module.exports = startMonitoring;
Testing
Jest Testing Framework
Install Jest:

bash
Copy code
npm install --save-dev jest
Configure Jest in package.json:

json
Copy code
{
  "scripts": {
    "test": "jest"
  }
}
Example Test (test_indeedScraper.js)
javascript
Copy code
const IndeedScraper = require('../scrapers/indeedScraper');

test('IndeedScraper should scrape job listings', async () => {
  const config = { url: 'https://www.indeed.com/jobs?q=software+developer' };
  const scraper = new IndeedScraper(config);

  const jobListings = await scraper.scrape();
  expect(jobListings.length).toBeGreaterThan(0);
  expect(jobListings[0]).toHaveProperty('title');
  expect(jobListings[0]).toHaveProperty('company');
});

This Blueprint provides detailed guidance and code snippets to help build the Job-Automator-V2 project. Each component is described with sample code and configuration details, ensuring clarity and context throughout the development process.