# Backend System Design Interview Coaching: AI Bio Generator

## Interview Format

- **Duration:** 60 minutes
- **Type:** Whiteboarding exercise
- **Focus:** AI/GenAI system design
- **Scoring:** 10 points

---

## The Golden Rule: ALWAYS Structure Your Approach

Never dive straight into drawing boxes. Follow this framework:

```
1. Clarify Requirements (5-7 min)
2. Define Scope & Constraints (3-5 min)
3. High-Level Design (10-15 min)
4. Deep Dive into Components (20-25 min)
5. Address Trade-offs & Scaling (10-15 min)
6. Wrap-up & Questions (5 min)
```

---

## Phase 1: Clarify Requirements (5-7 min)

### What to Ask

**Functional Requirements:**
- "What information does the client collect? (name, job history, skills, achievements?)"
- "What's the expected output format? (paragraph, bullet points, length?)"
- "Should users be able to edit/regenerate the bio?"
- "Any personalization options? (tone: formal/casual, length: short/long?)"

**Non-Functional Requirements:**
- "What's the expected latency? (real-time <3s, or async is acceptable?)"
- "Expected traffic volume? (requests per second, daily active users?)"
- "Any compliance requirements? (PII handling, data retention?)"
- "Multi-region or single region deployment?"

**AI-Specific Questions:**
- "Which LLM provider? (OpenAI, Anthropic, internal model?)"
- "Any fallback if primary LLM is unavailable?"
- "Content moderation requirements?"

### Why This Matters

Asking questions shows:
- You don't make assumptions
- You understand business context matters
- You're thinking about edge cases early

---

## Phase 2: Define Scope & Constraints (3-5 min)

### State Your Assumptions

After clarifying, verbalize what you're designing for:

> "Based on our discussion, I'll design for:
> - ~10,000 DAU, peak of 100 requests/second
> - Target latency of <5 seconds for bio generation
> - Using a third-party LLM (e.g., OpenAI GPT-4)
> - Need to handle PII carefully
> - Single region initially, with path to multi-region"

### Out of Scope (Explicitly State)

> "For this exercise, I'll defer:
> - Authentication/authorization (assume handled by Workday)
> - Client-side implementation details
> - Detailed CI/CD pipeline"

---

## Phase 3: High-Level Design (10-15 min)

### Draw This Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│   Client    │────▶│   API       │────▶│  Bio Generation │────▶│   LLM API   │
│ (Workday)   │◀────│   Gateway   │◀────│     Service     │◀────│  (OpenAI)   │
└─────────────┘     └─────────────┘     └─────────────────┘     └─────────────┘
                           │                     │
                           ▼                     ▼
                    ┌─────────────┐     ┌─────────────────┐
                    │    Rate     │     │     Cache       │
                    │   Limiter   │     │    (Redis)      │
                    └─────────────┘     └─────────────────┘
                                               │
                                               ▼
                                        ┌─────────────────┐
                                        │   Observability │
                                        │  (Logs/Metrics) │
                                        └─────────────────┘
```

### Walk Through the Flow

> "Let me walk you through the happy path:
> 1. Client submits user data (name, experience, skills)
> 2. Request hits API Gateway for auth, rate limiting
> 3. Bio Generation Service validates input, constructs prompt
> 4. Check cache for similar requests (cost optimization)
> 5. Call LLM API with structured prompt
> 6. Log request/response for observability
> 7. Return generated bio to client
> 8. Client displays in Workday profile"

---

## Phase 4: Deep Dive into Components (20-25 min)

### Component 1: API Gateway

**What to discuss:**
- Rate limiting (per user, per org)
- Request validation
- Authentication (JWT, API keys)

**Say something like:**
> "The API Gateway handles cross-cutting concerns. For rate limiting, we'd implement token bucket algorithm - maybe 10 requests/minute per user to control LLM costs."

---

### Component 2: Bio Generation Service (CORE COMPONENT)

**This is where you spend the most time.**

#### Request Processing

```
Input Validation
     │
     ▼
