# AgriSaarthi - Requirements Document

## 1. Executive Summary

### 1.1 Problem Statement

**The Crisis**: India has 146 million farmers, yet 86% are small/marginal landholders with <2 hectares. They face:
- **Information Gap**: No access to real-time, personalized agricultural advice
- **Technology Barrier**: Existing agri-tech apps are heavy (>50MB), require smartphones and high-speed internet
- **Trust Deficit**: Farmers rely on local dealers who may upsell unnecessary chemicals
- **Economic Loss**: Crop failures due to delayed pest detection and poor market timing cost ₹50,000+ per farmer annually

**The Result**: Debt cycles, crop failures, and farmer distress despite government schemes worth ₹2.8 lakh crore.

### 1.2 Solution Overview

**AgriSaarthi** is an AI-powered agricultural advisory chatbot accessible via WhatsApp that provides:
- 🐛 **Pest Detection**: Image-based disease identification in <8 seconds
- 🌾 **Crop Advisory**: Stage-wise guidance on irrigation, fertilizers, and harvesting
- 🌤️ **Weather Intelligence**: Hyperlocal forecasts with farming-specific alerts
- 💰 **Market Prices**: Real-time mandi rates to prevent distress selling
- 📋 **Government Schemes**: Eligibility checking and application guidance

**Key Differentiators**:
- Zero app installation (WhatsApp-based)
- Works on 2G/3G networks
- Hindi language support
- Voice-first interface (future)
- Free for farmers

---

## 2. Target Users

### 2.1 Primary Users: Small & Marginal Farmers

**Demographics**:
- Age: 30-60 years
- Land holding: 0.5-2 hectares
- Location: Rural India (Punjab, Haryana, UP, Maharashtra)
- Education: 5-10 years of schooling
- Digital literacy: Low (can use WhatsApp for voice/images)
- Device: Basic Android phone (₹5,000-₹10,000)
- Network: 2G/3G (50-500 Kbps)

**Pain Points**:
- Cannot identify crop diseases early
- Miss optimal sowing/harvesting windows
- Sell at low prices due to lack of market information
- Unaware of government subsidies
- Cannot afford private agronomist consultations (₹500-₹1000/visit)

### 2.2 Secondary Users

**Agri-Input Dealers**: Partner for product recommendations
**Extension Officers**: Monitor farmer queries and trends
**Researchers**: Analyze pest outbreak patterns

---

## 3. Functional Requirements

### FR1: User Onboarding & Profile Management

**FR1.1**: System shall register users via WhatsApp using phone number (no password required)

**FR1.2**: System shall collect:
- Name
- Location (GPS or manual district/state entry)
- Primary crop(s)
- Farm size (optional)
- Sowing date

**FR1.3**: System shall support profile updates via conversational interface

**Acceptance Criteria**:
- Onboarding completes in <2 minutes
- Location accuracy: District level minimum
- Profile stored securely (phone number hashed)

---

### FR2: Pest & Disease Detection

**FR2.1**: System shall accept crop images via WhatsApp (JPEG/PNG, max 5MB)

**FR2.2**: System shall identify:
- Disease/pest name (English + Hindi)
- Severity level (low/medium/high/critical)
- Visible symptoms

**FR2.3**: System shall provide:
- Chemical treatment options (with dosage)
- Organic/biological alternatives
- Prevention measures
- Estimated recovery time

**FR2.4**: System shall suggest nearby agri-input stores (within 10km)

**FR2.5**: System shall collect farmer feedback (helpful/not helpful/wrong diagnosis)

**Acceptance Criteria**:
- Response time: <8 seconds
- Accuracy: >80% for MVP diseases (5 diseases)
- Confidence threshold: >70% to show result
- Supports wheat and rice crops initially

**MVP Diseases**:
1. Wheat: Rust, Leaf Blight
2. Rice: Blast, Brown Leaf Spot, Bacterial Leaf Blight

