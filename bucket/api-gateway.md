# API Gateway
A centralized service acting as a reverse proxy. It accepts lient requests, can enforce some logic, and route traffic to backend sercvices and microservices.  
## What it can be used for?
1. Authentication / Authorization.  
Centralized token validation etc so that backend service / microervices dont need to handle login logic. 
2. IP Whitelisting / Blacklisting.
3. SSL / TLS Termination.  
Handle decryption of HTTPS traffic at the gateway level.
4. Rate Limiting and Throtelling
5. Load Balancing
6. Response Caching
7. Reverse Proxying  
Forwarding client requests to exact microservice needed based on URL path or header.
8. Protocol Translation.  
Convert client facing protocols (like REST) into backend friendly protocols (gRPC / GraphQL)
9. Request Aggrigation  
Combine data from multiple microservices into single JSON response, reducing network calls needed.
10. Centralized logging.  
11. Canary Deployments.  
Safely test new deployemnets by routing 5% of traffic to a new deployment and 95% stable version.  
12. Monetisation.  
Track API usage to inidvidual user or API key fir billing based in usage.  

Notes  
A reverse proxy is an intermediary server that sits in front of web servers. It intercepts incoming requests from clients on the internet and routes them to the appropriate backend server, acting as a traffic cop, bodyguard, and receptionist all in one. Forward Proxy: Sits in front of clients (users) to protect and hide the identity of the clients. It intercepts requests going out to the internet.Reverse Proxy: Sits in front of servers to protect and hide the identity of the servers. It intercepts requests coming in from the internet.  

## What I can build?
API Gateway that uses YARP (Yet Another Reverse Proxy) framework and ASP.NET Core middleware.  

```
[ CLIENT REQUEST ]
        https://yourcompany.com
        HTTP Header -> [X-User-Tier: Premium] | Method -> [GET]
                                 │
                                 ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│  API GATEWAY (Hosted Separately on Server A)                                  │
│                                                                               │
│  [ LAYER 1: NETWORK & INFRASTRUCTURE LAYER ]                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐    │
│  │ 3. SSL/TLS Termination  -> Decrypts incoming HTTPS to clear HTTP      │    │
│  │ 2. IP Access Control    -> Validates Client IP (Allow / Deny List)    │    │
│  └───────────────────────────────────────────────────────────────────────┘    │
│                                                                               │
│  [ LAYER 2: KESTREL & ASP.NET CORE MIDDLEWARE PIPELINE ]                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │ 4. Rate Limiting/Throttling -> Enforces quotas, drops malicious bursts  │  │
│  │ 1. Auth & Authorization     -> Decodes/validates JWT; extracts scopes   │  │
│  │ 6. Response Caching         -> Instantly returns data if cache hits     │  │
│  │ 12. Monetization Engine     -> Records usage per API Key for billing    │  │
│  │ 10. Centralized Logging     -> Streams request metrics & traces to cloud│  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                               │
│  [ LAYER 3: YARP CORE ENGINE & ROUTE MATCHING PIEPLINE ]                      │
│  ┌────────────────────────────────────────────────────────────────────────┐   │
│  │  PHASE A: Route Identification (Inspects Inbound Request)              │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐   │   │
│  │  │ 7. Reverse Proxy Path Match: URL starts with "/users/..."?      │   │   │
│  │  │    ├─► Header Check: Does request have [X-User-Tier: Premium]?  │   │   │
│  │  │    └─► Method Check: Is HTTP Verb a [GET] or [POST]?            │   │   │
│  │  └────────────────────────────────┬────────────────────────────────┘   │   │
│  │                                   │                                    │   │
│  │                                   ▼ (Match Confirmed!)                 │   │
│  │  PHASE B: Path Transformation                                          │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐   │   │
│  │  │ - "PathRemovePrefix": Strips "/users" from the inbound URL      │   │   │
│  │  │ - Transformed Internal Path Target: "/profile/avatar"           │   │   │
│  │  └────────────────────────────────┬────────────────────────────────┘   │   │
│  │                                   │                                    │   │
│  │                                   ▼                                    │   │
│  │  PHASE C: Cluster Selection & Traffic Rules                            │   │
│  │  ┌───────────────────────────────────────────────────────────────────┐ │   │
│  │  │ 11. Canary Rules Engine -> Evaluates deployment splits (95% vs 5%)│ │   |
│  │  │ 5.  Load Balancer       -> Selects healthiest backend server IP   │ │   |
│  │  └───────────────────────────────────────────────────────────────────┘ │   │
│  └────────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│  [ LAYER 4: PROTOCOL & AGGREGATION HANDLERS ]                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │ 8. Protocol Translation     -> Translates client REST to internal gRPC  │  │
│  │ 9. Request Aggregation      -> Resolves parallel sub-requests to a mesh │  │
│  └───────────────────────────────────┬─────────────────────────────────────┘  │
└──────────────────────────────────────┼────────────────────────────────────────┘
                                       │
           ┌───────────────────────────┴
           │ (Internal Private Network Route Map)                  
           │                                                       
           ├─► Path matched "/users"     ──► Forwarded to ──────┐  
           ├─► Header matched "Premium"   ──► Forwarded to ──┐  │  
           └─► Path matched "/orders"    ──► Forwarded to ─┐ │  │   
                                                           │ │  │   
                                                           │ │  │ [ ROUTED BY PATH ]
                                                           │ │  │ 🔗 http://10.0.0
                                                           │ │  ▼
                                                           │ │ ┌───────────────────┐
                                                           │ │ │  USER SERVICE     │
                                                           │ │ │  (Microservice)   │
                                                           │ │ └───────────────────┘
                                                           │ ▼ [ ROUTED BY HEADER ]
                                                           │   🔗 http://10.0.0
                                                           │   ┌───────────────────┐
                                                           │   │  PREMIUM BILLING  │
                                                           │   │  (Microservice)   │
                                                           │   └───────────────────┘
                                                           ▼ [ ROUTED BY CANARY WEIGHT ]
                                                             🔗 http://10.0.0 (5%)
                                                             ┌───────────────────┐
                                                             │  ORDER SERVICE V2 │
                                                             │  (Canary Node)    │
                                                             └───────────────────┘
```

For this architecture we can start with IP Access Control, Rate Limiting/Throtelling, Response Caching, Monetization Engine and Routing.   
This will be a serparate service and project. The Client connects to API Gateway and API Gateway communicates with our backend services/microservices.
Our backend microservices can live in a private network (VPC) with zero public IP addresses. They only accept traffic coming directly from your API Gateway's private IP.   

