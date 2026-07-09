# PingGlobe: Enterprise-Grade Distributed Cloud Observability Engine

PingGlobe is a multi-region, distributed synthetic monitoring platform engineered to observe, measure, and analyze global network decay and geographic routing inefficiencies. By executing concurrent, edge-dispatched L7 probes from distinct geographic cloud clusters, the system translates raw packet-round-trip telemetry into actionable edge-optimization insights.

## System Architecture & Data Flow

PingGlobe operates on a decoupled, serverless-first orchestration pipeline designed for maximum telemetry throughput and low edge latency. The complete end-to-end data lifecycle and execution pipeline flows as follows:

1. **Client Polling Trigger:** The React client initializes a non-blocking `12-second` scheduler loop in the user's browser, targeting the active URL node.
2. **Static Asset Edge Delivery:** The front-end SPA is compiled into static web assets and hosted in a high-availability AWS S3 production bucket (`s3://pingglobe-frontend-production`), securely fronted by a zero-cost CDN edge (e.g., Cloudflare Pages) providing automated SSL termination and global routing.
3. **API Aggregator Ingestion:** Dynamic client requests bypass static routing and target the AWS Application Load Balancer (ALB) gateway in `ap-south-1`.
4. **Middleware Load Balancing:** The ALB distributes incoming L7 HTTP probe commands to the Express-based Aggregator Engine (`server.mjs`) hosted on scalable EC2 instances.
5. **Parallel Regional Compute Dispatch:** The Aggregator extracts the target host and executes concurrent, asynchronous AWS Lambda invocations (`Promise.all()`) across three target AWS regions (`us-east-1`, `eu-west-1`, and `ap-south-1`) via the AWS SDK.
6. **Active Layer-7 Probing:** Each regional Lambda function initiates localized HTTPS handshakes to the targeted endpoint, tracking DNS resolution, connection, and transfer runtimes.
7. **Distributed Metrics Persistence:** Regional nodes log telemetry parameters directly into a clustered DynamoDB database to preserve temporal telemetry histories.
8. **Aggregation & Normalization:** The Express server normalizes the parallel regional outputs, flattens payload metrics, and streams the optimized JSON payload back to the React UI client.

### Component Interaction Blueprint

```
                                  +---------------------------------------+
                                  |            Browser Client             |
                                  |  [ React SPA Telemetry Interface ]    |
                                  +---------------------------------------+
                                    |                                   ^
              1. HTTP/S Asset Load  |                                   | 8. JSON Metric Stream
                                    v                                   |
+---------------------------------------+                               |
|        Edge SSL / Asset Storage       |                               |
| [ Cloudflare Pages / AWS S3 Bucket ]  |                               |
+---------------------------------------+                               |
                                    |                                   |
                                    | 2. 12s Polling (GET /metrics)     |
                                    v                                   |
+-----------------------------------------------------------------------+
|                       AWS Ingestion Gateway                           |
|                  [ Application Load Balancer (ALB) ]                  |
+-----------------------------------------------------------------------+
                                    |
                                    | 3. TCP Traffic Routing
                                    v
+-----------------------------------------------------------------------+
|                    Aggregator Compute Layer                           |
|             [ EC2 Aggregator Node (server.mjs) ]                       |
+-----------------------------------------------------------------------+
                                    |
                                    | 4. Concurrent Invocation (Promise.all)
                                    v
+-----------------------------------------------------------------------+
|               Multi-Region Serverless Probe Workers                   |
|   +-----------------------+  +---------------------+  +-----------+   |
|   | AWS Lambda (us-east-1)|  | AWS Lambda (eu-west)|  | ... (ap)  |   |
|   +-----------------------+  +---------------------+  +-----------+   |
+-----------------------------------------------------------------------+
               |                          |                   |
               | 5. L7 Probe              | 5. L7 Probe       | 5. L7 Probe
               v                          v                   v
+-----------------------------------------------------------------------+
|                            Target Host URL                            |
|                          (e.g., google.com)                           |
+-----------------------------------------------------------------------+
               |                          |                   |
               | 6. Telemetry Write       | 6. Telemetry Write| 6. Telemetry Write
               v                          v                   v
+-----------------------------------------------------------------------+
|                      Distributed Persistence                          |
|                     [ Amazon DynamoDB Tables ]                        |
+-----------------------------------------------------------------------+
```

