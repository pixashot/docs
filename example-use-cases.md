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
        "pixel_density": 2.0,  # High quality for products
        "wait_for_animation": True,
        "block_media": False,  # We need product images
        "interactions": [
            {
                "action": "wait_for",
                "wait_for": {
                    "type": "selector",
                    "value": "#product-main-image img"
                }
            },
            {
                "action": "wait_for",
                "wait_for": {
                    "type": "network_idle",
                    "value": 2000
                }
            },
            {
                "action": "hover",
                "selector": "#product-main-image"  # Trigger any hover effects
            }
        ],
        "custom_js": """
            // Remove unnecessary elements
            document.querySelectorAll('.popup, .cookie-notice, .chat-widget')
                .forEach(el => el.remove());
            
            // Force high-quality images
            document.querySelectorAll('img[srcset]').forEach(img => {
                const sources = img.srcset.split(',');
                const highest = sources
                    .map(s => ({
                        url: s.trim().split(' ')[0],
                        width: parseInt(s.trim().split(' ')[1] || '0')
                    }))
                    .sort((a, b) => b.width - a.width)[0];
                if (highest) img.src = highest.url;
            });
            
            // Ensure consistent background
            document.body.style.backgroundColor = 'white';
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
        template: 'social_preview',  // Use predefined template
        window_width: 1200,
        window_height: 630,
        pixel_density: 2.0,  // Higher quality for social
        wait_for_animation: true,
        interactions: [
            {
                action: 'type',
                selector: '#blog-title',
                text: title
            },
            {
                action: 'type',
                selector: '#blog-author',
                text: author
            },
            {
                action: 'type',
                selector: '#blog-url',
                text: blogUrl
            },
            {
                action: 'wait_for',
                wait_for: {
                    type: 'network_idle',
                    value: 2000
                }
            }
        ],
        custom_js: `
            // Wait for fonts to load
            await document.fonts.ready;
            
            // Ensure proper rendering
            document.body.style.overflow = 'hidden';
            document.documentElement.style.overflow = 'hidden';
        `
    };

    try {
        const response = await axios.post(pixashotUrl, payload, {
            headers: {
                'Authorization': `Bearer ${process.env.PIXASHOT_API_KEY}`,
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer'
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
            "format": "pdf",  # Use PDF for archives
            "full_page": full_page,
            "window_width": 1920,
            "window_height": 1080,
            "pdf_format": "A4",
            "pdf_print_background": True,
            "wait_for_network": "mostly_idle",
            "wait_for_animation": True,
            "interactions": [
                {
                    "action": "wait_for",
                    "wait_for": {
                        "type": "network_idle",
                        "value": 5000
                    }
                },
                {
                    "action": "scroll",
                    "x": 0,
                    "y": 1000  # Trigger lazy loading
                },
                {
                    "action": "wait_for",
                    "wait_for": {
                        "type": "network_idle",
                        "value": 2000
                    }
                },
                {
                    "action": "scroll",
                    "x": 0,
                    "y": 0  # Back to top
                }
            ],
            "custom_js": """
                // Remove dynamic elements
                document.querySelectorAll('[data-dynamic], .ad, .timestamp')
                    .forEach(el => el.remove());
                
                // Add print-specific styles
                const style = document.createElement('style');
                style.textContent = `
                    @media print {
                        body { font-size: 12pt; }
                        pre, code { white-space: pre-wrap; }
                        a[href]:after { content: " (" attr(href) ")"; }
                    }
                `;
                document.head.appendChild(style);
                
                // Wait for all images
                await Promise.all(
                    Array.from(document.images)
                        .filter(img => !img.complete)
                        .map(img => new Promise(resolve => {
                            img.onload = img.onerror = resolve;
                        }))
                );
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
                # Save archive
                filename = f"{domain}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.pdf"
                filepath = os.path.join(self.output_dir, filename)
                
                with open(filepath, "wb") as f:
                    f.write(response.content)
                
                # Save metadata
                metadata = {
                    "url": url,
                    "timestamp": timestamp,
                    "filename": filename,
                    "full_page": full_page,
                    "size_bytes": len(response.content),
                    "format": "pdf"
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

    async captureScreen(url, name, device = 'desktop') {
        // Use predefined templates for device testing
        const templates = {
            desktop: {
                window_width: 1280,
                window_height: 800,
                pixel_density: 1.0,
                user_agent_device: 'desktop',
                user_agent_platform: 'windows'
            },
            mobile: {
                window_width: 375,
                window_height: 812,
                pixel_density: 2.0,
                user_agent_device: 'mobile',
                user_agent_platform: 'ios'
            }
        };

        const payload = {
            url,
            format: 'png',
            wait_for_network: 'idle',
            wait_for_animation: true,
            use_random_user_agent: true,
            ...templates[device],
            interactions: [
                {
                    action: 'wait_for',
                    wait_for: {
                        type: 'network_idle',
                        value: 3000
                    }
                }
            ],
            custom_js: `
                // Remove dynamic content
                document.querySelectorAll('[data-dynamic], .date, .ad')
                    .forEach(el => el.remove());
                
                // Wait for fonts
                await document.fonts.ready;
                
                // Normalize styles for comparison
                document.body.style.overflow = 'auto';
                document.documentElement.style.overflow = 'auto';
            `
        };

        const response = await axios.post(this.pixashotUrl, payload, {
            headers: {
                'Authorization': `Bearer ${this.apiKey}`,
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer'
        });

        const filename = `${name}_${device}_${Date.now()}.png`;
        await fs.writeFile(filename, response.data);
        return filename;
    }

    // Rest of the class implementation remains the same...
}

// Usage in CI/CD
async function runVisualTests() {
    const tester = new VisualTester(process.env.PIXASHOT_API_KEY);
    
    try {
        // Test both desktop and mobile layouts
        for (const device of ['desktop', 'mobile']) {
            const results = await tester.runTest(
                'https://www.example.com',
                'https://staging.example.com',
                `homepage_${device}`,
                device
            );

            if (results.diffPercentage > 1) {
                console.error(`Visual differences detected on ${device}: ${results.diffPercentage}%`);
                console.error(`Diff file: ${results.diffFile}`);
                process.exit(1);
            }
        }

        console.log('Visual tests passed on all devices');

    } catch (error) {
        console.error('Test failed:', error);
        process.exit(1);
    }
}
```

These examples showcase best practices for using Pixashot, including:
- Structured interaction sequences
- Template usage for common scenarios
- User agent customization for device testing
- Comprehensive wait strategies
- Animation handling
- PDF optimization
- Error handling with retries
- Cleanup routines

Remember that Pixashot uses a single browser context, so:
- Configure extensions via environment variables
- Consider concurrent request limits
- Implement appropriate error handling
- Use caching when possible

For more details on configuration options, see the [API Reference](api-reference.md).