---

### FR3: Crop Advisory

**FR3.1**: System shall provide stage-wise guidance based on sowing date:
- Sowing recommendations
- Irrigation schedule (CRI, tillering, flowering stages)
- Fertilizer application (NPK ratios, timing)
- Pest monitoring windows
- Harvesting indicators

**FR3.2**: System shall recommend crops based on:
- Location (state/district)
- Season (kharif/rabi/zaid)
- Soil type (if provided)
- Historical yield data

**FR3.3**: System shall calculate current crop growth stage automatically

**Acceptance Criteria**:
- Stage calculation accuracy: ±2 days
- Recommendations localized to state/district
- Response time: <3 seconds

---

### FR4: Weather Intelligence

**FR4.1**: System shall provide 7-day weather forecast:
- Temperature (min/max)
- Rainfall probability
- Humidity
- Wind speed

**FR4.2**: System shall detect and alert for:
- Heavy rainfall (>50mm in 24h)
- Frost risk (<5°C)
- Heatwave (>40°C)
- Strong winds (>40 km/h)

**FR4.3**: System shall provide farming-specific advice:
- "Don't spray pesticides today (rain expected)"
- "Irrigate tomorrow morning (no rain for 3 days)"
- "Harvest within 2 days (heavy rain forecasted)"

**FR4.4**: System shall send proactive alerts to affected farmers

**Acceptance Criteria**:
- Forecast accuracy: >75% (3-day), >60% (7-day)
- Alert delivery: Within 6 hours of detection
- Response time: <2 seconds (cached)

---

### FR5: Market Price Information

**FR5.1**: System shall provide daily mandi prices for:
- User's district
- Nearby districts (within 50km)
- User's crop(s)

**FR5.2**: System shall display:
- Minimum, maximum, and modal (average) prices
- Price trend (up/down from last week)
- Comparison with MSP (Minimum Support Price)
- Arrival quantity (market supply indicator)

**FR5.3**: System shall recommend:
- Best time to sell (based on trends)
- Nearby mandis with better prices
- Storage advice if prices are low

**FR5.4**: System shall cache prices for 1 hour (updated 2x daily)

**Acceptance Criteria**:
- Data freshness: <24 hours
- Response time: <2 seconds (cached)
- Coverage: Top 50 mandis in 5 states (MVP)

---

### FR6: Government Schemes Information

**FR6.1**: System shall provide information on:
- PM-KISAN (direct benefit transfer)
- Pradhan Mantri Fasal Bima Yojana (crop insurance)
- Kisan Credit Card
- MSP rates
- State-specific subsidies

**FR6.2**: System shall check eligibility based on:
- Land holding size
- Crop type
- State/district

**FR6.3**: System shall provide:
- Scheme benefits (amount, frequency)
- Eligibility criteria
- Required documents
- Application process (step-by-step)
- Helpline numbers

**FR6.4**: System shall track application status (if API available)

**Acceptance Criteria**:
- Coverage: Top 5 central schemes + 2 state schemes per state
- Information accuracy: 100% (verified quarterly)
- Response time: <2 seconds

---

### FR7: Conversational Interface

**FR7.1**: System shall support Hindi language (primary)

**FR7.2**: System shall handle:
- Text messages
- Images (pest detection)
- Voice notes (future - Phase 2)

**FR7.3**: System shall maintain conversation context for 30 minutes

**FR7.4**: System shall provide:
- Button-based quick replies (1️⃣, 2️⃣, 3️⃣)
- Emojis for visual clarity (🌾, 🐛, 🌤️, 💰)
- Concise responses (<200 words)

**FR7.5**: System shall handle common queries:
- "मेरी फसल में कीड़े हैं" → Pest detection flow
- "मौसम कैसा रहेगा?" → Weather query
- "गेहूं का भाव क्या है?" → Mandi prices
- "PM-KISAN क्या है?" → Scheme information

