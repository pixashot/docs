---
title: PHP Integration
excerpt: Example code for using Pixashot with PHP using Guzzle HTTP client
meta:
  nav_order: 83
---

# PHP Integration

## Using Guzzle

```php
<?php

use GuzzleHttp\Client;
use GuzzleHttp\Exception\RequestException;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware;
use GuzzleHttp\Psr7\Response;

class PixashotClient
{
    private Client $client;
    private string $baseUrl;

    public function __construct(
        string $apiKey,
        string $baseUrl = 'https://api.example.com',
        array $config = []
    ) {
        // Create handler stack with retry middleware
        $stack = HandlerStack::create();
        $stack->push($this->retryMiddleware());

        $this->baseUrl = rtrim($baseUrl, '/');
        $this->client = new Client(array_merge([
            'base_uri' => $this->baseUrl,
            'handler' => $stack,
            'headers' => [
                'Authorization' => "Bearer {$apiKey}",
                'Content-Type' => 'application/json',
            ],
        ], $config));
    }

    private function retryMiddleware()
    {
        return Middleware::retry(
            function (
                $retries,
                $request,
                ?Response $response = null,
                ?RequestException $exception = null
            ) {
                // Retry on rate limits or server errors
                if ($response) {
                    if ($response->getStatusCode() === 429) {
                        $delay = (int)($response->getHeader('Retry-After')[0] ?? 5);
                        sleep($delay);
                        return true;
                    }
                }

                // Maximum 3 retries
                return $retries < 3;
            },
            function ($retries) {
                // Exponential backoff
                return 1000 * pow(2, $retries);
            }
        );
    }

    public function capture(array $options, ?string $outputPath = null): string
    {
        try {
            $response = $this->client->post('/capture', [
                'json' => $options,
            ]);

            $content = $response->getBody()->getContents();

            if ($outputPath !== null) {
                file_put_contents($outputPath, $content);
            }

            return $content;
        } catch (RequestException $e) {
            throw new Exception(
                "Screenshot capture failed: " . $e->getMessage(),
                $e->getCode(),
                $e
            );
        }
    }

    public function captureFullPage(
        string $url,
        string $format = 'png',
        ?string $outputPath = null
    ): string {
        return $this->capture([
            'url' => $url,
            'format' => $format,
            'full_page' => true,
            'wait_for_network' => 'idle',
        ], $outputPath);
    }

    public function captureElement(
        string $url,
        string $selector,
        string $format = 'png',
        ?string $outputPath = null
    ): string {
        return $this->capture([
            'url' => $url,
            'format' => $format,
            'selector' => $selector,
            'wait_for_selector' => $selector,
        ], $outputPath);
    }

    public function capturePDF(
        string $url,
        ?string $outputPath = null
    ): string {
        return $this->capture([
            'url' => $url,
            'format' => 'pdf',
            'pdf_format' => 'A4',
            'pdf_print_background' => true,
            'full_page' => true,
        ], $outputPath);
    }
}

// Usage examples
try {
    $client = new PixashotClient('your_token_here');

    // Full page screenshot
    $client->captureFullPage(
        'https://example.com',
        'png',
        'screenshot.png'
    );

    // Capture specific element
    $client->captureElement(
        'https://example.com',
        '#main-content',
        'png',
        'element.png'
    );

    // Generate PDF
    $client->capturePDF(
        'https://example.com',
        'output.pdf'
    );

} catch (Exception $e) {
    error_log("Screenshot capture failed: " . $e->getMessage());
}
```

## Type Definitions

```php
<?php

class CaptureOptions
{
    public function __construct(
        public ?string $url = null,
        public ?string $html_content = null,
        public string $format = 'png',
        public bool $full_page = false,
        public int $window_width = 1920,
        public int $window_height = 1080,
        public string $wait_for_network = 'idle',
        public ?string $selector = null,
        public bool $dark_mode = false
    ) {}

    public function toArray(): array
    {
        return array_filter(get_object_vars($this), fn($value) => $value !== null);
    }
}

// Usage with types
$options = new CaptureOptions(
    url: 'https://example.com',
    format: 'png',
    full_page: true
);
$client->capture($options->toArray());
```

## Error Handling

The client handles:
- Network timeouts
- Rate limiting
- Server errors
- Authentication errors
- File system errors

## Best Practices

1. **Resource Management**
    - Use persistent HTTP connections
    - Clean up temporary files
    - Monitor memory usage
    - Handle timeouts properly

2. **Error Handling**
    - Implement retry logic
    - Handle rate limits
    - Log errors appropriately
    - Use proper exception classes

3. **Security**
    - Store API keys securely
    - Validate input URLs
    - Use HTTPS
    - Sanitize file paths

4. **Performance**
    - Reuse client instances
    - Enable keep-alive
    - Stream large responses
    - Use appropriate timeouts