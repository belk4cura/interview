# System Design Examples for AWS Interviews

## Overview
System design interviews evaluate your ability to architect large-scale distributed systems. This guide provides detailed examples of common system design problems with AWS-specific solutions.

## Design Approach Framework

### 1. Requirements Gathering (5 minutes)
- **Functional Requirements**: What should the system do?
- **Non-Functional Requirements**: Scale, performance, reliability
- **Constraints**: Budget, timeline, technology limitations

### 2. Capacity Estimation (5 minutes)
- **Traffic**: Requests per second
- **Storage**: Data size and growth
- **Bandwidth**: Network requirements
- **Compute**: Processing needs

### 3. High-Level Design (10 minutes)
- **Architecture diagram**
- **Major components**
- **Data flow**

### 4. Detailed Design (20 minutes)
- **Component specifications**
- **Database schema**
- **API design**
- **Algorithm choices**

### 5. Scale & Optimize (10 minutes)
- **Bottleneck identification**
- **Caching strategy**
- **Database optimization**
- **Cost optimization**

## Example 1: URL Shortener (TinyURL Clone)

### Requirements
**Functional:**
- Shorten long URLs to 7-character short URLs
- Redirect short URLs to original URLs
- Custom aliases (optional)
- Analytics (click count, geographic data)
- URL expiration

**Non-Functional:**
- 100M URLs shortened per day
- < 100ms latency for redirection
- 99.9% availability
- Analytics can be eventually consistent

### Capacity Estimation
```
Write QPS: 100M / (24 * 3600) ≈ 1,200 URLs/second
Read QPS: Assume 100:1 read/write ratio = 120,000 redirects/second
Storage: 100M * 365 * 5 years * 500 bytes = ~91TB
Bandwidth: Read: 120K * 500 bytes = 60MB/s
Cache: 20% of daily traffic = 20M * 500 bytes = 10GB
```

### High-Level Architecture
```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Client    │────▶│  CloudFront  │────▶│   ALB       │
└─────────────┘     └──────────────┘     └─────────────┘
                            │                     │
                            ▼                     ▼
                    ┌──────────────┐     ┌─────────────┐
                    │  S3 Static   │     │  ECS/Lambda │
                    │   Content     │     │   Service   │
                    └──────────────┘     └─────────────┘
                                                 │
                            ┌────────────────────┼────────────────────┐
                            ▼                    ▼                    ▼
                    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
                    │ ElastiCache  │    │   DynamoDB   │    │   Kinesis    │
                    │   (Redis)     │    │  (URL Data)  │    │  (Analytics) │
                    └──────────────┘    └──────────────┘    └──────────────┘
```

### Detailed Design

**URL Shortening Service:**
```javascript
class URLShortener {
    constructor() {
        this.counter = new DistributedCounter(); // Using DynamoDB atomic counter
        this.base62 = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
    }
    
    async shortenURL(longURL, customAlias = null) {
        // Check if URL already exists
        const existing = await this.getExistingShortURL(longURL);
        if (existing) return existing;
        
        // Generate or validate short URL
        let shortURL;
        if (customAlias) {
            if (await this.isAliasAvailable(customAlias)) {
                shortURL = customAlias;
            } else {
                throw new Error('Alias not available');
            }
        } else {
            const id = await this.counter.getNextId();
            shortURL = this.encodeBase62(id);
        }
        
        // Store in database
        await this.storeURL(shortURL, longURL);
        
        // Invalidate cache if exists
        await this.cache.delete(shortURL);
        
        return shortURL;
    }
    
    encodeBase62(num) {
        let encoded = '';
        while (num > 0) {
            encoded = this.base62[num % 62] + encoded;
            num = Math.floor(num / 62);
        }
        return encoded.padStart(7, '0'); // Ensure 7 characters
    }
    
    async redirect(shortURL) {
        // Check cache first
        let longURL = await this.cache.get(shortURL);
        
        if (!longURL) {
            // Cache miss - fetch from database
            const data = await this.db.get(shortURL);
            if (!data) {
                throw new Error('URL not found');
            }
            
            longURL = data.longURL;
            
            // Update cache with TTL
            await this.cache.set(shortURL, longURL, 3600); // 1 hour TTL
        }
        
        // Async analytics
        this.analytics.track(shortURL);
        
        return longURL;
    }
}
```