**Acceptance Criteria**:
- Intent classification accuracy: >90%
- Context retention: 30 minutes
- Fallback to human support: <5% of queries

---

## 4. Non-Functional Requirements

### NFR1: Performance

**NFR1.1**: Response time targets:
- Text queries: <3 seconds (95th percentile)
- Image analysis: <8 seconds (95th percentile)
- Cached queries: <1 second

**NFR1.2**: System shall support 1000+ concurrent users

**NFR1.3**: API rate limits:
- 10 requests per minute per user (prevent abuse)
- 100 messages per second (system-wide)

---

### NFR2: Availability & Reliability

**NFR2.1**: System uptime: 99.9% (excluding planned maintenance)

**NFR2.2**: Planned maintenance window: Sunday 2-4 AM IST (max 2 hours/month)

**NFR2.3**: Data backup: Daily automated backups with 7-day retention

**NFR2.4**: Disaster recovery: <4 hour RTO (Recovery Time Objective)

---

### NFR3: Scalability

**NFR3.1**: System shall scale horizontally (add instances without code changes)

**NFR3.2**: Database shall support:
- 100,000 users
- 1 million conversations
- 500,000 pest detections

**NFR3.3**: Auto-scaling triggers:
- CPU >70% → Add instance
- CPU <30% → Remove instance

---

### NFR4: Security & Privacy

**NFR4.1**: Phone numbers shall be hashed (SHA-256) and encrypted (AES-256)

**NFR4.2**: Images shall be deleted after 7 days (GDPR compliance)

**NFR4.3**: No PII in logs or error messages

**NFR4.4**: HTTPS only (TLS 1.3)

**NFR4.5**: Twilio webhook signature validation (prevent spoofing)

**NFR4.6**: Rate limiting to prevent DDoS attacks

---

### NFR5: Usability

**NFR5.1**: Onboarding completion rate: >80%

**NFR5.2**: User satisfaction: >70% "helpful" responses

**NFR5.3**: Language: Simple Hindi (avoid technical jargon)

**NFR5.4**: Error messages: Clear and actionable
- ❌ "Error 500: Internal server error"
- ✅ "क्षमा करें, कुछ गलत हो गया। कृपया पुनः प्रयास करें।"

---

### NFR6: Bandwidth Optimization (Critical for 2G/3G)

**NFR6.1**: Image compression:
- Resize to 800x800 pixels
- JPEG quality: 85%
- Target size: <200KB

**NFR6.2**: Text responses: <500 characters (avoid long messages)

**NFR6.3**: Caching strategy:
- Mandi prices: 1 hour TTL
- Weather data: 6 hour TTL
- FAQ responses: 24 hour TTL

**NFR6.4**: No video content (bandwidth constraint)

---

### NFR7: Cost Efficiency

**NFR7.1**: Cost per user per month: <₹10

**NFR7.2**: API costs:
- Gemini: <₹5 per user per month
- Twilio: <₹2 per user per month

**NFR7.3**: Infrastructure: Use free tiers where possible (Railway, Gemini)

---

## 5. Constraints & Assumptions

### 5.1 Technical Constraints

- **Network**: Must work on 2G/3G (50-500 Kbps)
- **Device**: Basic Android phones (2GB RAM, 16GB storage)
- **Platform**: WhatsApp only (no native app for MVP)
- **Language**: Hindi only for MVP (English + regional languages in Phase 2)
- **Crops**: Wheat and rice only for MVP (expand to 15+ crops in Phase 2)

### 5.2 Business Constraints

- **Budget**: Minimal infrastructure costs (<₹5,000/month for MVP)
- **Timeline**: 7 days for hackathon MVP
- **Team**: 4-5 developers
- **Monetization**: Free for farmers (future: B2B model with agri-input companies)

### 5.3 Assumptions

