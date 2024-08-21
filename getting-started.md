---
excerpt: "Quick start guide for Pixashot, covering installation, authentication, and basic usage of the web screenshot service API."
published_at: true
---

# Getting Started

## Introduction

Pixashot is a powerful, flexible, and developer-friendly web screenshot service that simplifies the process of capturing high-quality screenshots. Whether you need to automate web testing, generate thumbnails, or create visual archives of web pages, Pixashot provides a robust solution with its easy-to-use API.

Key features of Pixashot include:

- High-quality screenshots at any resolution, including Retina displays
- Full-page captures and custom viewport sizes
- Multiple output formats (PNG, JPEG, WebP, PDF)
- Custom JavaScript injection for page manipulation
- Built-in popup and cookie consent blockers
- Flexible deployment options (self-hosted or cloud-based)

## Quick Start Guide

To get started with Pixashot quickly, follow these steps:

1. Install Pixashot (see Installation section below)
2. Set up authentication
3. Make your first API call

Here's a simple example using Node.js and axios to capture a screenshot:

```javascript
const axios = require('axios');
const fs = require('fs');

const PIXASHOT_API_URL = 'http://your-pixashot-instance.com/capture';
const API_KEY = 'your_api_key_here';

axios.post(PIXASHOT_API_URL, {
  url: 'https://example.com',
  format: 'png',
  full_page: true
}, {
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Content-Type': 'application/json'
  },
  responseType: 'arraybuffer'
})
.then(response => {
  fs.writeFileSync('screenshot.png', response.data);
  console.log('Screenshot saved as screenshot.png');
})
.catch(error => {
  console.error('Error capturing screenshot:', error.message);
});
```

## Installation

Pixashot can be installed and run either using Docker or as a self-hosted application.

### Docker Installation

To install and run Pixashot using Docker:

1. Pull the Pixashot Docker image:

```bash
docker pull gpriday/pixashot:latest
```

2. Run the container:

```bash
docker run -p 8080:8080 -e AUTH_TOKEN=your_secret_token gpriday/pixashot:latest
```

Replace `your_secret_token` with a secure authentication token of your choice.

### Self-hosted Installation

To install Pixashot as a self-hosted application:

1. Clone the Pixashot repository:

```bash
git clone https://github.com/yourusername/pixashot.git
cd pixashot
```

2. Install dependencies:

```bash
npm install
```

3. Set up environment variables:

Create a `.env` file in the project root and add the following:

```
AUTH_TOKEN=your_secret_token
PORT=8080
```

4. Start the application:

```bash
npm start
```

## Authentication

Pixashot uses token-based authentication to secure API access. You need to include this token in the `Authorization` header of your API requests.

There are two ways to authenticate with Pixashot:

1. **Bearer Token**: Include the `AUTH_TOKEN` in the `Authorization` header of your requests:

```
Authorization: Bearer your_secret_token
```

2. **Signed URLs**: For scenarios where you can't include headers (e.g., image tags), you can use signed URLs. Here's an example of generating a signed URL using Node.js:

```javascript
const crypto = require('crypto');

function generateSignedUrl(baseUrl, params, secretKey, expiresIn = 3600) {
  const expires = Math.floor(Date.now() / 1000) + expiresIn;
  const signature = crypto.createHmac('sha256', secretKey)
    .update(`${baseUrl}?${new URLSearchParams(params)}&expires=${expires}`)
    .digest('hex');
  
  return `${baseUrl}?${new URLSearchParams(params)}&expires=${expires}&signature=${signature}`;
}

const signedUrl = generateSignedUrl(
  'http://your-pixashot-instance.com/capture',
  { url: 'https://example.com', format: 'png' },
  'your_secret_key'
);
console.log(signedUrl);
```

Remember to keep your `AUTH_TOKEN` and secret key secure and never expose them in client-side code.

With these steps, you should be ready to start using Pixashot for capturing web screenshots. Explore the rest of the documentation to learn about advanced features and customization options.