**Database Schema (DynamoDB):**
```javascript
// Main table
{
    PK: "SHORT#<shortURL>",
    SK: "METADATA",
    longURL: "https://...",
    createdAt: "2024-01-01T00:00:00Z",
    expiresAt: "2025-01-01T00:00:00Z",
    userId: "user123",
    clickCount: 0
}

// GSI for reverse lookup
{
    GSI_PK: "URL_HASH#<md5(longURL)>",
    GSI_SK: "SHORT#<shortURL>"
}

// Analytics table (time-series)
{
    PK: "ANALYTICS#<shortURL>",
    SK: "TIMESTAMP#<timestamp>",
    ip: "192.168.1.1",
    userAgent: "...",
    referrer: "...",
    country: "US"
}
```

**Scaling Strategies:**
1. **Database Sharding**: Partition by first character of short URL
2. **Read Replicas**: DynamoDB global tables for geo-distribution
3. **Caching**: Multi-tier (CloudFront → ElastiCache → Application cache)
4. **Rate Limiting**: API Gateway throttling per API key
5. **Analytics**: Kinesis Data Firehose → S3 → Athena for batch processing

## Example 2: Video Streaming Platform (YouTube Clone)

### Requirements
**Functional:**
- Upload videos (up to 5GB)
- Stream videos (adaptive bitrate)
- Search videos
- Comments and likes
- Subscriptions and notifications
- Recommendations

**Non-Functional:**
- 1 billion users, 100 million daily active
- 500 hours of video uploaded per minute
- < 200ms latency for video start
- 99.99% availability

### Capacity Estimation
```
Upload: 500 hours * 60 min * 5GB/hour = 150TB/minute
Storage: 150TB * 60 * 24 * 365 = 78PB/year
Bandwidth: 100M users * 5 videos * 100MB = 50PB/day
CDN Cache: Popular 20% serves 80% traffic = 15PB
```

### Architecture
```
┌─────────────────────────── Upload Path ────────────────────────────┐
│                                                                     │
│  Client → S3 Transfer → Lambda → MediaConvert → S3 → CloudFront   │
│             Acceleration  (trigger)  (transcoding)                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────── Stream Path ────────────────────────────┐
│                                                                     │
│  Client → CloudFront → S3 (videos) → Adaptive Bitrate Streaming   │
│              ↓                                                     │
│          API Gateway → Lambda/ECS → DynamoDB (metadata)           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────── Metadata & Search ──────────────────────────┐
│                                                                     │
│  OpenSearch ← Kinesis ← DynamoDB Streams ← DynamoDB               │
│      ↓                                                            │
│  Search API → Lambda → API Gateway → Client                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Video Upload Pipeline
```javascript
class VideoUploadService {
    async initiateUpload(userId, metadata) {
        // Generate pre-signed S3 URL for multipart upload
        const uploadId = uuid();
        const s3Params = {
            Bucket: 'video-uploads',
            Key: `raw/${userId}/${uploadId}`,
            ContentType: metadata.contentType,
            Metadata: {
                userId,
                title: metadata.title,
                description: metadata.description
            }
        };
        
        const multipartUpload = await s3.createMultipartUpload(s3Params);
        
        // Store upload session
        await dynamodb.put({
            TableName: 'UploadSessions',
            Item: {
                uploadId,
                userId,
                status: 'INITIATED',
                parts: [],
                metadata,
                createdAt: Date.now()
            }
        });
        
        return {
            uploadId: multipartUpload.UploadId,
            uploadUrls: this.generatePartUploadUrls(multipartUpload)
        };
    }
    
