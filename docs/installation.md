Installation.md
Installation Guide
Prerequisites
Node.js
npm
Steps
Clone the Repository: Clone the project repository from GitHub to your local machine.

bash
Copy code
git clone https://github.com/yourusername/job-automator-v2.git
cd job-automator-v2
Install Dependencies: Navigate to the project directory and install all necessary dependencies using npm.

bash
Copy code
npm install
Setup Configuration: Configure the application settings by editing the configuration files as needed.

Environment Variables: Create a .env file in the root directory to manage environment variables.

plaintext
Copy code
NOTION_API_KEY=your_notion_api_key
NOTION_DATABASE_ID=your_database_id
RATE_LIMIT=10
SCRAPER_TYPE=puppeteer
ANTIBOT_MECHANISM=puppeteerExtra
LOGGING_LEVEL=info