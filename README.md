# Wappalyzergo

A high performance port of the Wappalyzer Technology Detection Library to Go. Inspired by [Webanalyze](https://github.com/rverton/webanalyze).

Uses data from https://github.com/AliasIO/wappalyzer

## Features

- Very simple and easy to use, with clean codebase.
- Normalized regexes + auto-updating database of wappalyzer fingerprints.
- Optimized for performance: parsing HTML manually for best speed.
- Built-in caching system for improved performance
- Performance metrics and monitoring
- Input validation and sanitization
- CLI tool for command-line usage
- Web server with REST API and web interface
- Comprehensive benchmarking suite

### Using *go install*

```sh
go install -v github.com/projectdiscovery/wappalyzergo/cmd/update-fingerprints@latest
go install -v github.com/projectdiscovery/wappalyzergo/cmd/wappalyzer-cli@latest
go install -v github.com/projectdiscovery/wappalyzergo/cmd/wappalyzer-server@latest
```

After this command *wappalyzergo* library source will be in your current go.mod.

## CLI Usage

The CLI tool provides a simple way to analyze websites from the command line:

```sh
# Basic analysis
wappalyzer-cli -url https://example.com

# With detailed information
wappalyzer-cli -url https://example.com -info -output table

# Include categories
wappalyzer-cli -url https://example.com -categories -output json

# Custom user agent and timeout
wappalyzer-cli -url https://example.com -user-agent "Custom Bot 1.0" -timeout 30s
```

## Web Server

Start the web server for a user-friendly interface and REST API:

```sh
# Start server on default port 8080
wappalyzer-server

# Custom port and timeout
wappalyzer-server -port 3000 -timeout 15s
```

Then visit `http://localhost:8080` for the web interface or use the REST API:

```sh
curl -X POST http://localhost:8080/api/analyze \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "with_info": true}'
```

## Example
Usage Example:

``` go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"

	wappalyzer "github.com/projectdiscovery/wappalyzergo"
)

func main() {
	resp, err := http.DefaultClient.Get("https://www.hackerone.com")
	if err != nil {
		log.Fatal(err)
	}
	data, _ := io.ReadAll(resp.Body) // Ignoring error for example

	wappalyzerClient, err := wappalyzer.New()
	fingerprints := wappalyzerClient.Fingerprint(resp.Header, data)
	fmt.Printf("%v\n", fingerprints)

	// Output: map[Acquia Cloud Platform:{} Amazon EC2:{} Apache:{} Cloudflare:{} Drupal:{} PHP:{} Percona:{} React:{} Varnish:{}]
}
```

## Performance Features

### Caching

Enable caching for improved performance on repeated requests:

```go
wappalyzerClient, err := wappalyzer.New()
if err != nil {
    log.Fatal(err)
}

// Create a cache with max 1000 entries, 5 minute TTL
cache := wappalyzer.NewFingerprintCache(1000, 5*time.Minute)

// Check cache first
if result, found := cache.Get(headers, body); found {
    fmt.Printf("Cached result: %v\n", result)
} else {
    result := wappalyzerClient.Fingerprint(headers, body)
    cache.Set(headers, body, result)
    fmt.Printf("Fresh result: %v\n", result)
}
```

### Performance Metrics

Track performance metrics:

```go
wappalyzerClient, _ := wappalyzer.New()

// Perform some fingerprinting...
wappalyzerClient.Fingerprint(headers, body)

// Get metrics
metrics := wappalyzerClient.GetMetrics()
fmt.Printf("Total requests: %d\n", metrics.TotalRequests)
fmt.Printf("Average duration: %v\n", metrics.AverageDuration)
fmt.Printf("Technologies found: %d\n", metrics.TechnologiesFound)
```

### Input Validation

Validate inputs before processing:

```go
validator := wappalyzer.NewValidator()

// Validate URL
if err := validator.ValidateURL("https://example.com"); err != nil {
    log.Printf("Invalid URL: %v", err)
}

// Validate headers
if err := validator.ValidateHeaders(headers); err != nil {
    log.Printf("Invalid headers: %v", err)
}

// Validate body
if err := validator.ValidateBody(body); err != nil {
    log.Printf("Invalid body: %v", err)
}
```

## Benchmarking

Run benchmarks to test performance:

```sh
go test -bench=. -benchmem
```

## API Endpoints

When running the web server, the following endpoints are available:

- `GET /` - Web interface
- `POST /api/analyze` - Analyze a URL
- `GET /api/health` - Health check
- `GET /api/stats` - Server statistics
