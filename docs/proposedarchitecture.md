++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
  # The fetch record logic consists here
    A[Client] --> B[API] (api routes and calls will remain same)
    B --> C{Record <90d?}  (adding custom logic for searching the record updated read module write will be same)
    C -->|Yes| D[Cosmos DB] (if record exists in cosmosdb process there)
    C -->|No| F[Check Blob Storage] (if record dors not exists in cosmosdb check in blob storage for same)
    F -->|Found| E[Return Record from Blob] (process record if exists in blob storage)
    F -->|Not Found| G[Return 404]  (if record does not exists in blob also continue with logic for state record unavailable)
  # The record migration logic consists here from cosmosdb to blob storage
    H[Archival Function | added custom cron in existing repos | seperate repo ] --> I[Query Cosmos DB for old records]
    I --> J[Write to Blob Storage]
    J --> K[Delete from Cosmos DB]
    L[Blob Lifecycle Policy] --> M[Move to Cool/Archive after 180d]
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
* Pros of proposed architecture
  **Significant Cost Reduction**: Azure Blob Storage (Cool/Archive tiers) is exponentially cheaper than Cosmos DB for rarely accessed data. This architecture allows ongoing cost savings as your dataset grows
  **No API Contract Changes**: Clients continue to use the same APIs and endpoints; the switch is entirely behind the scenes[attached_fileained Data Availability:** Old records remain accessible, just with a slightly longer retrieval time—within your defined SLA (order of seconds)
  **Flexible Organization**: Records can be archived Automated Lifecycle Policies: Blob Storage lifecycle can further reduce costs by tiering older archives, with no extra code requiredin a logical folder structure (e.g., by year/month/day), making management and auditing easier.
  **Automated Lifecycle Policies**: Blob Storage lifecycle can further reduce costs by tiering older archives, with no extra code required
  **Production-Grade Monitoring**: Built-in logging and monitoring (Azure Monitor, Application Insights) provide operational visibility across Cosmos DB, Blob Storage, and Azure Functions.
  _________________________________________________________________
* Cons of proposed architecture
  **Operational Complexity**: You must write, test, maintain, and monitor custom archival and retrieval logic—nowhere near as simple as a built-in feature
  **Multi-Component Dependency**: The system now depends on three Azure services (Cosmos DB, Blob Storage, Functions), each with its own failure modes and monitoring requirements. More components = more potential points of failure.
  **Latency for Archived Data**: While latency for archived records should remain within seconds, it will be higher than Cosmos DB (especially if retrieved from Archive tier), and cold start on Functions can occasionally slow things down.
  **Atomicity and Data Integrity**: Custom logic must ensure that records are only deleted from Cosmos DB after successful write to Blob Storage. Failure recovery (retries, dead-letter queues) adds complexity.
  


  
  
