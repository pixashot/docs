---
excerpt: "Comprehensive guide to integrating Pixashot with various programming languages, including detailed examples with error handling, retries, and advanced features."
published_at: true
---

# Integrations

Pixashot provides a RESTful API that can be integrated with any programming language supporting HTTP requests. This guide provides detailed examples for common languages and frameworks.

## Node.js

Using the `axios` library for HTTP requests with retry logic and advanced features:

```javascript
const axios = require('axios');
const fs = require('fs').promises;

class PixashotClient {
    constructor(apiKey, baseUrl = 'https://api.example.com') {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
        this.client = axios.create({
            baseURL: baseUrl,
            headers: {
                'Authorization': `Bearer ${apiKey}`,
                'Content-Type': 'application/json'
            }
        });
    }

    async capture(options, retries = 3) {
        for (let attempt = 1; attempt <= retries; attempt++) {
            try {
                const response = await this.client.post('/capture', {
                    ...options,
                    wait_for_network: options.wait_for_network || 'idle',
                    format: options.format || 'png'
                }, {
                    responseType: 'arraybuffer',
                    timeout: 30000 * attempt // Increase timeout with each retry
                });

                return response.data;

            } catch (error) {
                if (attempt === retries) throw error;
                
                // Handle rate limiting
                if (error.response?.status === 429) {
                    const waitTime = parseInt(error.response.headers['retry-after'] || '5') * 1000;
                    await new Promise(resolve => setTimeout(resolve, waitTime));
                    continue;
                }

                // Exponential backoff for other errors
                await new Promise(resolve => 
                    setTimeout(resolve, Math.pow(2, attempt) * 1000)
                );
            }
        }
    }

    // Convenience methods for common capture scenarios
    async captureFullPage(url, format = 'png') {
        return this.capture({
            url,
            format,
            full_page: true,
            wait_for_network: 'idle',
            window_width: 1920,
            window_height: 1080
        });
    }

    async captureElement(url, selector, format = 'png') {
        return this.capture({
            url,
            format,
            selector,
            wait_for_selector: selector,
            wait_for_network: 'idle'
        });
    }

    async capturePDF(url, options = {}) {
        return this.capture({
            url,
            format: 'pdf',
            full_page: true,
            pdf_format: options.format || 'A4',
            pdf_print_background: options.printBackground ?? true,
            wait_for_network: 'idle'
        });
    }
}

// Usage examples
async function examples() {
    const client = new PixashotClient('your_api_key');

    try {
        // Basic screenshot
        const screenshot = await client.captureFullPage('https://example.com');
        await fs.writeFile('screenshot.png', screenshot);

        // Capture specific element
        const element = await client.captureElement(
            'https://example.com',
            '#main-content'
        );
        await fs.writeFile('element.png', element);

        // Generate PDF
        const pdf = await client.capturePDF('https://example.com', {
            format: 'A4',
            printBackground: true
        });
        await fs.writeFile('document.pdf', pdf);

        // Advanced capture with custom options
        const custom = await client.capture({
            url: 'https://example.com',
            format: 'png',
            wait_for_network: 'idle',
            window_width: 1280,
            window_height: 720,
            pixel_density: 2,
            dark_mode: true,
            custom_js: 'document.body.style.backgroundColor = "white";',
            interactions: [
                {
                    action: 'click',
                    selector: '#accept-cookies'
                },
                {
                    action: 'wait_for',
                    wait_for: {
                        type: 'network_idle',
                        value: 2000
                    }
                }
            ]
        });
        await fs.writeFile('custom.png', custom);

    } catch (error) {
        console.error('Capture failed:', error.message);
        if (error.response) {
            console.error('Status:', error.response.status);
            console.error('Details:', error.response.data);
        }
    }
}
```

## Python

Using `requests` with comprehensive error handling and retry logic:

