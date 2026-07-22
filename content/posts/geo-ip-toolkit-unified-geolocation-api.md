+++
title = "A unified IP geolocation toolkit in Go"
date = 2025-02-23
type = "post"
description = "geo-ip-toolkit is an open-source Go service that queries multiple IP geolocation providers and returns one normalized response."
in_search_index = true
[taxonomies]
tags = ["Go", "Golang", "API", "Geolocation", "IP Intelligence", "REST API", "Open Source", "Backend Development", "Cybersecurity", "Network Programming"]
+++

If your application needs IP geolocation, for security monitoring, analytics, or content delivery, you end up integrating a provider like ipinfo.io or ip-api.com. Each has its own API shape, auth scheme, and response format. Committing to one means provider-specific parsing logic spread through your code and a single point of failure when that provider has downtime.

[geo-ip-toolkit](https://github.com/asikarwar007/geo-ip-toolkit) is my answer to that: an open-source Go service that queries either provider and returns one normalized response. Point it at an IP, pick a provider (or don't), and the JSON shape is the same either way.

## The core abstraction

Everything hangs off one interface:

```go
type IPInfoProvider interface {
    FetchIPInfo(ip string) (IPInfo, error)
}
```

The handler doesn't know which provider it's talking to. ipinfo.io, ip-api.com, or anything added later all satisfy the same contract. Adding a provider means implementing this one method plus a transformation function; the core handler doesn't change.

## Unified data model

To normalize the providers' different responses, the toolkit maps everything into a struct that covers the superset of available fields:

```go
type IPInfo struct {
    IP       string       `json:"ip"`
    Location LocationInfo `json:"location"`
    Isp      IspInfo      `json:"isp"`
    Privacy  PrivacyInfo  `json:"privacy"`
}

type LocationInfo struct {
    City          string  `json:"city"`
    District      string  `json:"district"`
    Region        string  `json:"region"`
    Country       string  `json:"country"`
    Timezone      string  `json:"timezone"`
    Lat           float64 `json:"lat"`
    Lon           float64 `json:"lon"`
    // ... additional fields
}

type PrivacyInfo struct {
    VPN     bool   `json:"vpn"`
    Proxy   bool   `json:"proxy"`
    Tor     bool   `json:"tor"`
    Hosting bool   `json:"hosting"`
    Mobile  bool   `json:"mobile"`
    // ... security flags
}
```

Whatever the underlying provider, callers get the same view: coordinates, ISP and ASN details, and the privacy flags (VPN, proxy, Tor, hosting) that matter for fraud and abuse detection.

## The HTTP server

The service is one REST endpoint that accepts an IP and a provider choice:

```go
func main() {
    http.HandleFunc("/info", infoHandler)
    log.Println("Server starting on http://localhost:8080...")
    http.ListenAndServe(":8080", nil)
}

func infoHandler(w http.ResponseWriter, r *http.Request) {
    queryParams := r.URL.Query()
    ip := queryParams.Get("ip")
    provider := queryParams.Get("provider")

    var ipProvider IPInfoProvider

    switch provider {
    case "ipinfo":
        ipProvider = ipinfoProvider{}
    default:
        ipProvider = ipAPIProvider{}
    }

    if ip == "" {
        ip, _, _ = net.SplitHostPort(r.RemoteAddr)
    }

    ipDetails, err := ipProvider.FetchIPInfo(ip)
    // ... error handling and response formatting
}
```

A few decisions in this handler: the provider is chosen per request via a query parameter, a missing `ip` parameter falls back to the requester's own address, failures return proper HTTP status codes, and responses are indented for readability during development.

## Provider implementations

Each provider implements `IPInfoProvider` with its own integration logic. The ipinfo.io version:

```go
func (p ipinfoProvider) FetchIPInfo(ip string) (IPInfo, error) {
    apiToken := os.Getenv("API_TOKEN")

    resp, err := http.Get("https://ipinfo.io/" + ip + "/json?token=" + apiToken)
    return parseIPInfoResponse(resp, err)
}

func parseIPInfoResponse(resp *http.Response, err error) (IPInfo, error) {
    var apiResult ipInfoResponse
    var result IPInfo

    if err != nil {
        return result, err
    }
    defer resp.Body.Close()

    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        return result, err
    }

    if err := json.Unmarshal(body, &apiResult); err != nil {
        return result, err
    }

    result = convertApiResponseToIpInfo(apiResult)
    return result, nil
}
```

The ip-api.com implementation follows the same pattern with a different endpoint:

```go
func (p ipAPIProvider) FetchIPInfo(ip string) (IPInfo, error) {
    resp, err := http.Get("http://ip-api.com/json/" + ip + "?fields=66846719")
    return parseIPAPIResponse(resp, err)
}
```

That `fields` value is a bitmask telling ip-api.com exactly which fields to include, which keeps the response payload to what the toolkit actually uses.

## Transforming responses

The transformation layer maps each provider's response struct to the unified model. For ip-api.com:

```go
func convertApiResponseToIpApi(apiResponse ipApiResponse) IPInfo {
    return IPInfo{
        IP: apiResponse.IP,
        Isp: IspInfo{
            Name:   apiResponse.Org,
            Asname: apiResponse.Asname,
            As:     apiResponse.As,
            Isp:    apiResponse.Isp,
        },
        Privacy: PrivacyInfo{
            Mobile:  apiResponse.Mobile,
            Proxy:   apiResponse.Proxy,
            Hosting: apiResponse.Hosting,
            VPN:     false,  // Not provided by this API
            Tor:     false,  // Not provided by this API
        },
        Location: LocationInfo{
            City:       apiResponse.City,
            Region:     apiResponse.RegionName,
            Country:    apiResponse.Country,
            Lat:        apiResponse.Lat,
            Lon:        apiResponse.Lon,
            Loc:        fmt.Sprintf("%f,%f", apiResponse.Lat, apiResponse.Lon),
            // ... additional mappings
        },
    }
}
```

When a provider doesn't offer a field (ip-api.com's free tier has no VPN or Tor detection), the transformation sets a default instead of failing the request. The response structure stays consistent even when provider capabilities differ.

