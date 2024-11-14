---
excerpt: "Practical examples and use cases for Pixashot, including e-commerce product captures, social media previews, web archiving, and automated testing."
published_at: true
---

# Examples and Use Cases

Pixashot provides powerful screenshot capabilities that can be applied across various industries. Here are practical examples showing how to integrate Pixashot into different workflows.

## E-commerce Product Captures

Generate high-quality product images automatically for your e-commerce platform.

**Use Case**: Automatically capture product images with consistent styling.

```python
import requests
from time import sleep
import json

def capture_product_image(product_url, product_id, retries=3):
    """Capture product image with retry logic and error handling."""
    pixashot_url = "https://api.example.com/capture"
    payload = {
        "url": product_url,
        "selector": "#product-main-image",
        "format": "png",
        "window_width": 1200,
        "window_height": 800,
        "wait_for_network": "idle",
        "block_media": False,       # We need images for products
        "custom_js": """
            // Remove unnecessary elements
            document.querySelectorAll('.popup, .cookie-notice, .chat-widget')
                .forEach(el => el.remove());
            
            // Ensure product image is loaded
            await new Promise(resolve => {
                const img = document.querySelector('#product-main-image img');
                if (img && img.complete) resolve();
                else if (img) {
                    img.onload = resolve;
                    img.onerror = resolve;
                } else resolve();
            });
        """
    }
    headers = {
        "Authorization": f"Bearer {YOUR_API_KEY}",
        "Content-Type": "application/json"
    }

    for attempt in range(retries):
        try:
            response = requests.post(pixashot_url, 
                                   json=payload, 
                                   headers=headers)
            
            if response.status_code == 200:
                filename = f"product_{product_id}.png"
                with open(filename, "wb") as f:
                    f.write(response.content)
                print(f"Product image saved: {filename}")
                return filename
                
            elif response.status_code == 429:  # Rate limit
                sleep((attempt + 1) * 2)  # Exponential backoff
                continue
                
        except Exception as e:
            if attempt == retries - 1:
                raise e
            sleep((attempt + 1) * 2)

    raise Exception(f"Failed to capture product image after {retries} attempts")

# Usage
try:
    capture_product_image(
        "https://yourstore.com/products/awesome-gadget",
        "12345"
    )
except Exception as e:
    print(f"Error: {str(e)}")
```

## Social Media Previews

Generate dynamic social media preview images for blog posts and articles.

```javascript
const axios = require('axios');
const fs = require('fs').promises;

async function generateSocialPreview(blogUrl, title, author) {
    const pixashotUrl = 'https://api.example.com/capture';
    const payload = {
        url: 'https://your-preview-generator.com',
        format: 'png',
        window_width: 1200,
        window_height: 630,
        wait_for_network: 'idle',
        pixel_density: 2.0,  // Higher quality for social
        custom_js: `
            // Insert content
            document.getElementById('blog-title').textContent = ${JSON.stringify(title)};
            document.getElementById('blog-author').textContent = ${JSON.stringify(author)};
            document.getElementById('blog-url').textContent = ${JSON.stringify(blogUrl)};
            
            // Wait for fonts to load
            await document.fonts.ready;
            
            // Wait for any animations
            await new Promise(resolve => setTimeout(resolve, 500));
        `
    };

    try {
        const response = await axios.post(pixashotUrl, payload, {
            headers: {
                'Authorization': `Bearer ${process.env.PIXASHOT_API_KEY}`,
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer',
            timeout: 15000
        });

        const filename = `${title.replace(/[^a-z0-9]/gi, '-').toLowerCase()}-og.png`;
        await fs.writeFile(filename, response.data);
        
        return {
            success: true,
            filename,
            size: response.data.length
        };
    } catch (error) {
        console.error('Preview generation failed:', error.message);
        throw error;
    }
}

// Usage with async/await
async function createPreview() {
    try {
        const result = await generateSocialPreview(
            'https://blog.example.com/post/123',
            'Understanding Web Performance',
            'Jane Smith'
        );
        console.log('Preview generated:', result.filename);
    } catch (error) {
        console.error('Failed:', error.message);
    }
}
```

## Web Archiving

Create visual archives of web pages with robust error handling and metadata.

