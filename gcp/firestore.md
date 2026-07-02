# Cloud Firestore Cost Optimization & Research

Google Cloud Firestore is a serverless, NoSQL document database built for automatic scaling, high performance, and ease of application development. Firestore charges based on the **number of read, write, and delete operations** and the amount of storage consumed. Because it is serverless, an inefficiently designed frontend query or sync loop can quickly trigger millions of reads, leading to dramatic cost spikes.

---

## 1. Firestore Billing Components

Firestore costs are driven by four primary metrics:
1. **Reads (per 100,000 documents):** Billed when retrieving documents via queries or real-time listeners.
2. **Writes (per 100,000 documents):** Billed when creating, updating, or deleting documents.
3. **Storage (per GB/month):** Billed for data stored in documents, plus metadata and index overhead.
4. **Network Egress (per GB):** Billed for outbound data sent to clients outside Google's network.

---

## 2. Core Cost-Reduction Tactics

### A. Eliminate the `offset` Parameter (Use Cursors)
When paginating data, developers often use offset queries (e.g., "get page 5 by skipping 40 documents").
* **The Cost Trap:** Firestore charges you a document read for **every document skipped** by the offset. To skip 40 documents and read 10, you are billed for 50 document reads!
* **The Solution:** Use Query Cursors instead (e.g., `startAfter()` or `startAt()`). Page using the last document object or value from the previous query. Cursors do not incur charges for skipped documents.

### B. Implement Client-Side Caching & Offline Persistence
By default, every time an application reloads, it queries Firestore and downloads data again.
* **Action:** Enable **Offline Persistence** (cache) in the client SDK:
  * For web: `enableIndexedDbPersistence()`
  * For iOS/Android: Enabled by default.
* **The Benefit:** Subsequent reads are served from the local device storage rather than reaching the database, reducing server-side document read bills by up to 70%.

### C. Exclude Unused Fields from Automatic Indexing
Firestore automatically creates single-field indexes for every field in every document in a collection.
* **The Waste:** If a document contains large text fields, maps, JSON blobs, or arrays that are never used in query filters (`where()` clauses) or sort orders (`orderBy()`), indexing them is completely wasted.
* **Action:** Create **Index Exemptions** in the Firestore console or database configurations. Exclude large or high-frequency fields from indexing.
* **The Benefit:** Lowers storage costs (indexes consume substantial GB) and reduces write latencies/costs.

### D. Denormalize Schema to Avoid Join-Queries
In relational databases, you normalize data to prevent duplication. In Firestore, normalization is a cost trap.
* **The Trap:** If you need to show a user profile and their 5 recent posts, and you store them in separate collections, you must run 1 read for the profile and 5 reads for the posts (total 6 reads).
* **The Solution:** Denormalize data. Store basic user metadata (like profile picture and name) directly inside the post document.
* **The Benefit:** A single query fetches the posts and author profiles in 1 call, reducing read operations.

### E. Monitor Real-Time Listeners (`onSnapshot`)
* Real-time listeners automatically push updates to connected clients. 
* **The Billing:** You are charged for a full read of the initial query results. After that, if a document changes, you are only billed **1 read** for the changed document (not the entire query set). However, if the listener is disconnected for more than 30 minutes, it performs a full billable query upon reconnection.
* **Action:** Keep queries targeted and use listeners only when immediate synchronization is strictly required to avoid excessive reconnection reads on mobile devices. For static tables, use standard `get()` requests.

### F. Use Firestore Query Explain
* Use the **Query Explain** feature in development.
* By prefixing queries with explain parameters, you can see how many index entries were scanned. This helps you identify queries that are scanning large indexes to return a small subset of documents.

---

## 3. Firestore Audit Checklist

1. [ ] **Pagination Audit:** Verify that all pagination queries in the codebase use cursors (`startAfter`) rather than `offset`.
2. [ ] **Offline Persistence:** Ensure offline persistence is enabled in client-side applications.
3. [ ] **Index Exemption Sweep:** Audit fields containing large text strings or lists and exempt them from indexing.
4. [ ] **Listener Review:** Identify active real-time listeners and verify if they can be replaced by one-time `get()` operations.
5. [ ] **Explain Profile:** Run Query Explain on your top 10 most frequent queries to check for index-scanning efficiency.
6. [ ] **Regional Location Check:** Ensure the database is located in a single region (e.g. `us-central1`) rather than a multi-region if global failover is not required.