Each provider gets its own response struct (`ipInfoResponse`, `ipApiResponse`) matching that API's exact shape, and a transformation function into `IPInfo`. The two-step approach keeps the mapping logic isolated and testable.

## Configuration

Credentials live in environment variables, loaded with `godotenv`:

```go
func init() {
    if err := godotenv.Load(); err != nil {
        log.Println("No .env file found")
    }
}
```

```bash
API_TOKEN=your_ipinfo_api_token_here
```

ipinfo.io needs a token; ip-api.com's basic tier doesn't. Each provider handles its own auth inside its implementation, so the interface stays clean.

## Using it

```bash
# Query using ipinfo.io
curl "http://localhost:8080/info?ip=8.8.8.8&provider=ipinfo"

# Query using ip-api.com
curl "http://localhost:8080/info?ip=1.1.1.1&provider=ipapi"

# Auto-detect requester's IP
curl "http://localhost:8080/info?provider=ipinfo"
```

The response shape is the same regardless of provider:

```json
{
  "ip": "8.8.8.8",
  "location": {
    "city": "Mountain View",
    "region": "California",
    "country": "United States",
    "timezone": "America/Los_Angeles",
    "lat": 37.4056,
    "lon": -122.0775
  },
  "isp": {
    "name": "GOOGLE",
    "asname": "GOOGLE",
    "type": "business"
  },
  "privacy": {
    "vpn": false,
    "proxy": false,
    "tor": false,
    "hosting": true
  }
}
```

## Why Go, and other trade-offs

Go fit this service well: the standard library covers the HTTP client and server, static typing catches integration mistakes at compile time, and the output is a single self-contained binary. Goroutines also leave the door open to querying several providers in parallel later, though v1 doesn't do that.

The current implementation queries providers synchronously. That keeps v1 simple and predictable. The obvious next steps are parallel queries with merged results, automatic fallback to another provider on failure, a cache for frequently queried IPs, and circuit breakers so one provider's outage doesn't cascade.

Rate limits are the other known gap: free tiers are limited, and the design leaves room for request caching and provider rotation, but v1 doesn't implement them.

## Getting started

```bash
# Clone the repository
git clone https://github.com/asikarwar007/geo-ip-toolkit
cd geo-ip-toolkit

# Configure API tokens
cp .env.example .env
# Edit .env and add your ipinfo.io token

# Run the service
go run .

# Test the API
curl "http://localhost:8080/info?ip=8.8.8.8&provider=ipinfo"
```

Requires Go 1.16+. The only external dependency is `godotenv`.

## Contributing

The project is open source under Apache License 2.0. To add a provider: implement `IPInfoProvider`, write the transformation for that provider's response format, add a case to the handler's switch, and include tests.

## Resources

- [github.com/asikarwar007/geo-ip-toolkit](https://github.com/asikarwar007/geo-ip-toolkit)
- [ipinfo.io/developers](https://ipinfo.io/developers)
- [ip-api.com/docs](https://ip-api.com/docs)
- [pkg.go.dev/net/http](https://pkg.go.dev/net/http)