- Farmers have WhatsApp installed (400M+ users in India)
- Farmers can take photos with their phones
- Farmers can read basic Hindi
- Internet connectivity available (even if slow)
- Twilio WhatsApp API remains accessible and affordable

---

## 6. Success Criteria

### 6.1 MVP Success (Hackathon)

- ✅ 100+ test users
- ✅ 2 crops supported (wheat, rice)
- ✅ 5 diseases detected with >80% accuracy
- ✅ <8 second image analysis
- ✅ Hindi language support
- ✅ Working live demo
- ✅ >70% user satisfaction

### 6.2 Production Success (3 months)

- ✅ 10,000 active users
- ✅ 40% Day-7 retention
- ✅ 99.9% uptime
- ✅ <₹10 cost per user per month
- ✅ 5 crops, 20 diseases
- ✅ Voice support (STT/TTS)
- ✅ 3 languages (Hindi, Punjabi, Marathi)

### 6.3 Impact Metrics (6 months)

- ✅ 10% crop yield improvement (farmer surveys)
- ✅ 75% pest control success rate
- ✅ ₹500 cost savings per farmer per season
- ✅ 20% increase in government scheme utilization

---

## 7. Out of Scope (MVP)

The following features are **NOT** included in the MVP but planned for future phases:

### Phase 2 (Post-Hackathon)
- Voice input/output (STT/TTS)
- Regional languages (Punjabi, Marathi, Telugu)
- 10+ additional crops
- 50+ diseases
- Soil testing recommendations
- Fertilizer calculator

### Phase 3 (Month 2-3)
- Mobile app (Flutter)
- Offline mode with cached FAQs
- Community features (farmer forums)
- Video tutorials
- Satellite imagery integration

### Phase 4 (Month 4-6)
- Fintech integration (micro-loans, insurance)
- E-commerce (agri-inputs marketplace)
- Drone-based crop monitoring
- Blockchain for supply chain traceability
- AI-powered yield prediction

---

## 8. Risks & Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Gemini API rate limits | High | Medium | Implement caching, use keyword fallback |
| Twilio costs exceed budget | High | Low | Monitor usage, set spending alerts |
| Low pest detection accuracy | High | Medium | Collect feedback, retrain model |
| Farmers don't adopt | High | Medium | User testing, simple onboarding |
| Network connectivity issues | Medium | High | Optimize for 2G/3G, retry logic |
| Data privacy concerns | High | Low | Encrypt PII, GDPR compliance |
| Seasonal usage spikes | Medium | High | Auto-scaling, load testing |

---

## 9. Stakeholder Sign-Off

| Stakeholder | Role | Approval |
|-------------|------|----------|
| Product Owner | Define requirements | ☐ |
| Tech Lead | Feasibility review | ☐ |
| UX Designer | Usability review | ☐ |
| Security Lead | Security review | ☐ |
| Farmer Representative | User validation | ☐ |

---

## 10. Appendix

### 10.1 Glossary

- **Kharif**: Monsoon crops (June-October) - Rice, Cotton, Maize
- **Rabi**: Winter crops (October-March) - Wheat, Mustard, Barley
- **Zaid**: Summer crops (March-June) - Vegetables, Melons
- **Mandi**: Agricultural market/wholesale market
- **MSP**: Minimum Support Price (government-guaranteed price)
- **PM-KISAN**: Direct income support scheme (₹6,000/year)
- **CRI**: Crown Root Initiation (wheat growth stage at 20-25 days)

### 10.2 References

- National Sample Survey (NSS) 77th Round - Agricultural Households
- Ministry of Agriculture & Farmers Welfare - Annual Report 2023
- NITI Aayog - Digital Agriculture Strategy
- Agmarknet API Documentation
- Twilio WhatsApp Business API Documentation
- Google Gemini API Documentation

---

**Document Version**: 1.0  
**Last Updated**: January 2025  
**Next Review**: Post-MVP (February 2025)
