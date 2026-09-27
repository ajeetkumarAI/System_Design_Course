# System Design Interview: TikTok Architecture

---

## 1. Requirements & Scope Clarification

### Functional Requirements
1. **Video Uploading:** Creators can upload short video clips (typically 15s to 60s).
2. **Video Transcoding & Processing:** Videos must be transcoded into multiple resolutions and adaptive bitrate streams (DASH/HLS).
3. **Personalized Feed Generation ("For You" Page / FYP):** Real-time recommendation feed based on user interactions, likes, watches, skips, and profile metadata.
4. **Video Streaming/Consumption:** Fast, low-latency playback with prefetching for seamless vertical scrolling.
5. **User Engagement:** Actions like Liking, Commenting, Sharing, and Following.

### Non-Functional Requirements
* **Low Latency Feed & Playback:** Initial video loading time must be sub-second (< 200ms) to ensure smooth scrolling.
* **High Scalability:** 1 Billion Active Users, tens of millions of daily uploads, and petabytes of video traffic daily.
* **High Availability:** Read-heavy architecture (Reads to Writes ratio ~100:1) prioritizing availability over immediate consistency for non-critical features (e.g., like counters).
* **Data Consistency:** Eventual consistency for likes/views, high consistency for user authentication and direct actions.

---

## 2. Capacity & Scale Estimations

* **Daily Active Users (DAU):** 500 Million
* **Daily Video Uploads:** ~10 Million videos/day
* **Average Video Size (Post-Processing):** ~10 MB
* **Daily Video Consumption:** ~100 videos watched per user per day $\rightarrow 50\text{ Billion video views/day}$.

### Storage Estimation
* **Daily Raw Upload Storage:** $10\text{ Million videos} \times 10\text{ MB} = 100\text{ TB/day}$.
* **Transcoded Storage (5 formats/resolutions):** $100\text{ TB} \times 5 = 500\text{ TB/day}$.
* **Annual Storage:** $500\text{ TB/day} \times 365 \approx 182.5\text{ PB/year}$.

### Bandwidth Estimation
* **Egress (Download Bandwidth):** 
  $$\frac{50\text{ Billion views} \times 10\text{ MB}}{86400\text{ seconds}} \approx 5.78\text{ TB/s} = 46.24\text{ Tbps}$$
* Massive dependency on **Content Delivery Networks (CDNs)** with local edge caching.

---

## 3. High-Level Architecture Diagram

```
[ Mobile Client ]
       │
       ├──► (1) Upload Video ────► API Gateway ──► Upload Service ──► Blob Storage (S3)
       │                                                                  │
       │                                                       Event Bus (Kafka)
       │                                                                  │
       │                                            ┌─────────────────────┴─────────────────────┐
       │                                            ▼                                           ▼
       │                                    Transcoding Service                     Video Processing Pipeline
       │                                 (FFmpeg Cluster / CDN)                     (Extract Features, Audio, Tags)
       │                                            │                                           │
       │                                            └─────────────────────┬─────────────────────┘
       │                                                                  ▼
       │                                                         Metadata DB (Cassandra)
       │                                                                  │
       ├──► (2) Stream Video ◄── Edge CDNs / Cache Node ◄─────────────────┤
       │                                                                  │
       └──► (3) Fetch FYP ────► API Gateway ──► Recommendation Engine ◄──┘
                                                        │
                                            ┌───────────┴───────────┐
                                            ▼                       ▼
                                   In-Memory User Cache      Vector Search / ML Models
                                 (Redis / Aerospike)        (User/Video Embeddings)
```

---

## 4. Key Subsystem & Architecture Breakdown

