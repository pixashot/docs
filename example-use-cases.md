---
excerpt: "Practical examples and use cases for Pixashot, including e-commerce product captures, social media previews, web archiving, and automated testing."
published_at: true
---

# Examples and Use Cases

Pixashot is a versatile tool that can be applied to various scenarios across different industries. This section explores some common use cases and provides examples of how Pixashot can be implemented to solve specific challenges.

## E-commerce Product Captures

Pixashot can significantly enhance the product presentation in e-commerce platforms by automating the capture of product pages.

**Use Case**: Automatically generate product images for an online store.

**Example Implementation**:
```python
import requests

def capture_product_image(product_url, product_id):
    pixashot_url = "https://api.example.com/capture"
    payload = {
        "url": product_url,
        "selector": "#product-main-image",
        "format": "png",
        "window_width": 1200,
        "window_height": 800
    }
    headers = {
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    }

    response = requests.post(pixashot_url, json=payload, headers=headers)
    if response.status_code == 200:
        with open(f"product_{product_id}.png", "wb") as f:
            f.write(response.content)
        print(f"Product image saved for product ID: {product_id}")
    else:
        print(f"Error capturing product image: {response.status_code}")

# Usage
capture_product_image("https://yourstore.com/products/awesome-gadget", "12345")
```

This script captures a specific product image element from a product page, which can be used for thumbnails, catalog images, or social media sharing.

## Social Media Previews

Generate eye-catching preview images for social media posts to increase engagement.

**Use Case**: Create custom Open Graph images for blog posts.

**Example Implementation**:
```javascript
const axios = require('axios');
const fs = require('fs');

async function generateSocialPreview(blogUrl, title, author) {
    const pixashotUrl = 'https://api.example.com/capture';
    const payload = {
        url: 'https://your-preview-generator.com',
        format: 'png',
        window_width: 1200,
        window_height: 630,
        custom_js: `
            document.getElementById('blog-title').textContent = "${title}";
            document.getElementById('blog-author').textContent = "${author}";
            document.getElementById('blog-url').textContent = "${blogUrl}";
        `
    };

    try {
        const response = await axios.post(pixashotUrl, payload, {
            headers: {
                'Authorization': 'Bearer YOUR_API_KEY',
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer'
        });

        fs.writeFileSync(`${title.replace(/\s+/g, '-')}-og-image.png`, response.data);
        console.log('Social media preview generated successfully');
    } catch (error) {
        console.error('Error generating preview:', error);
    }
}

// Usage
generateSocialPreview(
    'https://yourblog.com/awesome-post',
    'The Ultimate Guide to Awesome Things',
    'Jane Doe'
);
```

This script generates a custom Open Graph image by injecting blog post details into a template page and capturing the result.

## Web Archiving

Pixashot can be used to create visual archives of web pages, preserving their appearance at specific points in time.

**Use Case**: Periodically archive the homepage of a news website.

**Example Implementation**:
```python
import requests
from datetime import datetime

def archive_homepage(url):
    pixashot_url = "https://api.example.com/capture"
    payload = {
        "url": url,
        "format": "png",
        "full_page": True,
        "window_width": 1920,
        "window_height": 1080
    }
    headers = {
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    }

    response = requests.post(pixashot_url, json=payload, headers=headers)
    if response.status_code == 200:
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"archive_{timestamp}.png"
        with open(filename, "wb") as f:
            f.write(response.content)
        print(f"Homepage archived: {filename}")
    else:
        print(f"Error archiving homepage: {response.status_code}")

# Usage
archive_homepage("https://www.newswebsite.com")
```

This script captures full-page screenshots of a website's homepage, which can be run periodically to create a visual archive.

## Automated Testing

Integrate Pixashot into your testing pipeline to catch visual regressions and ensure consistent UI across different environments.

**Use Case**: Perform visual regression testing as part of a CI/CD pipeline.

**Example Implementation**:
```javascript
const axios = require('axios');
const fs = require('fs');
const { execSync } = require('child_process');

async function performVisualTest(testUrl, baselineImage) {
    const pixashotUrl = 'https://api.example.com/capture';
    const payload = {
        url: testUrl,
        format: 'png',
        full_page: true
    };

    try {
        const response = await axios.post(pixashotUrl, payload, {
            headers: {
                'Authorization': 'Bearer YOUR_API_KEY',
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer'
        });

        const currentImage = 'current_snapshot.png';
        fs.writeFileSync(currentImage, response.data);

        // Compare images using ImageMagick (assuming it's installed)
        const diff = execSync(`compare -metric AE ${baselineImage} ${currentImage} diff.png`);
        
        if (parseInt(diff.toString()) > 0) {
            console.log('Visual differences detected. Check diff.png for details.');
            process.exit(1);
        } else {
            console.log('No visual differences detected.');
            fs.unlinkSync(currentImage);
            fs.unlinkSync('diff.png');
        }
    } catch (error) {
        console.error('Error in visual testing:', error);
        process.exit(1);
    }
}

// Usage
performVisualTest('https://your-staging-site.com', 'baseline.png');
```

This script captures a screenshot of a staging environment and compares it to a baseline image, flagging any visual differences.

These examples demonstrate the versatility of Pixashot across various use cases. By leveraging Pixashot's features, you can automate visual tasks, enhance your web applications, and gain valuable insights into your online presence. Remember to customize these examples to fit your specific needs and always adhere to the terms of service of the websites you're capturing.