```python
import requests
from datetime import datetime
import json
import os
from urllib.parse import urlparse

class WebArchiver:
    def __init__(self, api_key, output_dir="archives"):
        self.api_key = api_key
        self.output_dir = output_dir
        os.makedirs(output_dir, exist_ok=True)

    async def archive_page(self, url, full_page=True):
        """Archive a webpage with metadata."""
        pixashot_url = "https://api.example.com/capture"
        timestamp = datetime.now().isoformat()
        domain = urlparse(url).netloc

        payload = {
            "url": url,
            "format": "png",
            "full_page": full_page,
            "window_width": 1920,
            "window_height": 1080,
            "wait_for_network": "mostly_idle",
            "wait_for_timeout": 10000,
            "block_media": False,
            "custom_js": """
                // Remove dynamic elements that might change
                document.querySelectorAll('[data-dynamic], .ad, .timestamp')
                    .forEach(el => el.remove());
                
                // Wait for lazy-loaded content
                await new Promise(resolve => {
                    let lastHeight = 0, checks = 0;
                    const interval = setInterval(() => {
                        const height = document.body.scrollHeight;
                        if (height === lastHeight || checks > 20) {
                            clearInterval(interval);
                            resolve();
                        }
                        lastHeight = height;
                        checks++;
                        window.scrollTo(0, height);
                    }, 250);
                });
                
                // Scroll back to top
                window.scrollTo(0, 0);
            """
        }

        try:
            response = requests.post(
                pixashot_url,
                json=payload,
                headers={
                    "Authorization": f"Bearer {self.api_key}",
                    "Content-Type": "application/json"
                }
            )
            
            if response.status_code == 200:
                # Save screenshot
                filename = f"{domain}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.png"
                filepath = os.path.join(self.output_dir, filename)
                
                with open(filepath, "wb") as f:
                    f.write(response.content)
                
                # Save metadata
                metadata = {
                    "url": url,
                    "timestamp": timestamp,
                    "filename": filename,
                    "full_page": full_page,
                    "size_bytes": len(response.content)
                }
                
                metadata_path = f"{filepath}.json"
                with open(metadata_path, "w") as f:
                    json.dump(metadata, f, indent=2)
                
                return {
                    "success": True,
                    "filepath": filepath,
                    "metadata": metadata
                }
        
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }

# Usage
archiver = WebArchiver(api_key="your_api_key")
result = await archiver.archive_page("https://www.example.com")
```

## Visual Regression Testing

Implement automated visual testing in your CI/CD pipeline.

```javascript
const axios = require('axios');
const fs = require('fs').promises;
const pixelmatch = require('pixelmatch');
const PNG = require('pngjs').PNG;

class VisualTester {
    constructor(apiKey, options = {}) {
        this.apiKey = apiKey;
        this.threshold = options.threshold || 0.1;
        this.pixashotUrl = 'https://api.example.com/capture';
    }

    async captureScreen(url, name) {
        const payload = {
            url: url,
            format: 'png',
            window_width: 1280,
            window_height: 800,
            wait_for_network: 'idle',
            pixel_density: 1,
            custom_js: `
                // Remove dynamic content
                document.querySelectorAll('[data-dynamic], .date, .ad')
                    .forEach(el => el.remove());
                
                // Wait for fonts
                await document.fonts.ready;
                
                // Wait for animations to complete
                await new Promise(resolve => setTimeout(resolve, 1000));
            `
        };

        const response = await axios.post(this.pixashotUrl, payload, {
            headers: {
                'Authorization': `Bearer ${this.apiKey}`,
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer'
        });

        const filename = `${name}_${Date.now()}.png`;
        await fs.writeFile(filename, response.data);
        return filename;
    }

    async compareScreenshots(filename1, filename2) {
        const img1 = PNG.sync.read(await fs.readFile(filename1));
        const img2 = PNG.sync.read(await fs.readFile(filename2));
        const {width, height} = img1;
        const diff = new PNG({width, height});

        const numDiffPixels = pixelmatch(
            img1.data,
            img2.data,
            diff.data,
            width,
            height,
            {
                threshold: this.threshold,
                includeAA: true
            }
        );

        const diffFile = `diff_${Date.now()}.png`;
        await fs.writeFile(diffFile, PNG.sync.write(diff));

        return {
            diffPixels: numDiffPixels,
            diffPercentage: (numDiffPixels / (width * height)) * 100,
            diffFile
        };
    }

    async runTest(productionUrl, stagingUrl, name) {
        try {
            const [prodScreen, stagingScreen] = await Promise.all([
                this.captureScreen(productionUrl, `${name}_prod`),
                this.captureScreen(stagingUrl, `${name}_staging`)
            ]);

            const comparison = await this.compareScreenshots(prodScreen, stagingScreen);
            
            // Cleanup screenshots but keep diff if there are changes
            if (comparison.diffPercentage < 1) {
                await Promise.all([
                    fs.unlink(prodScreen),
                    fs.unlink(stagingScreen),
                    fs.unlink(comparison.diffFile)
                ]);
            }

            return comparison;

        } catch (error) {
            console.error('Visual test failed:', error);
            throw error;
        }
    }
}

// Usage in CI/CD
async function runVisualTests() {
    const tester = new VisualTester(process.env.PIXASHOT_API_KEY);
    
    try {
        const results = await tester.runTest(
            'https://www.example.com',
            'https://staging.example.com',
            'homepage'
        );

        if (results.diffPercentage > 1) {
            console.error(`Visual differences detected: ${results.diffPercentage}%`);
            console.error(`Diff file: ${results.diffFile}`);
            process.exit(1);
        }

        console.log('Visual test passed');

    } catch (error) {
        console.error('Test failed:', error);
        process.exit(1);
    }
}
```

These examples showcase best practices for using Pixashot, including:
- Proper error handling
- Retry logic
- Resource optimization
- Comprehensive wait strategies
- Metadata tracking
- Cleanup routines

Remember that Pixashot uses a single browser context, so:
- Configure extensions via environment variables
- Consider concurrent request limits
- Implement appropriate error handling
- Use caching when possible

For more details on configuration options, see the [API Reference](api-reference.md).