    async processVideo(s3Event) {
        const { bucket, key } = s3Event;
        const videoId = uuid();
        
        // Start MediaConvert job
        const job = await mediaConvert.createJob({
            Input: `s3://${bucket}/${key}`,
            OutputGroups: [{
                Name: 'HLS',
                Outputs: [
                    { Resolution: '1080p', Bitrate: 5000 },
                    { Resolution: '720p', Bitrate: 2500 },
                    { Resolution: '480p', Bitrate: 1000 },
                    { Resolution: '360p', Bitrate: 500 }
                ],
                OutputGroupSettings: {
                    Type: 'HLS_GROUP_SETTINGS',
                    Destination: `s3://video-processed/${videoId}/`
                }
            }]
        });
        
        // Extract metadata and thumbnails
        await this.extractMetadata(videoId, `s3://${bucket}/${key}`);
        
        // Update database
        await this.updateVideoStatus(videoId, 'PROCESSING');
        
        return job.Id;
    }
}
```

### Recommendation System
```javascript
class RecommendationEngine {
    async getRecommendations(userId, limit = 20) {
        // Get user profile and history
        const [profile, history] = await Promise.all([
            this.getUserProfile(userId),
            this.getWatchHistory(userId, 100)
        ]);
        
        // Collaborative filtering
        const collaborativeRecs = await this.collaborativeFiltering(userId, history);
        
        // Content-based filtering
        const contentRecs = await this.contentBasedFiltering(profile, history);
        
        // Trending videos
        const trending = await this.getTrendingVideos();
        
        // Merge and rank
        const merged = this.mergeRecommendations(
            { collaborative: collaborativeRecs, weight: 0.4 },
            { content: contentRecs, weight: 0.3 },
            { trending: trending, weight: 0.3 }
        );
        
        // Filter watched videos
        const filtered = merged.filter(v => !history.includes(v.id));
        
        // A/B testing
        if (this.isInExperiment(userId, 'new_algorithm')) {
            return this.applyMLModel(filtered, profile);
        }
        
        return filtered.slice(0, limit);
    }
    
    async updateUserVector(userId, videoId, interaction) {
        // Update user embedding for real-time personalization
        const videoEmbedding = await this.getVideoEmbedding(videoId);
        const userEmbedding = await this.getUserEmbedding(userId);
        
        // Update based on interaction type
        const weight = {
            'view': 0.1,
            'like': 0.3,
            'share': 0.5,
            'subscribe': 0.7
        }[interaction];
        
        const newEmbedding = this.updateEmbedding(userEmbedding, videoEmbedding, weight);
        
        // Store in feature store
        await this.featureStore.put(userId, newEmbedding);
    }
}
```

## Example 3: Ride-Sharing Service (Uber Clone)

### Requirements
**Functional:**
- Request rides
- Match drivers with riders
- Real-time tracking
- Pricing calculation
- Payments
- Ratings

**Non-Functional:**
- 1 million concurrent rides
- < 5 seconds matching time
- Real-time location updates (every 4 seconds)
- 99.95% availability

### Architecture
```
┌────────────────── Real-time Location ──────────────────────┐
│                                                             │
│  Drivers → API Gateway → Kinesis → Lambda → DynamoDB      │
│     ↓                                    ↓                 │
│  WebSocket ← IoT Core ← Location Service → S3 (analytics) │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌────────────────── Matching Engine ──────────────────────────┐
│                                                              │
│  Ride Request → SQS → ECS (Matcher) → ElastiCache (Geo)    │
│                           ↓                                 │
│                    DynamoDB (Rides)                        │
│                           ↓                                 │
│                    SNS → Driver App                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Geospatial Matching Algorithm
```javascript
class GeoMatcher {
    constructor() {
        this.gridSize = 0.01; // ~1km grid cells
        this.redis = new Redis(); // For geo-indexing
    }
    
    async findNearbyDrivers(pickup, radius = 5) {
        // Use Redis GEO commands for efficiency
        const drivers = await this.redis.georadius(
            'drivers:available',
            pickup.lng,
            pickup.lat,
            radius,
            'km',
            'WITHDIST',
            'ASC'
        );
        
        // Filter by driver preferences and status
        const eligible = [];
        for (const [driverId, distance] of drivers) {
            const driver = await this.getDriverInfo(driverId);
            
            if (this.isEligible(driver, pickup)) {
                eligible.push({
                    id: driverId,
                    distance,
                    eta: this.calculateETA(distance, driver.location, pickup),
                    score: this.calculateScore(driver, distance)
                });
            }
        }
        
        return eligible.sort((a, b) => b.score - a.score);
    }
    
    calculateScore(driver, distance) {
        // Multi-factor scoring
        const factors = {
            distance: 1 - (distance / 10), // Normalize to 0-1
            rating: driver.rating / 5,
            completionRate: driver.completionRate,
            vehicleType: driver.isPremium ? 1.2 : 1.0
        };
        
        const weights = {
            distance: 0.4,
            rating: 0.3,
            completionRate: 0.2,
            vehicleType: 0.1
        };
        
        return Object.keys(factors).reduce((score, key) => 
            score + factors[key] * weights[key], 0
        );
    }
    
    async dispatchRide(rideRequest) {
        const { pickup, destination, userId } = rideRequest;
        
        // Find nearby drivers
        const drivers = await this.findNearbyDrivers(pickup);
        
        if (drivers.length === 0) {
            throw new Error('No drivers available');
        }
        
        // Try dispatching to drivers in order
        for (const driver of drivers.slice(0, 10)) {
            const accepted = await this.offerRide(driver.id, rideRequest);
            
            if (accepted) {
                await this.createRide({
                    rideId: uuid(),
                    driverId: driver.id,
                    userId,
                    pickup,
                    destination,
                    estimatedFare: this.calculateFare(pickup, destination),
                    status: 'MATCHED'
                });
                
                return driver;
            }
        }
        
        throw new Error('No driver accepted');
    }
}
```

