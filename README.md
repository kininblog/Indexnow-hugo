[![Automated IndexNow Integration for Hugo & Netlify](https://akazed.com/panduan-integrasi-indexnow-hugo/images/indexnow_hua650d485e6aa5989042dc4b35e5c97c0_39690_1600x0_resize_q75_h2_box_2.webp)](https://akazed.com/panduan-integrasi-indexnow-hugo/)

# Automated IndexNow Integration for Hugo on Netlify

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Works with Hugo](https://img.shields.io/badge/Works%20with-Hugo-ff4088.svg)](https://gohugo.io/)
![Powered by Netlify](https://img.shields.io/badge/Powered%20by-Netlify-00C7B7.svg)

A modern and efficient method for automating URL submissions to the IndexNow API every time your Hugo site is successfully deployed on **Netlify**.

This solution uses **serverless Netlify Functions**, eliminating the need for external scripts or complex CI/CD processes.

This project is based entirely on the original guide written in Indonesian by **[Akazed](httpss://akazed.com/panduan-integrasi-indexnow-hugo/)**.

---

## Table of Contents

- [Why Use This Method?](#why-use-this-method)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Setup and Installation Steps](#setup-and-installation-steps)
  - [1. Get Your IndexNow API Key](#1-get-your-indexnow-api-key)
  - [2. Set Environment Variables on Netlify](#2-set-environment-variables-on-netlify)
  - [3. Create the Netlify Function](#3-create-the-netlify-function)
  - [4. Configure Netlify to Run the Function](#4-configure-netlify-to-run-the-function)
- [Triggering a Submission](#triggering-a-submission)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Why Use This Method?

-   **🚀 Fully Integrated**: No need for third-party services or external scripts. Everything runs within your Netlify ecosystem.
-   **⚙️ Serverless**: No server to manage. This function only runs when needed after a successful deployment.
-   **💡 Efficient and Automated**: Set it up once. Every time you `git push` and Netlify successfully deploys your site, the function runs automatically.
-   **⚡ Fast and Lightweight**: Uses JavaScript (Node.js), which is extremely fast for simple tasks like API calls.

## How It Works

This workflow is clean and automated:
1.  You `git push` changes to your Hugo content.
2.  Netlify detects the change and begins its build & deploy process.
3.  After your site is successfully deployed, Netlify's `onSuccess` event is triggered.
4.  This event automatically executes the Netlify Function (`indexnow.js`) we've prepared.
5.  The function retrieves your `SITEMAP_URL` and `INDEXNOW_API_KEY` from Netlify's environment variables.
6.  Finally, the function sends a GET request to the IndexNow API with your sitemap URL and API key, telling search engines to index your site.

## Prerequisites

-   A Hugo site hosted and deployed via **Netlify**.
-   An account on [Bing Webmaster Tools](https://www.bing.com/webmasters/) to get an API Key.

## Setup and Installation Steps

Follow these steps to integrate IndexNow with your site.

### 1. Get Your IndexNow API Key

-   Log in to **Bing Webmaster Tools**.
-   Navigate to `Settings` > `API Access`.
-   Click `API Key` and `Generate API Key`.
-   Copy the key. We will use this in the next step.

### 2. Set Environment Variables on Netlify

Never put your API key directly in your code. Use Netlify's environment variables.
1.  Go to your site's dashboard on Netlify.
2.  Navigate to `Site configuration` > `Build & deploy` > `Environment`.
3.  Click `Edit variables` and add the following two variables:
    -   **Key**: `INDEXNOW_API_KEY`
        -   **Value**: (Paste the API Key you copied from Bing Webmaster Tools)
    -   **Key**: `SITEMAP_URL`
        -   **Value**: `https://your-domain.com/sitemap.xml` (Replace with your sitemap URL)

### 3. Create the Netlify Function

1.  In the root directory of your Hugo project, create a new folder structure: `netlify/functions/`.
2.  Inside the `functions` folder, create a new file named `indexnow.js`.
3.  Copy and paste the following JavaScript code into `indexnow.js`:

```javascript
// netlify/functions/indexnow.js

// We use an async import for node-fetch as it's an ES module.
// Netlify will bundle it correctly during the build process.
exports.handler = async function(event, context) {
  const fetch = (await import('node-fetch')).default;

  const apiKey = process.env.INDEXNOW_API_KEY;
  const sitemapUrl = process.env.SITEMAP_URL;

  // Validate that environment variables are set
  if (!apiKey || !sitemapUrl) {
    const errorMessage = 'INDEXNOW_API_KEY or SITEMAP_URL environment variables are not set in Netlify.';
    console.error(errorMessage);
    return {
      statusCode: 500,
      body: errorMessage,
    };
  }

  const host = new URL(sitemapUrl).hostname;
  const indexNowApiUrl = `https://api.indexnow.org/indexnow?url=${encodeURIComponent(sitemapUrl)}&key=${apiKey}`;

  console.log(`Attempting to ping IndexNow for: ${sitemapUrl}`);

  try {
    const response = await fetch(indexNowApiUrl);

    if (response.ok) {
      const successMessage = `Successfully pinged IndexNow API. Status: ${response.status}`;
      console.log(successMessage);
      return {
        statusCode: response.status,
        body: successMessage,
      };
    } else {
      const errorBody = await response.text();
      const errorMessage = `Failed to submit to IndexNow. Status: ${response.status}. Message: ${errorBody}`;
      console.error(errorMessage);
      return {
        statusCode: response.status,
        body: errorMessage,
      };
    }
  } catch (error) {
    const errorMessage = `An error occurred while calling the IndexNow API: ${error.message}`;
    console.error(errorMessage);
    return {
      statusCode: 500,
      body: errorMessage,
    };
  }
};
