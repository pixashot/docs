---
excerpt: "Guide to integrating Pixashot with various programming languages through its HTTP API, including examples for Node.js, Python, Java, C#, PHP, Go, and Ruby."
published_at: true
---

# Integrations

Pixashot can be easily integrated with various programming languages through its HTTP API. Below are examples of how to use Pixashot in different languages.

## Node.js

Using the `axios` library for HTTP requests:

```javascript
const axios = require('axios');
const fs = require('fs');

async function captureScreenshot() {
  try {
    const response = await axios.post('https://api.example.com/capture', {
      url: 'https://example.com',
      format: 'png',
      full_page: true
    }, {
      headers: {
        'Authorization': 'Bearer YOUR_API_KEY',
        'Content-Type': 'application/json'
      },
      responseType: 'arraybuffer'
    });

    fs.writeFileSync('screenshot.png', response.data);
    console.log('Screenshot saved as screenshot.png');
  } catch (error) {
    console.error('Error capturing screenshot:', error.message);
  }
}

captureScreenshot();
```

## Python

Using the `requests` library:

```python
import requests

def capture_screenshot():
    url = 'https://api.example.com/capture'
    payload = {
        'url': 'https://example.com',
        'format': 'png',
        'full_page': True
    }
    headers = {
        'Authorization': 'Bearer YOUR_API_KEY',
        'Content-Type': 'application/json'
    }

    try:
        response = requests.post(url, json=payload, headers=headers)
        if response.status_code == 200:
            with open('screenshot.png', 'wb') as f:
                f.write(response.content)
            print('Screenshot saved as screenshot.png')
        else:
            print(f'Error: {response.status_code} - {response.text}')
    except Exception as e:
        print(f'Error capturing screenshot: {str(e)}')

capture_screenshot()
```

## Java

Using `HttpClient` from Java 11+:

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Paths;

public class PixashotClient {
    public static void main(String[] args) {
        try {
            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.example.com/capture"))
                .header("Authorization", "Bearer YOUR_API_KEY")
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(
                    "{\"url\":\"https://example.com\",\"format\":\"png\",\"full_page\":true}"
                ))
                .build();

            HttpResponse<byte[]> response = client.send(request, HttpResponse.BodyHandlers.ofByteArray());

            if (response.statusCode() == 200) {
                Files.write(Paths.get("screenshot.png"), response.body());
                System.out.println("Screenshot saved as screenshot.png");
            } else {
                System.out.println("Error: " + response.statusCode());
            }
        } catch (Exception e) {
            System.out.println("Error capturing screenshot: " + e.getMessage());
        }
    }
}
```

## C#

Using `HttpClient`:

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using System.IO;

class PixashotClient
{
    static async Task Main(string[] args)
    {
        using (var client = new HttpClient())
        {
            client.DefaultRequestHeaders.Add("Authorization", "Bearer YOUR_API_KEY");

            var content = new StringContent(
                "{\"url\":\"https://example.com\",\"format\":\"png\",\"full_page\":true}",
                Encoding.UTF8, 
                "application/json"
            );

            try
            {
                var response = await client.PostAsync("https://api.example.com/capture", content);
                if (response.IsSuccessStatusCode)
                {
                    var imageBytes = await response.Content.ReadAsByteArrayAsync();
                    await File.WriteAllBytesAsync("screenshot.png", imageBytes);
                    Console.WriteLine("Screenshot saved as screenshot.png");
                }
                else
                {
                    Console.WriteLine($"Error: {response.StatusCode}");
                }
            }
            catch (Exception e)
            {
                Console.WriteLine($"Error capturing screenshot: {e.Message}");
            }
        }
    }
}
```

## PHP

Using `cURL`:

```php
<?php

$url = 'https://api.example.com/capture';
$data = json_encode([
    'url' => 'https://example.com',
    'format' => 'png',
    'full_page' => true
]);

$ch = curl_init($url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Authorization: Bearer YOUR_API_KEY',
    'Content-Type: application/json'
]);

$response = curl_exec($ch);

if (curl_errno($ch)) {
    echo 'Error capturing screenshot: ' . curl_error($ch);
} else {
    $httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    if ($httpcode == 200) {
        file_put_contents('screenshot.png', $response);
        echo 'Screenshot saved as screenshot.png';
    } else {
        echo 'Error: ' . $httpcode;
    }
}

curl_close($ch);
```

## Go

Using the `net/http` package:

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {
    url := "https://api.example.com/capture"
    data := map[string]interface{}{
        "url":       "https://example.com",
        "format":    "png",
        "full_page": true,
    }
    jsonData, _ := json.Marshal(data)

    req, _ := http.NewRequest("POST", url, bytes.NewBuffer(jsonData))
    req.Header.Set("Authorization", "Bearer YOUR_API_KEY")
    req.Header.Set("Content-Type", "application/json")

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println("Error capturing screenshot:", err)
        return
    }
    defer resp.Body.Close()

    if resp.StatusCode == 200 {
        body, _ := ioutil.ReadAll(resp.Body)
        err = ioutil.WriteFile("screenshot.png", body, 0644)
        if err != nil {
            fmt.Println("Error saving screenshot:", err)
        } else {
            fmt.Println("Screenshot saved as screenshot.png")
        }
    } else {
        fmt.Println("Error:", resp.Status)
    }
}
```

## Ruby

Using the `net/http` library:

```ruby
require 'net/http'
require 'json'

uri = URI('https://api.example.com/capture')
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

request = Net::HTTP::Post.new(uri)
request['Authorization'] = 'Bearer YOUR_API_KEY'
request['Content-Type'] = 'application/json'
request.body = {
  url: 'https://example.com',
  format: 'png',
  full_page: true
}.to_json

response = http.request(request)

if response.code == '200'
  File.open('screenshot.png', 'wb') do |file|
    file.write(response.body)
  end
  puts 'Screenshot saved as screenshot.png'
else
  puts "Error: #{response.code} - #{response.message}"
end
```

These examples demonstrate basic usage of the Pixashot API in various programming languages. Remember to replace `'YOUR_API_KEY'` with your actual Pixashot API key in each example. For more advanced usage and error handling, refer to the API documentation and the specific documentation for the HTTP libraries used in each language.