### Dynamic Pricing Engine
```javascript
class DynamicPricing {
    async calculatePrice(pickup, destination, time = Date.now()) {
        const basePrice = this.calculateBasePrice(pickup, destination);
        const surgeFactor = await this.getSurgeFactor(pickup, time);
        
        return {
            base: basePrice,
            surge: surgeFactor,
            total: basePrice * surgeFactor,
            breakdown: {
                distance: basePrice * 0.7,
                time: basePrice * 0.2,
                service: basePrice * 0.1
            }
        };
    }
    
    async getSurgeFactor(location, time) {
        // Get supply and demand metrics
        const grid = this.getGrid(location);
        const metrics = await this.getMetrics(grid, time);
        
        const {
            availableDrivers,
            activeRides,
            pendingRequests,
            historicalDemand
        } = metrics;
        
        // Calculate real-time ratio
        const demandSupplyRatio = (activeRides + pendingRequests) / 
                                 Math.max(availableDrivers, 1);
        
        // Time-based factors
        const hourOfDay = new Date(time).getHours();
        const isRushHour = [7,8,9,17,18,19].includes(hourOfDay);
        const dayOfWeek = new Date(time).getDay();
        const isWeekend = [0,6].includes(dayOfWeek);
        
        // Calculate surge
        let surge = 1.0;
        
        if (demandSupplyRatio > 2) {
            surge = Math.min(1 + (demandSupplyRatio - 2) * 0.5, 3.0);
        }
        
        if (isRushHour) surge *= 1.2;
        if (isWeekend) surge *= 1.1;
        
        // Check for special events
        const eventSurge = await this.getEventSurge(location, time);
        surge *= eventSurge;
        
        return Math.min(surge, 5.0); // Cap at 5x
    }
}
```

## Common Design Patterns

### 1. Rate Limiting
```javascript
class RateLimiter {
    constructor(redis) {
        this.redis = redis;
        this.limits = {
            'api': { requests: 1000, window: 60 },
            'upload': { requests: 10, window: 3600 },
            'auth': { requests: 5, window: 300 }
        };
    }
    
    async isAllowed(userId, action) {
        const limit = this.limits[action];
        const key = `rate:${action}:${userId}`;
        const now = Date.now();
        const window = now - (limit.window * 1000);
        
        // Use Redis sorted set for sliding window
        await this.redis.zremrangebyscore(key, '-inf', window);
        const count = await this.redis.zcard(key);
        
        if (count < limit.requests) {
            await this.redis.zadd(key, now, uuid());
            await this.redis.expire(key, limit.window);
            return true;
        }
        
        return false;
    }
}
```