---

## Deep-Dive: 10-Panel Telemetry Console Breakdown

The dashboard UI employs a responsive CSS Grid system (24-column layout) styled with Tailwind CSS dark-mode parameters, presenting ten optimized bento panels:

| # | Panel Name | Technical Subsystem | Production Problem Solved |
|---|---|---|---|
| **1** | **Global Incident Ticker** | Floating CSS Pill Marquee | **Mitigates alert fatigue** by streaming system packet metrics and region availability updates in a low-overhead ticker. |
| **2** | **Command Header** | Form Controller & Framer Motion SVG Globe | **Centralizes targeting and configuration control**, allowing SREs to update target nodes and toggle polling frequencies on the fly. |
| **3** | **AWS Core Region Cards** | Parallel Latency Matrices | **Isolates localized latency drift** across `us-east-1`, `eu-west-1`, and `ap-south-1` through responsive regional cards and color-coded status pulses. |
| **4** | **Global Metrics Health Gauge** | Animated SVG Health Calculator | **Speeds up operational triage** by transforming raw multi-region millisecond averages into a simplified health rating (Optimal, Nominal, Degraded). |
| **5** | **Historical Timelines Chart** | Recharts Area Chart with Custom Tooltips | **Identifies long-term performance degradation**, jitter, and anomalous network spikes through multi-point linear gradient data stream plots. |
| **6** | **Infrastructure Grid Matrix** | Operational Status Monitor | **Decouples application health from server health** by showing state mappings of internal Lambda, ALB, and Dynamo locks. |
| **7** | **CDN Cache Optimizer** | Cache Hit Ratio Estimator | **Optimizes edge server configuration** by computing hit ratios and tracking latency deficits to verify caching behavior. |
| **8** | **Regional DNS Propagation Radar** | Layer-7 Record Readout | **Exposes Split-Brain DNS conflicts** and routing anomalies by displaying active regional resolution records (A, AAAA, CNAME). |
| **9** | **Serverless Budget Tracker** | FinOps Pricing Engine | **Prevents ballooning cloud compute bills** by forecasting API costs per million invocations against actual regional request execution runtimes. |
| **10** | **Live Telemetry Terminal Console** | Log Console with State-Flash Hooks | **Provides developer-centric debugging**, capturing AWS connection handshakes, HTTP status codes, and edge probe warning logs. |

---

## Cloud Engineering & FinOps Highlights

* **Parallel Edge Compute Integration:** Utilizing `Promise.all()` to trigger simultaneous AWS SDK calls. This eliminates sequential invocation bottlenecks, slashing backend processing times to the duration of the slowest regional probe.
* **Efficient Telemetry Normalization:** The backend server flattens raw XML/SDK structures and database inputs into a single, payload-optimized JSON response, conserving network bandwidth for browser client interfaces.
* **FinOps Governance Controls:** Embedded pricing calculations link AWS Lambda duration billing models with actual target latency statistics, giving developers a direct visualization of real-time serverless operational budgets.
* **Zero-Cost Hybrid Edge Hosting:** By integrating Cloudflare Pages fronting an AWS S3 static asset build, the deployment architecture bypasses standard unencrypted bucket access issues, routing assets with global SSL termination without any extra cost.

---

## Local Installation & Deployment

### 1. Clone the Repository
```bash
git clone https://github.com/aws-project-ping/pingglobe-frontend.git
cd pingglobe-frontend
```

### 2. Install Project Dependencies
Run clean installations to align dependencies specified in your `package.json`:
```bash
npm install
```

### 3. Environment Configurations
Create a `.env` configuration file in the project directory to map Amazon Web Services credentials and target endpoints:
```env
AWS_ACCESS_KEY_ID=your_aws_access_key_id
AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key
AWS_REGION=ap-south-1
EXPRESS_PORT=3000
```

### 4. Initiate Development Servers
Execute the development pipeline:
* **Start Vite Frontend:**
  ```bash
  npm run dev
  ```
* **Start Node Aggregator:**
  ```bash
  node server.mjs
  ```
