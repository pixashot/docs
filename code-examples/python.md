---
title: Python Integration
excerpt: Example code for using Pixashot with Python using requests or aiohttp
meta:
    nav_order: 82
---

# Python Integration

## Using Requests

```python
import requests
from pathlib import Path
from typing import Optional, Dict, Any, Union
import time
import logging

logger = logging.getLogger(__name__)

class PixashotClient:
    def __init__(self, api_key: str, base_url: str = "https://api.example.com"):
        self.session = requests.Session()
        self.session.headers.update({
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        })
        self.base_url = base_url.rstrip('/')

    def capture(
        self,
        options: Dict[str, Any],
        output_path: Optional[Path] = None,
        retries: int = 3
    ) -> bytes:
        """
        Capture screenshot with retry logic and error handling.
        
        Args:
            options: Screenshot options
            output_path: Optional path to save the capture
            retries: Number of retry attempts
        """
        for attempt in range(retries):
            try:
                response = self.session.post(
                    f"{self.base_url}/capture",
                    json=options,
                    timeout=30 * (attempt + 1)  # Increase timeout with each retry
                )
                
                if response.status_code == 429:
                    retry_after = int(response.headers.get('retry-after', '5'))
                    time.sleep(retry_after)
                    continue
                
                response.raise_for_status()
                
                if output_path:
                    output_path.write_bytes(response.content)
                
                return response.content
                
            except requests.RequestException as e:
                if attempt == retries - 1:
                    raise
                time.sleep(2 ** attempt)  # Exponential backoff

    def capture_full_page(
        self,
        url: str,
        format: str = "png",
        output_path: Optional[Path] = None
    ) -> bytes:
        """Capture a full page screenshot."""
        return self.capture({
            "url": url,
            "format": format,
            "full_page": True,
            "wait_for_network": "idle"
        }, output_path)

    def capture_element(
        self,
        url: str,
        selector: str,
        format: str = "png",
        output_path: Optional[Path] = None
    ) -> bytes:
        """Capture a specific element."""
        return self.capture({
            "url": url,
            "format": format,
            "selector": selector,
            "wait_for_selector": selector
        }, output_path)

    def capture_pdf(
        self,
        url: str,
        output_path: Optional[Path] = None
    ) -> bytes:
        """Generate a PDF of the page."""
        return self.capture({
            "url": url,
            "format": "pdf",
            "full_page": True,
            "pdf_format": "A4",
            "pdf_print_background": True
        }, output_path)

# Usage examples
client = PixashotClient("your_token_here")

try:
    # Full page screenshot
    client.capture_full_page(
        "https://example.com",
        output_path=Path("screenshot.png")
    )

    # Capture specific element
    client.capture_element(
        "https://example.com",
        "#main-content",
        output_path=Path("element.png")
    )

    # Generate PDF
    client.capture_pdf(
        "https://example.com",
        output_path=Path("output.pdf")
    )

except requests.RequestException as e:
    logger.error(f"Capture failed: {e}")
```

## Using aiohttp (Async)

```python
import aiohttp
import asyncio
from pathlib import Path
from typing import Optional, Dict, Any

class AsyncPixashotClient:
    def __init__(self, api_key: str, base_url: str = "https://api.example.com"):
        self.api_key = api_key
        self.base_url = base_url.rstrip('/')
        self.session = None

    async def __aenter__(self):
        self.session = aiohttp.ClientSession(headers={
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        })
        return self

    async def __aexit__(self, exc_type, exc, tb):
        if self.session:
            await self.session.close()

    async def capture(
        self,
        options: Dict[str, Any],
        output_path: Optional[Path] = None
    ) -> bytes:
        if not self.session:
            raise RuntimeError("Client not initialized. Use async with.")

        async with self.session.post(
            f"{self.base_url}/capture",
            json=options
        ) as response:
            if response.status == 429:
                retry_after = int(response.headers.get('retry-after', '5'))
                await asyncio.sleep(retry_after)
                return await self.capture(options, output_path)

            response.raise_for_status()
            data = await response.read()

            if output_path:
                async with aiofiles.open(output_path, 'wb') as f:
                    await f.write(data)

            return data

# Usage example with async
async def main():
    async with AsyncPixashotClient("your_token_here") as client:
        # Capture multiple screenshots concurrently
        tasks = [
            client.capture({
                "url": f"https://example{i}.com",
                "format": "png"
            }, Path(f"screenshot{i}.png"))
            for i in range(3)
        ]
        await asyncio.gather(*tasks)

if __name__ == "__main__":
    asyncio.run(main())
```

## Type Annotations

```python
from dataclasses import dataclass
from typing import Literal, Optional

@dataclass
class CaptureOptions:
    url: Optional[str] = None
    html_content: Optional[str] = None
    format: Literal["png", "jpeg", "webp", "pdf"] = "png"
    full_page: bool = False
    window_width: int = 1920
    window_height: int = 1080
    wait_for_network: Literal["idle", "mostly_idle"] = "idle"
    selector: Optional[str] = None
    dark_mode: bool = False

# Usage with type hints
client = PixashotClient("your_token_here")
options = CaptureOptions(
    url="https://example.com",
    format="png",
    full_page=True
)
client.capture(options.__dict__)
```

## Error Handling

The client includes handling for:
- Network errors
- Rate limiting
- Validation errors
- Timeouts
- File I/O errors

## Best Practices

1. **Resource Management**
    - Use session reuse
    - Implement proper cleanup
    - Handle file operations safely
    - Monitor memory usage

2. **Error Handling**
    - Implement retry logic
    - Handle rate limits
    - Log failures appropriately
    - Clean up temporary files

3. **Performance**
    - Use async for concurrent captures
    - Enable keepalive
    - Stream large responses
    - Manage memory efficiently