### 2. Circuit Breaker
```javascript
class CircuitBreaker {
    constructor(threshold = 5, timeout = 60000) {
        this.threshold = threshold;
        this.timeout = timeout;
        this.failures = 0;
        this.lastFailTime = null;
        this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    }
    
    async call(fn, ...args) {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailTime > this.timeout) {
                this.state = 'HALF_OPEN';
            } else {
                throw new Error('Circuit breaker is OPEN');
            }
        }
        
        try {
            const result = await fn(...args);
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }
    
    onSuccess() {
        this.failures = 0;
        this.state = 'CLOSED';
    }
    
    onFailure() {
        this.failures++;
        this.lastFailTime = Date.now();
        
        if (this.failures >= this.threshold) {
            this.state = 'OPEN';
        }
    }
}
```

### 3. Message Queue Pattern
```javascript
class MessageQueue {
    constructor(sqs, dlq) {
        this.sqs = sqs;
        this.dlq = dlq;
        this.maxRetries = 3;
    }
    
    async processMessages() {
        while (true) {
            const messages = await this.sqs.receiveMessage({
                QueueUrl: this.queueUrl,
                MaxNumberOfMessages: 10,
                WaitTimeSeconds: 20 // Long polling
            });
            
            if (!messages.Messages) continue;
            
            await Promise.all(messages.Messages.map(msg => 
                this.processMessage(msg)
            ));
        }
    }
    
    async processMessage(message) {
        const retryCount = parseInt(message.Attributes.ApproximateReceiveCount);
        
        try {
            await this.handler(JSON.parse(message.Body));
            await this.deleteMessage(message);
        } catch (error) {
            if (retryCount >= this.maxRetries) {
                await this.sendToDLQ(message);
                await this.deleteMessage(message);
            }
            // else let message return to queue for retry
        }
    }
}
```

## Scaling Strategies

### Horizontal Scaling
```yaml
# ECS Auto-scaling configuration
AutoScalingTarget:
  MinCapacity: 2
  MaxCapacity: 100
  TargetValue: 70
  ScaleInCooldown: 300
  ScaleOutCooldown: 60
  PredefinedMetricType: ECSServiceAverageCPUUtilization
```

### Database Scaling
```javascript
// Read replica routing
class DatabaseRouter {
    constructor(writer, readers) {
        this.writer = writer;
        this.readers = readers;
        this.currentReader = 0;
    }
    
    getConnection(operation) {
        if (operation === 'write') {
            return this.writer;
        }
        
        // Round-robin read replicas
        const reader = this.readers[this.currentReader];
        this.currentReader = (this.currentReader + 1) % this.readers.length;
        return reader;
    }
}
```

## Cost Optimization

### Reserved Capacity Planning
```javascript
class CapacityPlanner {
    analyzeUsage(metrics) {
        const baseline = this.calculatePercentile(metrics, 70);
        const peak = this.calculatePercentile(metrics, 95);
        
        return {
            reserved: baseline * 0.8, // 80% of baseline
            onDemand: baseline * 0.2, // 20% buffer
            spot: peak - baseline,     // Handle spikes with spot
            savings: this.calculateSavings(baseline, peak)
        };
    }
}
```

## Key Takeaways

1. **Start Simple**: Begin with monolith, evolve to microservices
2. **Data First**: Choose right database for each use case
3. **Cache Aggressively**: Multiple layers (CDN, Application, Database)
4. **Async Everything**: Decouple with queues and events
5. **Plan for Failure**: Circuit breakers, retries, fallbacks
6. **Monitor Everything**: Metrics, logs, traces, alarms
7. **Optimize Costs**: Right-sizing, reserved capacity, spot instances

---

*Previous: [← Behavioral Questions](./11-behavioral-questions.md)*
*Next: [Quick Reference →](./13-quick-reference.md)*
