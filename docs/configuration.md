### Configuration.md

## Configuration Guide

### Global Configuration
- **scraper**: The type of scraper to use (e.g., Puppeteer, Selenium).
- **antiBotMechanism**: The anti-bot mechanism to use (e.g., Puppeteer-extra, FlareSolverr).
- **rateLimit**: Sets the rate limit for requests.
- **monitoring**: Configuration for enabling and setting up monitoring.

### Website-Specific Configuration
- Separate JSON files for each target website.
- Includes settings specific to each website’s scraping logic, form submission details, and navigation requirements.

### Example Configuration (config.json)
```json
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
