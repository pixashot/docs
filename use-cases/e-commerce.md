---
title: E-commerce Implementation Guide
excerpt: Detailed guide for implementing Pixashot in e-commerce applications, including product image capture, catalog generation, and social media integration.
meta:
    nav_order: 71
---

# E-commerce Implementation

## Product Image Capture

### Basic Product Screenshot
```python
import asyncio
from pixashot import Client

async def capture_product(product_url: str, product_id: str) -> str:
    client = Client('your_api_key')
    
    response = await client.capture({
        'url': product_url,
        'format': 'png',
        'selector': '#main-product-image',
        'window_width': 1200,
        'window_height': 1200,
        'pixel_density': 2.0,
        'wait_for_network': 'idle',
        'custom_js': '''
            // Remove distracting elements
            document.querySelectorAll('.popup, .chat-widget, .newsletter').forEach(el => el.remove());
            
            // Ensure product image is loaded
            await new Promise(resolve => {
                const img = document.querySelector('#main-product-image img');
                if (img.complete) resolve();
                else img.onload = resolve;
            });
        '''
    })
    
    return f"product_{product_id}.png"
```

## Catalog Generation

### Batch Product Processing
```python
async def generate_catalog(products: list) -> None:
    client = Client('your_api_key')
    
    async def process_product(product):
        try:
            screenshot = await client.capture({
                'url': product['url'],
                'format': 'png',
                'selector': '.product-container',
                'wait_for_network': 'idle',
                'block_media': False,
                'custom_js': '''
                    // Ensure consistent styling
                    document.body.style.backgroundColor = 'white';
                    document.querySelector('.product-container').style.padding = '20px';
                '''
            })
            
            await save_to_storage(
                screenshot, 
                f"catalog/{product['id']}.png"
            )
            
        except Exception as e:
            logger.error(f"Failed to process {product['id']}: {str(e)}")
    
    # Process in batches of 5
    batch_size = 5
    for i in range(0, len(products), batch_size):
        batch = products[i:i + batch_size]
        await asyncio.gather(*[
            process_product(product) 
            for product in batch
        ])
```

## Social Media Integration

### Product Card Generation
```python
async def create_social_card(product: dict) -> bytes:
    client = Client('your_api_key')
    
    return await client.capture({
        'url': 'https://your-template-url/product-card',
        'format': 'png',
        'window_width': 1200,
        'window_height': 630,
        'pixel_density': 2.0,
        'wait_for_network': 'idle',
        'custom_js': f'''
            document.getElementById('product-name').textContent = "{product['name']}";
            document.getElementById('product-price').textContent = "${product['price']}";
            document.getElementById('product-image').src = "{product['image_url']}";
        '''
    })
```

## Performance Optimization

### Image Processing Pipeline
```python
async def process_product_images(product_id: str):
    # Primary product image
    main_image = await capture_product_main(product_id)
    
    # Generate variants in parallel
    variants = await asyncio.gather(
        create_thumbnail(main_image),
        create_social_preview(main_image),
        create_zoom_view(main_image)
    )
    
    return {
        'main': main_image,
        'thumbnail': variants[0],
        'social': variants[1],
        'zoom': variants[2]
    }
```

## Error Handling

```python
class ProductImageProcessor:
    def __init__(self, client: Client, retries: int = 3):
        self.client = client
        self.retries = retries
    
    async def capture_with_retry(self, options: dict) -> bytes:
        for attempt in range(self.retries):
            try:
                return await self.client.capture(options)
            except Exception as e:
                if attempt == self.retries - 1:
                    raise
                await asyncio.sleep(2 ** attempt)  # Exponential backoff
```

## Integration Example

### Shopify Integration
```python
from shopify import Product
from pixashot import Client

class ShopifyImageProcessor:
    def __init__(self, shop_url: str, api_key: str):
        self.shop_url = shop_url
        self.client = Client(api_key)
    
    async def process_new_product(self, product: Product):
        # Capture main product image
        main_image = await self.client.capture({
            'url': product.url,
            'selector': '#product-main-image',
            'format': 'png',
            'wait_for_network': 'idle'
        })
        
        # Create social media variants
        social_image = await self.create_social_preview(product)
        
        # Update Shopify product
        await self.update_product_images(
            product.id,
            main_image,
            social_image
        )
```

## Best Practices

1. **Resource Management**
    - Implement proper retry logic
    - Use batch processing for multiple products
    - Cache results when possible
    - Monitor rate limits

2. **Image Quality**
    - Use appropriate pixel density
    - Set viewport sizes based on use case
    - Test across different devices
    - Validate output quality

3. **Performance**
    - Process images in parallel
    - Implement efficient error handling
    - Use appropriate timeouts
    - Monitor system resources

For more information on specific features, refer to:
- [Capture Options](../capture-options/index.md)
- [Resource Management](../core-concepts/resource-management.md)
- [API Reference](../api-reference/index.md)