```python
import requests
from typing import Optional, Dict, Any, Union
import time
import json
from pathlib import Path

class PixashotClient:
    def __init__(self, api_key: str, base_url: str = 'https://api.example.com'):
        self.api_key = api_key
        self.base_url = base_url
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        })

    def capture(
        self,
        options: Dict[str, Any],
        retries: int = 3,
        output_path: Optional[Path] = None
    ) -> bytes:
        """
        Capture a screenshot with retry logic and error handling.
        
        Args:
            options: Screenshot options
            retries: Number of retry attempts
            output_path: Optional path to save the capture
            
        Returns:
            bytes: The captured screenshot data
        """
        for attempt in range(retries):
            try:
                response = self.session.post(
                    f'{self.base_url}/capture',
                    json=options,
                    timeout=30 * (attempt + 1)  # Increase timeout with each retry
                )
                
                response.raise_for_status()
                
                if output_path:
                    output_path.write_bytes(response.content)
                
                return response.content
                
            except requests.exceptions.RequestException as e:
                if attempt == retries - 1:
                    raise

                # Handle rate limiting
                if getattr(e.response, 'status_code', None) == 429:
                    retry_after = int(e.response.headers.get('retry-after', '5'))
                    time.sleep(retry_after)
                    continue

                # Exponential backoff for other errors
                time.sleep(2 ** attempt)

    def capture_full_page(
        self,
        url: str,
        format: str = 'png',
        output_path: Optional[Path] = None
    ) -> bytes:
        """Convenience method for full-page captures."""
        return self.capture({
            'url': url,
            'format': format,
            'full_page': True,
            'wait_for_network': 'idle',
            'window_width': 1920,
            'window_height': 1080
        }, output_path=output_path)

    def capture_element(
        self,
        url: str,
        selector: str,
        format: str = 'png',
        output_path: Optional[Path] = None
    ) -> bytes:
        """Convenience method for element captures."""
        return self.capture({
            'url': url,
            'format': format,
            'selector': selector,
            'wait_for_selector': selector,
            'wait_for_network': 'idle'
        }, output_path=output_path)

    def capture_pdf(
        self,
        url: str,
        options: Dict[str, Any] = None,
        output_path: Optional[Path] = None
    ) -> bytes:
        """Convenience method for PDF generation."""
        pdf_options = {
            'url': url,
            'format': 'pdf',
            'full_page': True,
            'pdf_format': 'A4',
            'pdf_print_background': True,
            'wait_for_network': 'idle'
        }
        if options:
            pdf_options.update(options)
        
        return self.capture(pdf_options, output_path=output_path)

# Usage examples
def examples():
    client = PixashotClient('your_api_key')
    
    try:
        # Basic screenshot
        client.capture_full_page(
            'https://example.com',
            output_path=Path('screenshot.png')
        )

        # Capture element
        client.capture_element(
            'https://example.com',
            '#main-content',
            output_path=Path('element.png')
        )

        # Generate PDF
        client.capture_pdf(
            'https://example.com',
            options={'pdf_format': 'A4', 'pdf_print_background': True},
            output_path=Path('document.pdf')
        )

        # Advanced capture
        client.capture({
            'url': 'https://example.com',
            'format': 'png',
            'wait_for_network': 'idle',
            'window_width': 1280,
            'window_height': 720,
            'pixel_density': 2,
            'dark_mode': True,
            'custom_js': 'document.body.style.backgroundColor = "white";',
            'interactions': [
                {
                    'action': 'click',
                    'selector': '#accept-cookies'
                },
                {
                    'action': 'wait_for',
                    'wait_for': {
                        'type': 'network_idle',
                        'value': 2000
                    }
                }
            ]
        }, output_path=Path('custom.png'))

    except requests.exceptions.RequestException as e:
        print(f'Capture failed: {str(e)}')
        if e.response is not None:
            print(f'Status: {e.response.status_code}')
            try:
                print(f'Details: {e.response.json()}')
            except json.JSONDecodeError:
                print(f'Details: {e.response.text}')
```

## Java

Modern Java implementation using HttpClient with comprehensive error handling:

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Path;
import java.time.Duration;
import java.util.Map;
import java.util.concurrent.CompletableFuture;

import com.fasterxml.jackson.databind.ObjectMapper;

public class PixashotClient {
    private final HttpClient client;
    private final String apiKey;
    private final String baseUrl;
    private final ObjectMapper objectMapper;

    public PixashotClient(String apiKey, String baseUrl) {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
        this.objectMapper = new ObjectMapper();
        this.client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(30))
            .build();
    }

    public CompletableFuture<byte[]> capture(Map<String, Object> options) {
        String jsonBody;
        try {
            jsonBody = objectMapper.writeValueAsString(options);
        } catch (Exception e) {
            return CompletableFuture.failedFuture(e);
        }

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/capture"))
            .header("Authorization", "Bearer " + apiKey)
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
            .timeout(Duration.ofSeconds(30))
            .build();

        return client.sendAsync(request, HttpResponse.BodyHandlers.ofByteArray())
            .thenApply(response -> {
                if (response.statusCode() == 200) {
                    return response.body();
                }
                throw new RuntimeException("Capture failed: " + response.statusCode());
            });
    }

    public CompletableFuture<byte[]> captureFullPage(String url, String format) {
        return capture(Map.of(
            "url", url,
            "format", format,
            "full_page", true,
            "wait_for_network", "idle",
            "window_width", 1920,
            "window_height", 1080
        ));
    }

    public CompletableFuture<byte[]> captureElement(String url, String selector) {
        return capture(Map.of(
            "url", url,
            "format", "png",
            "selector", selector,
            "wait_for_selector", selector,
            "wait_for_network", "idle"
        ));
    }

    public CompletableFuture<byte[]> capturePDF(String url, String format) {
        return capture(Map.of(
            "url", url,
            "format", "pdf",
            "pdf_format", format,
            "pdf_print_background", true,
            "full_page", true,
            "wait_for_network", "idle"
        ));
    }

    // Example usage
    public static void main(String[] args) {
        PixashotClient client = new PixashotClient("your_api_key", "https://api.example.com");

        client.captureFullPage("https://example.com", "png")
            .thenAccept(bytes -> {
                try {
                    Files.write(Path.of("screenshot.png"), bytes);
                    System.out.println("Screenshot saved");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            })
            .exceptionally(throwable -> {
                System.err.println("Capture failed: " + throwable.getMessage());
                return null;
            });
    }
}
```

For examples in other languages (C#, PHP, Go, Ruby) and more advanced usage scenarios, please visit our [API Reference](api-reference.md) documentation.

Each client implementation includes:
- Robust error handling
- Retry logic with exponential backoff
- Rate limit handling
- Timeout management
- Convenience methods for common operations
- Support for all Pixashot features
- Type safety (where applicable)
- Proper resource cleanup

Remember to:
1. Replace `'your_api_key'` with your actual Pixashot API key
2. Update the base URL to match your Pixashot instance
3. Implement proper error handling in your application
4. Consider implementing caching for frequently accessed pages
5. Monitor rate limits and adjust retry strategies accordingly

For detailed API documentation and additional features, refer to the [API Reference](api-reference.md).