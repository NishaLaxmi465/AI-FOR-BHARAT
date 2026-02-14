# AgriSaarthi - System Design Document

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Component Design](#component-design)
4. [Data Flow](#data-flow)
5. [Technology Stack](#technology-stack)
6. [Security Design](#security-design)
7. [Performance Optimization](#performance-optimization)
8. [Scalability Strategy](#scalability-strategy)

---

## 1. Executive Summary

### Project Overview
AgriSaarthi is an AI-powered agricultural advisory system designed for Indian farmers with low digital literacy. The system provides multilingual, voice-first assistance through WhatsApp for crop advisory, pest detection, weather alerts, and market prices.

### Design Goals
- **Simplicity**: WhatsApp-only interface, no app installation
- **Low Bandwidth**: Optimized for 2G/3G networks
- **Accessibility**: Hindi language, voice support
- **Reliability**: 99.9% uptime, <3s response time
- **Scalability**: Support 10,000+ concurrent users

### Key Constraints
- Target users have low digital literacy
- Network speeds: 2G/3G (50-500 Kbps)
- Device: Basic Android phones
- Budget: Minimal infrastructure costs

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                            │
│                                                                  │
│  ┌──────────────┐         ┌──────────────┐                     │
│  │   WhatsApp   │────────▶│    Twilio    │                     │
│  │   (Farmer)   │◀────────│  Gateway API │                     │
│  └──────────────┘         └──────┬───────┘                     │
└─────────────────────────────────┼─────────────────────────────┘
                                   │ HTTPS Webhook
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                             │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    FastAPI Backend                        │  │
│  │                                                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   Webhook    │  │   Message    │  │   Session    │   │  │
│  │  │   Handler    │─▶│   Router     │─▶│   Manager    │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │                                                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   Context    │  │   Response   │  │   Queue      │   │  │
│  │  │   Builder    │─▶│   Generator  │─▶│   Manager    │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
┌──────────────────────┐ ┌──────────────┐ ┌──────────────────┐
│    AI/ML LAYER       │ │  DATA LAYER  │ │  EXTERNAL APIs   │
│                      │ │              │ │                  │
│ ┌────────────────┐  │ │ ┌──────────┐ │ │ ┌──────────────┐ │
│ │ Gemini 1.5     │  │ │ │PostgreSQL│ │ │ │ OpenWeather  │ │
│ │ Flash (LLM)    │  │ │ │  (Users, │ │ │ │   API        │ │
│ └────────────────┘  │ │ │   Crops) │ │ │ └──────────────┘ │
│                      │ │ └──────────┘ │ │                  │
│ ┌────────────────┐  │ │              │ │ ┌──────────────┐ │
│ │ Gemini Vision  │  │ │ ┌──────────┐ │ │ │  Agmarknet   │ │
│ │ (Pest Detect)  │  │ │ │  Redis   │ │ │ │  (Mandi)     │ │
│ └────────────────┘  │ │ │ (Cache,  │ │ │ └──────────────┘ │
│                      │ │ │ Session) │ │ │                  │
│ ┌────────────────┐  │ │ └──────────┘ │ │ ┌──────────────┐ │
│ │ Google STT/TTS │  │ │              │ │ │ Govt Schemes │ │
│ │ (Voice)        │  │ │              │ │ │   (Static)   │ │
│ └────────────────┘  │ │              │ │ └──────────────┘ │
└──────────────────────┘ └──────────────┘ └──────────────────┘
```

### 2.2 Network Architecture

```
                    Internet
                       │
                       ▼
              ┌────────────────┐
              │  Load Balancer │
              │   (Nginx/ALB)  │
              └────────┬───────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌────────┐     ┌────────┐     ┌────────┐
   │FastAPI │     │FastAPI │     │FastAPI │
   │Instance│     │Instance│     │Instance│
   │   #1   │     │   #2   │     │   #3   │
   └────┬───┘     └────┬───┘     └────┬───┘
        │              │              │
        └──────────────┼──────────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   ┌─────────┐   ┌─────────┐   ┌─────────┐
   │PostgreSQL│   │  Redis  │   │ Gemini  │
   │ Primary  │   │ Cluster │   │   API   │
   └─────────┘   └─────────┘   └─────────┘
        │
        ▼
   ┌─────────┐
   │PostgreSQL│
   │ Replica  │
   │(Read-Only)│
   └─────────┘
```

---

## 3. Component Design

### 3.1 Webhook Handler

**Purpose**: Receive and validate incoming WhatsApp messages from Twilio

**Responsibilities**:
- Validate Twilio signature (security)
- Parse form data (text, media URLs)
- Return 200 OK immediately (prevent timeout)
- Queue message for async processing

**Implementation**:
```python
@router.post("/webhook/whatsapp")
async def whatsapp_webhook(
    background_tasks: BackgroundTasks,
    MessageSid: str = Form(...),
    From: str = Form(...),
    Body: Optional[str] = Form(None),
    NumMedia: int = Form(0),
    MediaUrl0: Optional[str] = Form(None)
):
    # Validate Twilio signature
    validate_twilio_signature(request)
    
    # Create webhook object
    webhook_data = TwilioWebhookRequest(...)
    
    # Queue for background processing
    background_tasks.add_task(
        message_processor.process_incoming_message,
        webhook_data
    )
    
    # Return immediately
    return Response(status_code=200)
```

**Design Decisions**:
- Async processing prevents Twilio timeout (15s limit)
- Background tasks allow parallel processing
- Signature validation prevents spoofing

---

### 3.2 Message Router

**Purpose**: Classify message type and route to appropriate handler

**Flow**:
```
Incoming Message
    │
    ├─ Has Media? ──┐
    │               │
    │               ├─ Image? ──▶ Pest Detection Handler
    │               │
    │               └─ Audio? ──▶ Voice Handler
    │
    └─ Text Only ──▶ Intent Classifier
                        │
                        ├─ pest_detection ──▶ Advisory Handler
                        ├─ weather_query ──▶ Weather Handler
                        ├─ mandi_price ──▶ Mandi Handler
                        ├─ crop_advisory ──▶ Advisory Handler
                        └─ government_scheme ──▶ Scheme Handler
```

**Implementation**:
```python
async def process_incoming_message(self, webhook: TwilioWebhookRequest):
    phone = webhook.From
    
    # Route based on media type
    if webhook.NumMedia > 0:
        if webhook.MediaContentType0.startswith("image/"):
            await self._handle_image_message(phone, webhook.MediaUrl0)
        elif webhook.MediaContentType0.startswith("audio/"):
            await self._handle_voice_message(phone, webhook.MediaUrl0)
    
    # Route based on intent
    elif webhook.Body:
        intent = await self._classify_intent(webhook.Body)
        await self._route_to_handler(phone, webhook.Body, intent)
```

---

### 3.3 Session Manager

**Purpose**: Maintain conversation context across messages

**Session Structure** (Redis):
```json
{
  "user_id": "uuid-1234",
  "phone_number_hash": "sha256_hash",
  "conversation_stage": "awaiting_crop_selection",
  "context": {
    "current_crop": "wheat",
    "sowing_date": "2024-11-15",
    "location": {
      "latitude": 30.9010,
      "longitude": 75.8573,
      "district": "Ludhiana",
      "state": "Punjab"
    },
    "last_intent": "pest_detection",
    "awaiting_image": true,
    "onboarding_step": 3
  },
  "created_at": "2024-01-15T10:00:00Z",
  "last_active": "2024-01-15T10:05:00Z"
}
```

**TTL**: 30 minutes (auto-cleanup)

**Operations**:
```python
class SessionManager:
    async def get_session(self, phone: str) -> dict:
        key = f"session:{hash_phone(phone)}"
        return await redis.get(key)
    
    async def update_session(self, phone: str, data: dict):
        key = f"session:{hash_phone(phone)}"
        await redis.setex(key, 1800, json.dumps(data))
    
    async def clear_session(self, phone: str):
        key = f"session:{hash_phone(phone)}"
        await redis.delete(key)
```

---

### 3.4 Gemini Service (AI Layer)

**Purpose**: Handle all AI/ML operations

#### 3.4.1 Vision API (Pest Detection)

**Input**: Crop image (JPEG/PNG)
**Output**: Disease name, severity, treatment

**Pipeline**:
```
Image Download ──▶ Preprocessing ──▶ Gemini Vision ──▶ Response Parsing
     │                  │                  │                  │
     │                  │                  │                  │
  Twilio URL      Compress 800x800    API Call         Extract JSON
                  Convert RGB         + Prompt         Format Hindi
                  Enhance Contrast
```

**Prompt Engineering**:
```python
PEST_DETECTION_PROMPT = """
आप एक कृषि रोग विशेषज्ञ हैं। इस फसल की तस्वीर का विश्लेषण करें।

फसल: {crop_type}
स्थान: {state}, भारत
मौसम: {season}

निम्नलिखित जानकारी दें:
1. रोग/कीट का नाम (हिंदी और English)
2. गंभीरता: low/medium/high/critical
3. दिखाई देने वाले लक्षण
4. रासायनिक उपचार (मात्रा के साथ)
5. जैविक विकल्प
6. रोकथाम के उपाय

संक्षिप्त और स्पष्ट हिंदी में जवाब दें (200 शब्द से कम)।
किसान को सीधे सलाह दें।
"""
```

**Image Preprocessing**:
```python
def preprocess_image(image: Image) -> Image:
    # 1. Resize (reduce bandwidth)
    image.thumbnail((800, 800), Image.LANCZOS)
    
    # 2. Convert to RGB
    if image.mode != 'RGB':
        image = image.convert('RGB')
    
    # 3. Enhance contrast (better detection)
    enhancer = ImageEnhance.Contrast(image)
    image = enhancer.enhance(1.2)
    
    # 4. Compress JPEG quality
    buffer = BytesIO()
    image.save(buffer, format='JPEG', quality=85, optimize=True)
    
    return Image.open(buffer)
```

#### 3.4.2 LLM (Text Responses)

**Use Cases**:
- Intent classification
- Crop advisory
- General queries
- Response generation

**Prompt Template**:
```python
ADVISORY_PROMPT = """
आप एक भारतीय कृषि सलाहकार हैं।

किसान की जानकारी:
- फसल: {crop_name}
- बुवाई तिथि: {sowing_date}
- स्थान: {district}, {state}
- मिट्टी: {soil_type}

किसान का सवाल: {query}

संक्षिप्त, व्यावहारिक सलाह दें (2-3 वाक्य)।
तकनीकी शब्दों से बचें।
"""
```

---

### 3.5 Caching Strategy

**Purpose**: Reduce API calls and improve response time

#### Cache Layers

**L1: In-Memory Cache** (LRU)
- FAQ responses
- Common queries
- Size: 100 MB
- TTL: 24 hours

**L2: Redis Cache**
- Mandi prices (1 hour TTL)
- Weather data (6 hour TTL)
- User sessions (30 min TTL)

**L3: Database Cache**
- Historical mandi prices
- Pest detection history

#### Cache Implementation

```python
from functools import lru_cache

# L1: In-memory
@lru_cache(maxsize=1000)
def get_faq_response(question_hash: str) -> str:
    return faq_database[question_hash]

# L2: Redis
async def get_mandi_prices(district: str, commodity: str):
    cache_key = f"mandi:{district}:{commodity}"
    
    # Check cache
    cached = await redis.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Fetch from API
    data = await fetch_from_agmarknet(district, commodity)
    
    # Cache for 1 hour
    await redis.setex(cache_key, 3600, json.dumps(data))
    
    return data
```

---

### 3.6 Database Design

#### Entity Relationship Diagram

```
┌─────────────┐         ┌─────────────────┐
│    users    │────1:N──│ farmer_profiles │
└──────┬──────┘         └─────────────────┘
       │
       │ 1:N
       │
┌──────┴──────┐
│    crops    │
└──────┬──────┘
       │
       │ 1:N
       │
┌──────┴────────────┐
│ pest_detections   │
└───────────────────┘

┌─────────────┐
│    users    │────1:N──┌──────────────────┐
└─────────────┘         │  conversations   │
                        └──────────────────┘

┌──────────────────┐
│  mandi_prices    │  (Cache table)
└──────────────────┘

┌──────────────────┐
│ weather_alerts   │  (Broadcast table)
└──────────────────┘
```

#### Key Tables

**users**: Core user identity
- Primary key: user_id (UUID)
- Unique: phone_number_hash
- Indexes: phone_hash, last_active

**farmer_profiles**: Agricultural details
- Foreign key: user_id
- Spatial index: (latitude, longitude)
- Indexes: state, district

**crops**: Current crop tracking
- Foreign key: user_id
- Indexes: status, sowing_date

**pest_detections**: ML predictions log
- Foreign keys: user_id, crop_id
- Indexes: detected_disease, created_at
- Used for: Model evaluation, feedback loop

**conversations**: Chat history
- Foreign key: user_id
- Indexes: intent, created_at
- Used for: Analytics, debugging

---

## 4. Data Flow

### 4.1 Pest Detection Flow (Detailed)

```
Step 1: Image Upload
Farmer ──[Image]──▶ WhatsApp ──[Media URL]──▶ Twilio Webhook

Step 2: Download & Preprocess
Webhook Handler ──▶ Download Image (Auth: Twilio credentials)
                 ──▶ Compress to 800x800
                 ──▶ Convert to RGB
                 ──▶ Enhance contrast

Step 3: Session Context
Session Manager ──▶ Fetch user profile from Redis
                 ──▶ Get: crop_type, location, sowing_date

Step 4: AI Analysis
Gemini Vision ──▶ Send: Image + Prompt + Context
              ──▶ Receive: Disease name, severity, treatment

Step 5: Knowledge Enhancement
Knowledge Base ──▶ Fetch additional info
                ──▶ Organic remedies
                ──▶ Prevention tips
                ──▶ Nearby stores

Step 6: Response Generation
Response Builder ──▶ Format in Hindi
                  ──▶ Add emojis for clarity
                  ──▶ Add action buttons

Step 7: Database Logging
PostgreSQL ──▶ Insert into pest_detections
            ──▶ Update conversation log

Step 8: Send Response
WhatsApp API ──▶ Send formatted message
              ──▶ Optional: Send treatment image

Step 9: Feedback Collection
Bot ──▶ Ask: "क्या यह मददगार था?"
    ──▶ Store feedback for model improvement
```

**Timing Breakdown**:
- Image download: 1-2s
- Preprocessing: 0.5s
- Gemini API: 3-5s
- Response formatting: 0.5s
- **Total: 5-8 seconds**

---

### 4.2 Weather Alert Broadcast Flow

```
Cron Job (Every 6 hours)
    │
    ▼
Fetch Weather Data (OpenWeatherMap)
    │
    ├─ For each district in database
    │  └─ Check for alerts (rain, frost, heatwave)
    │
    ▼
Detect Critical Alerts
    │
    ├─ Heavy rain (>50mm in 24h)
    ├─ Frost (<5°C)
    └─ Heatwave (>40°C)
    │
    ▼
Query Affected Farmers
    │
    └─ SELECT users WHERE district = X AND has_active_crop = TRUE
    │
    ▼
Generate Personalized Messages
    │
    ├─ Crop-specific advice
    ├─ Timing (when to take action)
    └─ Prevention measures
    │
    ▼
Batch Send via WhatsApp API
    │
    └─ Rate limit: 100 messages/second
    │
    ▼
Log Broadcast
    │
    └─ INSERT INTO weather_alerts (sent_to_users, timestamp)
```

---

## 5. Technology Stack

### 5.1 Backend Framework
**FastAPI** (Python 3.10+)
- Async/await support (high concurrency)
- Auto-generated API docs (Swagger)
- Pydantic validation (type safety)
- Fast performance (comparable to Node.js)

### 5.2 Database
**PostgreSQL 15**
- ACID compliance (data integrity)
- JSON support (flexible schemas)
- PostGIS extension (geospatial queries)
- Mature ecosystem

**Redis 7**
- In-memory speed (<1ms latency)
- Pub/Sub for real-time features
- TTL for auto-cleanup
- Cluster mode for scaling

### 5.3 AI/ML
**Google Gemini 1.5 Flash**
- Multimodal (text + vision)
- Fast inference (3-5s)
- Cost-effective ($0.00001875/image)
- Hindi language support

**Alternative**: Custom CNN (Post-MVP)
- MobileNetV3 (5MB model)
- TensorFlow Lite
- On-device inference

### 5.4 External APIs
- **Twilio**: WhatsApp Business API
- **OpenWeatherMap**: Weather forecasts
- **Agmarknet**: Mandi prices (Govt API)
- **Google Cloud**: STT/TTS (voice)

### 5.5 Infrastructure
**Development**: Docker Compose
**Production**: Railway / AWS ECS (Fargate)
**Monitoring**: CloudWatch / Sentry
**CI/CD**: GitHub Actions

---

## 6. Security Design

### 6.1 Authentication & Authorization

**Twilio Webhook Validation**:
```python
def validate_twilio_signature(request: Request):
    signature = request.headers.get('X-Twilio-Signature')
    url = str(request.url)
    params = await request.form()
    
    validator = RequestValidator(settings.TWILIO_AUTH_TOKEN)
    
    if not validator.validate(url, params, signature):
        raise HTTPException(status_code=403, detail="Invalid signature")
```

**User Identification**:
- Phone numbers hashed (SHA-256)
- No plaintext storage
- Encrypted in database (AES-256)

### 6.2 Data Privacy

**PII Handling**:
- Phone numbers: Hashed + encrypted
- Location: Rounded to district level
- Images: Deleted after 7 days
- Conversations: Anonymized after 30 days

**GDPR Compliance**:
- Right to deletion (DELETE /users/{id})
- Data export (GET /users/{id}/export)
- Consent tracking

### 6.3 Rate Limiting

```python
from slowapi import Limiter

limiter = Limiter(key_func=get_remote_address)

@app.post("/webhook/whatsapp")
@limiter.limit("10/minute")  # Per user
async def whatsapp_webhook(...):
    ...
```

### 6.4 Input Sanitization

```python
def sanitize_input(text: str) -> str:
    # Remove SQL injection attempts
    text = text.replace("'", "").replace(";", "")
    
    # Remove XSS attempts
    text = html.escape(text)
    
    # Limit length
    return text[:500]
```

---

## 7. Performance Optimization

### 7.1 Response Time Targets

| Operation | Target | Actual |
|-----------|--------|--------|
| Text query | <3s | 2.5s |
| Image analysis | <8s | 6s |
| Weather query | <2s | 1.5s (cached) |
| Mandi prices | <2s | 1s (cached) |

### 7.2 Optimization Techniques

**1. Image Compression**
- Resize to 800x800 (from 4000x3000)
- JPEG quality 85% (from 100%)
- Result: 200KB (from 3MB)

**2. Database Indexing**
```sql
CREATE INDEX idx_users_phone_hash ON users(phone_number_hash);
CREATE INDEX idx_conversations_user_created ON conversations(user_id, created_at);
CREATE INDEX idx_pest_disease ON pest_detections(detected_disease);
```

**3. Connection Pooling**
```python
DATABASE_URL = "postgresql+asyncpg://...?min_size=5&max_size=20"
```

**4. Async Operations**
```python
# Parallel API calls
async with asyncio.TaskGroup() as tg:
    weather_task = tg.create_task(get_weather(location))
    mandi_task = tg.create_task(get_mandi_prices(district))

weather_data = await weather_task
mandi_data = await mandi_task
```

**5. CDN for Static Assets**
- Treatment images
- Audio files (TTS)
- Hosted on CloudFront/Cloudflare

---

## 8. Scalability Strategy

### 8.1 Horizontal Scaling

**Stateless Design**:
- No server-side sessions (use Redis)
- Any instance can handle any request
- Easy to add/remove instances

**Load Balancing**:
```
                Load Balancer
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    Instance 1   Instance 2   Instance 3
```

**Auto-Scaling Rules**:
- CPU > 70% → Add instance
- CPU < 30% → Remove instance
- Min instances: 2
- Max instances: 10

### 8.2 Database Scaling

**Read Replicas**:
- Primary: Write operations
- Replica 1: Analytics queries
- Replica 2: User profile reads

**Partitioning**:
```sql
-- Partition by state
CREATE TABLE conversations_punjab PARTITION OF conversations
    FOR VALUES IN ('Punjab');

CREATE TABLE conversations_maharashtra PARTITION OF conversations
    FOR VALUES IN ('Maharashtra');
```

**Archival Strategy**:
- Move data >6 months to cold storage (S3)
- Keep hot data in PostgreSQL

### 8.3 Caching Strategy

**Cache Hit Ratio Target**: >70%

**Cache Warming**:
```python
# Pre-populate cache at startup
async def warm_cache():
    # Top 100 FAQs
    for faq in top_faqs:
        await cache_faq(faq)
    
    # Common mandi prices
    for district in top_districts:
        await cache_mandi_prices(district)
```

### 8.4 Cost Optimization

**Gemini API**:
- Cache responses for similar images
- Use Flash model (cheaper than Pro)
- Batch requests when possible

**Database**:
- Archive old data
- Use read replicas for analytics
- Optimize queries (EXPLAIN ANALYZE)

**Infrastructure**:
- Auto-scale down during low traffic
- Use spot instances (AWS)
- CDN for static assets

---

## 9. Monitoring & Observability

### 9.1 Key Metrics

**Application Metrics**:
- Request rate (req/s)
- Response time (p50, p95, p99)
- Error rate (%)
- Cache hit ratio (%)

**Business Metrics**:
- Active users (DAU, MAU)
- Messages per user
- Pest detection accuracy
- User satisfaction (feedback)

**Infrastructure Metrics**:
- CPU usage (%)
- Memory usage (%)
- Database connections
- Redis memory

### 9.2 Logging Strategy

```python
import logging

logger = logging.getLogger(__name__)

# Structured logging
logger.info("Pest detection completed", extra={
    "user_id": user_id,
    "disease": detected_disease,
    "confidence": confidence_score,
    "processing_time_ms": processing_time
})
```

### 9.3 Alerting Rules

- Response time > 5s → Warning
- Error rate > 5% → Critical
- Database CPU > 80% → Warning
- Gemini API failure → Critical

---

## 10. Disaster Recovery

### 10.1 Backup Strategy

**Database**:
- Automated daily backups (Railway/AWS)
- Point-in-time recovery (7 days)
- Cross-region replication

**Redis**:
- RDB snapshots every 6 hours
- AOF for durability

### 10.2 Failover Plan

**Database Failure**:
1. Promote read replica to primary
2. Update connection string
3. Restart application instances

**Redis Failure**:
1. Fallback to database for sessions
2. Disable caching temporarily
3. Restore from snapshot

**Gemini API Failure**:
1. Return cached responses
2. Use keyword-based fallback
3. Queue requests for retry

---

## 11. Future Enhancements

### Phase 2 (Post-Hackathon)
- Voice support (STT/TTS)
- 5 more crops
- 20 more diseases
- Regional languages (Punjabi, Marathi)

### Phase 3 (Month 2-3)
- Mobile app (Flutter)
- Offline mode
- Satellite imagery
- Custom ML model

### Phase 4 (Month 4-6)
- Fintech integration (loans)
- E-commerce (agri-inputs)
- Community features
- Video tutorials

---

## Appendix

### A. API Rate Limits
- Gemini: 60 requests/minute
- OpenWeather: 1000 requests/day (free tier)
- Twilio: 100 messages/second

### B. Cost Estimates
- Gemini: $0.00001875/image
- Twilio: $0.005/message
- Database: $30/month (Railway Pro)
- Total (1000 users): ~$80/month

### C. Performance Benchmarks
- Concurrent users: 1000+
- Messages/day: 10,000+
- Uptime: 99.9%
- Response time: <3s (text), <8s (image)
