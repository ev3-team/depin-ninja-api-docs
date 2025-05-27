# DePIN.Ninja API Guide

Welcome to **DePIN.Ninja API**!  
<br>We turn the raw onchain and offchain data produced by Decentralized Physical Infrastructure Networks (DePINs) into **transparent, standardized metrics** enabling developers and researchers to track and benchmark DePIN projects and power their own custom dashboards and analyses.
<br><br> DePIN.Ninja is a free, public API built by [EV3 Labs](https://ev3.xyz/):
* For a production API key, reach out to ninja@ev3.xyz. 
* To add a new project, see the [add project](#adding-new-projects) section. 
* To report a data error, reach out to us on [telegram](https://t.me/salgala).

<br>We are currently focused on providing **verified onchain revenue data** for DePINs. See our data in action at [DePIN Pulse](https://www.depinpulse.app/) or on [Chakra](https://console.chakra.dev/marketplace). Onchain revenue data is refreshed every 24 hours.

---

## TL;DR — Quick Start

1. **Get an API key** by emailing us at ninja@ev3.xyz.  
2. **Call** `GET /external-access/revenue` (see examples below).  
3. **Receive** JSON with project‑level revenue snapshots you can drop into Excel, Grafana, or your own app.

Every request must include your API key. Keys are rate‑limited to 60 requests per minute.

---

## What is Onchain Revenue?


DePINs are decentralized marketplaces for providing useful real-world services such as energy, bandwidth and compute, coordinated by smart contracts. **Onchain revenues** aims to measure the amount of **paid demand** for the core service offered by each protocol. For more a more in-depth discussion on the nature of DePIN onchain revenues, see EV3's [Q1'2025 Investor Letter](https://docsend.com/view/6pibwss8rhunkudx).

DePIN.Ninja tracks three types of revenue metrics. By default, the API returns `gross` revenues:

| Type | Description | Beneficiary |
|----------------|------------|--------------|
| **Gross** revenue | The total amount of paid demand for the network's services. | Miners (supply-side) and, sometimes, tokenholders. |
| **Net** revenue | The portion of paid demand used for token buybacks or burns. | Tokenholders. |
| **Unverified** revenue | Self-reported, offchain revenues that are not evident onchain. | Shareholders. |

<br>There is often ambiguity as to the source and amount of onchain revenues. Some DePINs claim to dedicate a fixed percentage of offchain company revenues to token buybacks, however this percentage cannot be verified onchain. Other DePINs use pre-mined tokens to pay for network services on behalf of B2B clients, effectively allowing the company to exit their token position indirectly rather than conduct open-market purchases that benefit all tokenholders.

To increase transparency we've added two fields to the standard API response, `revenueSource` and `disclaimer`, which provides a link to the underlying datasource as well as notable shortcomings in data transparency or verification for the specific project. 

---

## Retrieving Data

Retrieve live and historical revenue data with `GET /external-access/revenue`. 

#### Generic Request

```bash
curl -H "x-api-key: $YOUR_API_KEY" "https://api.depin.ninja/external-access/revenue"
```

<br>This will return a JSON with the following columns:


| Field | Type | Description |
|-----------|------|-------------|
| `projectId` | string | Unique six-digit alphanumeric identifier. |
| `projectName` | string | Project name, case-insensitive fuzzy match. |
| `category` | enum | Compute, energy, wireless, bandwidth, logistics, sensors, or human capital. |
| `chain` | enum | The blockchain where a project's smart contracts are deployed. |
| `revenueType` | enum | `Gross`, `net` or `unverified` revenues (`gross` by default). |
| `mrr` | integer | Represents trailing 30-day revenues in USD terms. |
| `arr` | integer | Represents annualized revenues, i.e. `mrr` multiplied by 12 months. |
| `revenueData` | array | Array with daily revenue data for the requested time period. |
| `revenueSource` | string | URL of the underlying data source (e.g., Dune query). |
| `disclaimer` | string | Disclaimers around revenue data transparency or verification concerns. |
| `updatedAt` | date | Revenue data is refreshed daily at 00:05 UTC. |

<br>

#### Request Parameters
The following parameters can be passed to refine the output: 

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | `1` | Pagination index. |
| `limit` | integer | `10` | Items per page; capped at 100. |
| `sortField` | string | `mrr`  | Rank by `projectName`, `projectId`, or `mrr`. |
| `sortOrder` | string | `desc`  | Rank by `asc` vs `desc` order. |
| `revenueType` | string | `gross`  | Toggle `gross` vs `net` vs `unverified` revenue. |
| `requestType` | string | `byProject`  | Stratify revenues `byProject` vs `byCategory` vs `byChain`. |
| `projectId` | string | —  | Only return one project; useful for cron jobs. |
| `projectName` | string | —  | Case‑insensitive fuzzy match. |
| `getRevenueHistoricalData` | boolean | `false` | `true` adds chart‑ready array for 90-day revenues. |

<br>

#### Example Requests

<br><br>To pull the most recently-updated data for all projects:

```bash
curl -H "x-api-key: $API_KEY" \
     "https://api.depin.ninja/external-access/revenue/latest"
```


<br>To pull revenues for a certain date, pass in the date field in `yyyy-mm-dd` format. For example, to pull revenues for the entire DePIN sector on April 19, 2025: 
```bash
curl -H "x-api-key: $API_KEY" \
     "https://api.depin.ninja/external-access/revenue/2025-04-19"
```
<details>
<summary>Example 200 Response</summary>

```jsonc
{"totalRevenue":92645.79955488985,
"breakDown":[{
    "name":"Althea","revenue":365.6336856858585},{
    "name":"Arkreen","revenue":0.34560796491746637},{
    "name":"Filecoin","revenue":39063.3857344671},{
    "name":"Helium","revenue":4683.05896},{
    "name":"Hivemapper","revenue":0.0022781750156353553},{
    "name":"IO.Net","revenue":40122.647946221274},{
    "name":"Livepeer","revenue":185.8753423756818},{
    "name":"NodeOps","revenue":7922.6},{
    "name":"PAAL AI","revenue":0},{
    "name":"Puffpaw","revenue":302.25},{
}
```
</details>

<br>To pull net revenues for Helium the last 90 days in chart-friendly format: 

```bash
curl -H "x-api-key: $API_KEY" \
     "https://api.depin.ninja/external-access/revenue?projectName=helium&revenueType=net&getRevenueHistoricalData=true"
```

<br>To pull revenues for projects with unverified (offchain) revenues, ranked from low to high:
 ```bash
curl -H "x-api-key: $API_KEY" \
     "https://api.depin.ninja/external-access/revenue?revenueType=unverified&sortField=mrr&sortOrder=asc"
```

<br>To compare the net revenues of the DePIN ecosystem across different blockchains on April 19, 2025:
 ```bash
curl -H "x-api-key: $API_KEY" \
     "https://api.depin.ninja/external-access/revenue/2025-04-19?revenueType=net&requestType=byChain"
```

---

## Adding New Projects

To add a new project to DePIN.Ninja, please email us at ninja@ev3.xyz with the following information:
1. Dune queries for the protocol's gross and/or net onchain revenues.
2. If onchain data is not available, a webhook that provides the protocol's daily offchain revenues.

We will inspect the queries and follow-up with questions if necessary. Project data typically goes live on the API within 1-2 weeks.

---

## Errors

| HTTP | Code string | When it happens | Tip |
|------|-------------|-----------------|-----|
| 400 | `http-400` | Missing or invalid parameters. | Check spelling & types. |
| 401 | `http-401` | Missing or bad API key. | Verify the `x-api-key` header. |
| 404 | `http-404` | Unknown project or date. | Confirm `projectId` or date exists. |
| 500 | `http-500` | Something went wrong server‑side. | Wait a moment & retry. |


<details>
<summary>Example Error response</summary>

```jsonc
{
  "status": "error",
  "code": "http-401",
  "message": "Invalid API key."
}
```
</details>

---

## Changelog
| Date | Change |
|------|--------|
| 2025‑04‑18 | First public release. |

---

## License
See [`MIT LICENSE`](https://opensource.org/license/mit). Commercial usage is permitted, but please attribute DePIN.Ninja whenever you display our data.

---

Need help, spot a bug, or want a higher rate limit?  Ping us at [ninja@ev3.xyz](mailto:ninja@ev3.xyz) or reach out on [Telegram](https://t.me/salgala).