Prompt Construction ──────▶ Prompt Template + User Data
     │
     ▼
Cache Check ──────────────▶ Hash of (user_data + settings)
     │
     ▼
LLM Call (if cache miss)
     │
     ▼
Response Processing ──────▶ Validation, Formatting
     │
     ▼
Cache Store + Return
```

#### Prompt Engineering (AI-Specific!)

> "For the prompt, I'd use a structured template:"

```
System: You are a professional bio writer. Create concise,
professional bios suitable for corporate profiles.

Context:
- Name: {name}
- Current Role: {role}
- Experience: {experience_summary}
- Key Skills: {skills}
- Achievements: {achievements}

Instructions:
- Write in third person
- Keep to 150-200 words
- Tone: {formal/casual}
- Highlight leadership and impact

Output the bio only, no additional commentary.
```

**Why this matters:**
- Structured prompts = consistent output
- Token efficiency = cost control
- Clear instructions = fewer retries

---

### Component 3: Caching Strategy (COST OPTIMIZATION)

**Two-level caching:**

1. **Exact Match Cache**
   - Key: Hash of (user_id + input_data + settings)
   - TTL: 24 hours
   - Use case: Same user regenerating

2. **Semantic Cache (Advanced)**
   - Store embeddings of inputs
   - Find similar past requests
   - Return cached result if similarity > threshold

> "Caching is critical for LLM cost control. If someone regenerates with same data, we return cached result instead of paying for another API call."

---

### Component 4: Observability (THEY MENTIONED THIS!)

**Three pillars:**

```
┌─────────────────────────────────────────────────────────┐
│                    OBSERVABILITY                        │
├─────────────────┬─────────────────┬─────────────────────┤
│     LOGGING     │     METRICS     │      TRACING        │
├─────────────────┼─────────────────┼─────────────────────┤
│ - Request/Resp  │ - Latency P50,  │ - End-to-end trace  │
│ - Prompt used   │   P95, P99      │ - LLM call duration │
│ - Token count   │ - Token usage   │ - Cache hit/miss    │
│ - Errors        │ - Cost per req  │ - Request ID prop   │
│ - Model version │ - Cache hit %   │                     │
└─────────────────┴─────────────────┴─────────────────────┘
```

**AI-Specific Observability:**
- **Token tracking:** Input tokens, output tokens, cost per request
- **Quality metrics:** User regeneration rate (high = poor quality)
- **Prompt versioning:** Track which prompt template version was used
- **Model comparison:** A/B test different models

> "For GenAI systems, I'd add specific metrics like token consumption, cost-per-bio, and regeneration rate. If users frequently regenerate, it signals quality issues."

---

### Component 5: Error Handling & Resilience

**LLM APIs are unreliable. Plan for:**

1. **Timeouts:** LLM calls can hang
   - Set aggressive timeout (30s)
   - Return graceful error to user

2. **Rate Limits:** Provider may throttle
   - Implement retry with exponential backoff
   - Queue requests if at limit

3. **Fallback Strategy:**
   - Primary: GPT-4 (quality)
   - Fallback: GPT-3.5 (faster, cheaper)
   - Last resort: Template-based bio (no LLM)

```
┌─────────┐     ┌─────────┐     ┌─────────────┐
│  GPT-4  │────▶│ GPT-3.5 │────▶│  Template   │
│ Primary │fail │Fallback │fail │  Fallback   │
└─────────┘     └─────────┘     └─────────────┘
```

---

## Phase 5: Address Trade-offs & Scaling (10-15 min)

### Trade-offs to Discuss

| Decision | Option A | Option B | Recommendation |
|----------|----------|----------|----------------|
| Sync vs Async | Synchronous (simple) | Async with polling (scalable) | Start sync, add async for long requests |
| Cache granularity | Per-user | Semantic similarity | Start per-user, add semantic later |
| Model choice | GPT-4 (quality) | GPT-3.5 (cost) | GPT-3.5 default, GPT-4 for premium |

### Scaling Considerations

**Horizontal Scaling:**
- Bio Generation Service is stateless → easy to scale
- Add more instances behind load balancer

**Cost Scaling:**
- Implement tiered pricing: free users get GPT-3.5, premium gets GPT-4
- Aggressive caching to reduce LLM calls
- Batch requests during off-peak (if async acceptable)

**Future Enhancements:**
- Fine-tuned model for bio generation (lower cost, better quality)
- RAG with company data for context-aware bios
- Multi-language support

---

## Phase 6: Wrap-up (5 min)

### Summarize Key Points

> "To summarize, I've designed a system that:
> 1. Handles the core flow from client to LLM and back
> 2. Optimizes costs through caching and rate limiting
> 3. Ensures reliability with fallback strategies
> 4. Provides observability for GenAI-specific metrics
> 5. Scales horizontally with stateless services"

### Acknowledge Limitations

> "Given more time, I'd explore:
> - Semantic caching for better cache hit rates
> - Fine-tuning a smaller model for cost efficiency
> - A/B testing framework for prompt optimization"

---

## Common Follow-up Questions & How to Answer

### "How would you handle PII?"

> "PII handling is critical. I'd:
> 1. Encrypt data in transit (TLS) and at rest
> 2. Minimize data sent to LLM - only necessary fields
> 3. Implement data retention policies - delete after X days
> 4. Consider on-premise LLM for highly sensitive data
> 5. Log redaction - mask PII in observability logs"

### "What if the LLM generates inappropriate content?"

> "Content safety is important. I'd add:
> 1. Output validation layer - check for profanity, bias
> 2. Use LLM's content filtering if available
> 3. Human review queue for flagged content
> 4. User feedback mechanism to report issues"

### "How would you reduce latency?"

> "Several strategies:
> 1. Use streaming responses - show bio as it generates
> 2. Pre-compute during off-peak for known users
> 3. Use faster model (GPT-3.5 vs GPT-4)
> 4. Optimize prompt length - fewer tokens = faster
> 5. Regional LLM endpoints for lower network latency"

### "How would you test this system?"

> "Multi-level testing:
> 1. Unit tests: Prompt construction, validation logic
> 2. Integration tests: Mock LLM responses
> 3. Load tests: Verify scaling under traffic
> 4. Quality tests: Human evaluation of generated bios
> 5. A/B tests: Compare prompt versions"

---

## Whiteboard Tips

### DO:
- Draw boxes and arrows clearly
- Label everything
- Talk while you draw
- Ask "Does this make sense so far?"
- Use consistent notation

### DON'T:
- Dive into code
- Over-engineer from the start
- Forget to discuss trade-offs
- Ignore the interviewer's hints
- Rush through without explaining

---

## Quick Reference: AI System Design Checklist

```
[ ] Clarified requirements (functional + non-functional)
[ ] Stated assumptions and scope
[ ] Drew high-level architecture
[ ] Explained request flow
[ ] Deep-dived into core components
[ ] Discussed prompt engineering strategy
[ ] Addressed caching for cost optimization
[ ] Covered observability (AI-specific metrics)
[ ] Explained error handling and fallbacks
[ ] Discussed trade-offs
[ ] Mentioned scaling strategy
[ ] Acknowledged limitations and future work
```

---

## Final Advice

1. **Structure > Perfection** - A structured incomplete answer beats a chaotic complete one
2. **Think out loud** - Let them see your reasoning process
3. **Engage the interviewer** - Ask if they want you to go deeper on any area
4. **AI-specific concerns** - Cost, latency, quality, safety are key for GenAI
5. **Be honest** - If you don't know something, say so and reason through it

Good luck!
