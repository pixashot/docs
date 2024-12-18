---
title: Node.js Integration
excerpt: Example code for using Pixashot with Node.js using fetch or axios
meta:
  nav_order: 81
---

# Node.js Integration

## Using Fetch API

```javascript
class PixashotClient {
    constructor(apiKey, baseUrl = 'https://api.example.com') {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
    }

    async capture(options, retries = 3) {
        for (let attempt = 1; attempt <= retries; attempt++) {
            try {
                const response = await fetch(`${this.baseUrl}/capture`, {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${this.apiKey}`,
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(options)
                });

                if (response.status === 429) {
                    const retryAfter = parseInt(response.headers.get('Retry-After') || '5');
                    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
                    continue;
                }

                if (!response.ok) throw new Error(`HTTP ${response.status}`);

                return await response.arrayBuffer();

            } catch (error) {
                if (attempt === retries) throw error;
                await new Promise(r => setTimeout(r, Math.pow(2, attempt) * 1000));
            }
        }
    }

    async saveScreenshot(url, outputPath) {
        const buffer = await this.capture({
            url,
            format: 'png',
            full_page: true,
            wait_for_network: 'idle'
        });

        await fs.promises.writeFile(outputPath, Buffer.from(buffer));
    }
}

// Usage example
const client = new PixashotClient('your_token_here');

try {
    await client.saveScreenshot(
        'https://example.com',
        'screenshot.png'
    );
    console.log('Screenshot saved');
} catch (error) {
    console.error('Screenshot failed:', error.message);
}
```

## Using Axios

```javascript
const axios = require('axios');
const fs = require('fs');

class PixashotClient {
    constructor(apiKey, baseUrl = 'https://api.example.com') {
        this.client = axios.create({
            baseURL: baseUrl,
            headers: {
                'Authorization': `Bearer ${apiKey}`,
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer'
        });
    }

    async capture(options) {
        try {
            const response = await this.client.post('/capture', options);
            return response.data;
        } catch (error) {
            if (error.response?.status === 429) {
                const retryAfter = parseInt(error.response.headers['retry-after'] || '5');
                await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
                return this.capture(options);
            }
            throw error;
        }
    }

    // Convenience methods
    async capturePDF(url) {
        return this.capture({
            url,
            format: 'pdf',
            pdf_format: 'A4',
            pdf_print_background: true,
            full_page: true
        });
    }

    async captureElement(url, selector) {
        return this.capture({
            url,
            selector,
            format: 'png',
            wait_for_selector: selector
        });
    }
}

// Usage examples
const client = new PixashotClient('your_token_here');

// Capture PDF
const pdfBuffer = await client.capturePDF('https://example.com');
await fs.promises.writeFile('output.pdf', pdfBuffer);

// Capture specific element
const elementBuffer = await client.captureElement(
    'https://example.com',
    '#main-content'
);
await fs.promises.writeFile('element.png', elementBuffer);
```

## TypeScript Support

```typescript
interface CaptureOptions {
    url?: string;
    html_content?: string;
    format?: 'png' | 'jpeg' | 'webp' | 'pdf';
    full_page?: boolean;
    window_width?: number;
    window_height?: number;
    wait_for_network?: 'idle' | 'mostly_idle';
    selector?: string;
    custom_js?: string;
    dark_mode?: boolean;
}

class PixashotClient {
    constructor(
        private readonly apiKey: string,
        private readonly baseUrl: string = 'https://api.example.com'
    ) {}

    async capture(options: CaptureOptions): Promise<Buffer> {
        // Implementation as above
    }
}
```

## Error Handling

All examples include comprehensive error handling:
- Network errors
- Rate limiting
- Validation errors
- Timeout handling
- Resource cleanup

## Best Practices

1. **Error Handling**
    - Implement retry logic
    - Handle rate limits gracefully
    - Set appropriate timeouts
    - Clean up temporary files

2. **Performance**
    - Reuse client instances
    - Enable keep-alive
    - Stream large responses
    - Handle memory efficiently

3. **Security**
    - Store API keys securely
    - Validate input URLs
    - Use HTTPS
    - Handle secrets properly