### A. Video Upload & Transcoding Pipeline
1. **Direct-to-S3 Upload:** Client requests a pre-signed URL from the `Upload Service` to upload video binary directly to Blob Storage (e.g., AWS S3 / Distributed File System) without overloading application servers.
2. **Event Trigger:** S3 emits an `ObjectCreated` event into Apache Kafka.
3. **Transcoding Worker Pool:**
   * Workers pull video files and break them into chunks.
   * Transcode into multiple formats (H.264, HEVC, AV1) and resolutions (1080p, 720p, 480p) using HLS/DASH manifest protocols.
   * Extracts video thumbnail frame, audio track (for sound library mapping), and visual embeddings using ML models.

### B. Recommendation System ("For You" Page Engine)
The FYP architecture uses a multi-stage funnel:

1. **Candidate Generation (Retrieval):**
   * Narrows down millions of videos to ~1,000 candidates using user demographics, graph interactions, and collaborative filtering.
   * Employs vector search (e.g., Faiss, Milvus) over user and video embeddings.
2. **Scoring & Ranking:**
   * Deep Learning models (e.g., Deep & Cross Networks) score each candidate based on predicted probabilities:
     $$\text{Score} = w_1 \cdot P(\text{Watch Time}) + w_2 \cdot P(\text{Like}) + w_3 \cdot P(\text{Share}) - w_4 \cdot P(\text{Skip})$$
3. **Re-Ranking & Diversity:**
   * Removes duplicate audio/creator videos.
   * Injects exploration videos (cold start strategy for new videos or categories).

### C. Video Delivery & Prefetching
* **Edge CDN Caching:** Videos are cached on CDN nodes close to the user.
* **Smart Prefetching (Client-Side Optimization):**
  * While watching video $N$, video $N+1$ and $N+2$ are partially prefetched (first few seconds/chunks) to ensure zero buffer time when swiping.

---

## 5. Core Interview Questions & Detailed Answers

### Q1: How do you handle the "Cold Start" problem for newly uploaded videos?
**Answer:**
* **Metadata & Embedding Inference:** Upon upload, an ML vision model extracts tags, visual content features, and sound tracks to create an initial feature vector.
* **Exploration Bucket (A/B Testing):** The new video is immediately served to a small, diverse test group (e.g., 100-500 active users).
* **Feedback Loop:** If key metrics (completion rate, share rate, likes) exceed standard thresholds, the video is pushed into wider audience rings via the Candidate Retrieval pipeline.

### Q2: How do you handle high read throughput and low latency for the Feed API?
**Answer:**
* **Pre-computed Candidate Feeds:** Feeds are generated asynchronously in the background and stored in a high-throughput, low-latency in-memory cache (e.g., Redis or Aerospike).
* **Pagination & Cursor-based Fetching:** Instead of offset pagination, use cursor-based tokens containing timestamp and ranking scores to avoid duplicate/skipped items during real-time list mutation.
* **Multi-Layer CDN Strategy:** Dynamic manifest files are fetched via API, but heavy video segments (.ts / .m4s files) are cached aggressively across global CDNs.

### Q3: How do you count Likes, Views, and Shares at massive scale without database bottlenecks?
**Answer:**
* **Decoupled Aggregation Pipeline:** Client interactions are sent to Kafka/Pulsar topics.
* **Stream Processing (Flink/Spark):** Flink processes stream windows (e.g., 5-second tumbling windows) to aggregate counts in-memory.
* **In-Memory Cache Updates:** Consolidated count increments are written to Redis counters (`INCRBY`).
* **Batch Persistence:** Aggregated numbers are periodically batch-persisted to Cassandra/ScyllaDB for durability.

---

## 6. Trade-offs & Deep Dives

* **Consistency vs. Availability (CAP Theorem):** TikTok prioritizes **Availability and Partition Tolerance (AP)**. Seeing a slightly stale like count or follower count is acceptable, but feed unavailability or video load delay breaks user retention.
* **Push vs. Pull Architecture:**
  * **Pull Model:** Recommended for TikTok. Pre-computing feeds for all users (Push) wastes massive resources because feed algorithms are dynamic and continuous. Instead, candidates are generated on demand or continuously updated in small pre-cached windows.
