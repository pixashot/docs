---
title: Web Page Archiving Guide
excerpt: Complete guide for implementing web page archiving with Pixashot, including content preservation, document generation, and metadata handling.
meta:
  nav_order: 133
---

# Web Page Archiving

## Basic Implementation

### Page Archiver
```python
from datetime import datetime
import json

class WebArchiver:
    def __init__(self, storage_path: str):
        self.client = Client('your_api_key')
        self.storage_path = storage_path
    
    async def archive_page(self, url: str) -> dict:
        timestamp = datetime.utcnow().isoformat()
        
        # Capture visual snapshot
        screenshot = await self.client.capture({
            'url': url,
            'format': 'png',
            'full_page': True,
            'wait_for_network': 'idle',
            'custom_js': '''
                // Remove dynamic elements
                document.querySelectorAll('[data-dynamic], .timestamp').forEach(el => el.remove());
            '''
        })
        
        # Generate PDF version
        pdf = await self.client.capture({
            'url': url,
            'format': 'pdf',
            'pdf_format': 'A4',
            'full_page': True,
            'pdf_print_background': True
        })
        
        # Capture HTML content
        html = await self.client.capture({
            'url': url,
            'format': 'html',
            'wait_for_network': 'idle'
        })
        
        return {
            'timestamp': timestamp,
            'url': url,
            'formats': {
                'png': screenshot,
                'pdf': pdf,
                'html': html
            }
        }
```

## Metadata Management

```python
class ArchiveMetadata:
    def __init__(self, archive_id: str):
        self.archive_id = archive_id
        self.metadata = {
            'id': archive_id,
            'created_at': datetime.utcnow().isoformat(),
            'captures': []
        }
    
    def add_capture(self, url: str, artifacts: dict) -> None:
        self.metadata['captures'].append({
            'url': url,
            'timestamp': datetime.utcnow().isoformat(),
            'artifacts': {
                format: {
                    'path': path,
                    'size': len(content)
                }
                for format, content in artifacts.items()
            }
        })
    
    def save(self) -> None:
        with open(f"metadata/{self.archive_id}.json", 'w') as f:
            json.dump(self.metadata, f, indent=2)
```

## Legal Compliance

### Authenticated Archiving
```python
async def archive_with_auth(url: str, credentials: dict) -> dict:
    client = Client('your_api_key')
    
    # Capture with authentication
    return await client.capture({
        'url': url,
        'format': 'pdf',
        'custom_headers': {
            'Authorization': f"Bearer {credentials['token']}"
        },
        'wait_for_network': 'idle',
        'pdf_format': 'A4',
        'pdf_print_background': True,
        'custom_js': '''
            // Add timestamp watermark
            const watermark = document.createElement('div');
            watermark.textContent = new Date().toISOString();
            watermark.style.position = 'fixed';
            watermark.style.bottom = '10px';
            watermark.style.right = '10px';
            watermark.style.opacity = '0.5';
            document.body.appendChild(watermark);
        '''
    })
```

## Best Practices

1. **Content Preservation**
    - Capture multiple formats
    - Store complete metadata
    - Include timestamps
    - Remove dynamic elements

2. **Storage Management**
    - Implement versioning
    - Use efficient storage
    - Maintain backups
    - Monitor storage usage

3. **Legal Compliance**
    - Add timestamps/watermarks
    - Maintain audit logs
    - Ensure data integrity
    - Document all processes

For additional information:
- [PDF Options](../capture-options/output-formats.md)
- [Authentication](../security/authentication.md)
- [Storage Integration](../deployment/index.md)