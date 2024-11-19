# Seiðr

**Seidr** (`Seiðr`) is RESTfull API used for logging of http traffic from a proxy agent into elasticsearch for analytical and visualization purposes. It serves as a standardized logging service for http MitM proxies like Burpsuite or ZAP. The posibility to load offline captured traffic from burpsuite/ZAP/logger/pcap is planed to be implemented later.

## Usage

### seidr

The seidr APi service listens by default on `http://127.0.0.1:8008`. Network configuration, as well as other settings can be changed in `config.toml`

```
seidr

Seidr service.

Usage:
  seidr --config <config> [--verbose]
  seidr -h | --help
  seidr --version

Options:
  -l --config <config>	    File used for configuration, default: ./config.toml
  -d --daemon               Run as a daemon.
  -D --debug                Run in debug mode.
  -h --help                 Shows this screen.
  --version                 Shows version.
```

### seidr-keys

```
seidr-keys

Cli for the seidr service.

Usage:
  seidr-keys --test <apikey> [--verbose]
  seidr-keys --list 
  seidr-keys --count
  seidr-keys -h | --help
  seidr-keys --version

Required options:
  -i --id <identifier>	    User identifier of the user.
  -c --create               Create API key for user
  -d --delete [apikey]      Delete API key for user. 	 
  -t --test <apikey>        Test validity of key.

Options:
  -l --list             List existing apikeys or stats.
  -h --help             Shows this screen.
  --version             Shows version.
```

### API description

Detailed description is available in `Seidr.postman_collection.json` as a postman collection, or as OpenAPI in `Seidr-openapi.yaml` . The OpenaAPI file is generated fromt he postman collection using [postman2openapi](https://github.com/kevinswiber/postman2openapi).
The following list represents a shortened list of available endpoints.

* GET http://localhost:8008/ - Basic service information and version.
* POST http://localhost:8008/add - Endpoint to log new entry
* GET http://localhost:8008/stats - All stats for the service
* GET http://localhost:8008/stats/uptime - Specific stat only - uptime.
* GET http://localhost:8008/stats/entries - Specific stat only - total amount of entries.
* GET http://localhost:8008/stats/requests - Specific stat only - Total amount of requests - processed, processing errors, backend errors, unauthorized
* GET http://localhost:8008/stats/rpm - Specific stat only - count of requests per last minute - processed, processing errors, backend errors, unauthorized


## Configuration

Configuration options are set using `config.toml`.
Password for the PKCS12 client certificate for Elasticsearch is stored in `secrets/elasticsearch-keysent-cert-password.toml`.
Password for the Elasticsearch user is stored `secrets/elasticsearch-password.toml`.

## apikey

The keys are stored in the `api-keys` collection in the folowing format.

```json
{
    "apikey": "c475a3d1-8277-428f-bf5e-ab49dab6a680", // API key UID
    "id": "user@example.com", // user_id to pair traffic to user if needed ["UUID", "username", "email","ip-address"]
    }
```

## Input format

The values of `agent` will not be associated 1:1 to an apikey because tools such as burpsuite are using multiple agent names to distinguish traffic comming from various components, such as Scanner, Repeater and Proxy. The values will be evaluated against a static list of allowed values.

The data added via a `POST` request to  http://localhost:8008/add.
The `X-Apikey: c475a3d1-8277-428f-bf5e-ab49dab6a680` Header is required for authentication purposes.

```json
{
    "meta":{
        "agent": "burp-proxy", // name of agent ["zap","burp-proxy","burp-scanner","burp-repeater","seidr-proxy"]
        "group": 1234, // identifier used to create agent groups, for example a testing team
        "rt": 23, // response time in ms
        "error": false // Indication of an errneous request that contains no response
    },
    "request":"BASE64_ENCODED_REQUEST",
    "response":"BASE64_ENCODED_RESPONSE"
}
```


## Document format

Mimicking the input data fromat, extended with additional (extracted) metadata and parsed request/response. Data is stored in the `traffic` collection.

```json
{
    "meta":{
        "timestamp": 1731596481 // Epoch timestamp
        "agent": "burp-proxy", // name of agent ["zap","burp-proxy","burp-scanner","burp-repeater","seidr-proxy"]
        "id": "user@example.com", // user_id taken from api-keys
        "group": 1234, // identifier used to create agent groups, for example a testing team
        "rt": 23, // response time in ms
        "error": false, // Indication of an errneous request that contains no response
    },
    "request":{
        "method":"POST", // HTTP method ["GET","POST","PUT",..]
        "path":"/some/path", // path
        "url_params":"param1=val1&param2=val2", // url parameters
        "protocol": "HTTP/1.1", // used protocol
        "headers":{
            "Host": "example.com",
            "Cookie": "SessionId=d34db33f10O1",
            "Connection": "keep-alive"
            },
        "data":"multiline-string-of-data-section"
    },
    "response":{
        "protocol":"HTTP/1.1",
         "rc":200, 
         "rc-str":"OK",
        "headers":{
            "Cache-Control": "private",
            "Content-Type": "text/html; charset=utf-8",
            "Date": "Tue, 24 Oct 2023 14:11:58 GMT",
            "Content-Length:" 4534,
        },
        "data":"data":"multiline-string-of-data-section"
    }
}
```

## Visualization

Using Kibana and/or Grafana.

## Future plans

* Burpsuite plugin
* ZAP extension
* Standalone Golang proxy server
* HAProxy Stream Processing Offload Agent (SPOA)

## Disclaimer

Just a hobby project to explore and play around with new stuff. Do not take it too seriously.
