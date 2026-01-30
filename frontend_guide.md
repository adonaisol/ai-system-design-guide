# Frontend System Design Interview Coaching: AI Bio Chatbot

## Interview Format

- **Duration:** 60 minutes
- **Type:** Whiteboarding exercise
- **Focus:** Frontend architecture for real-time AI chat
- **Scoring:** 10 points

---

## Key Differences from Backend Design

| Aspect | Backend Focus | Frontend Focus |
|--------|---------------|----------------|
| Primary concern | Data flow, storage, APIs | User experience, responsiveness |
| Scaling | Horizontal servers | Bundle size, render performance |
| State | Database, cache | Component state, global store |
| Real-time | WebSocket server | WebSocket client, streaming UI |
| Errors | HTTP codes, retries | User feedback, graceful degradation |

---

## The Golden Framework (Same Structure, Different Content)

```text
1. Clarify Requirements (5-7 min)
2. Define Scope & Constraints (3-5 min)
3. High-Level Architecture (10-15 min)
4. Deep Dive into Components (20-25 min)
5. Address Trade-offs & UX Considerations (10-15 min)
6. Wrap-up & Questions (5 min)
```

---

## Phase 1: Clarify Requirements (5-7 min)

### Questions to Ask

**User Experience:**

- "What's the target device? Desktop only, or mobile responsive?"
- "Expected user flow - form first, then chat, or combined?"
- "Should the bio stream in word-by-word or appear all at once?"
- "Any accessibility requirements (WCAG compliance)?"

**Technical Constraints:**

- "Framework preference? React, Vue, Angular, or framework-agnostic?"
- "Browser support requirements? (IE11, modern only?)"
- "Will this be embedded in Workday or standalone?"
- "Any existing design system to follow?"

**Real-time Requirements:**

- "What protocol for real-time? WebSocket, SSE, or polling?"
- "Should the chat maintain history across sessions?"
- "Offline support needed?"

**Integration:**

- "How do we authenticate with Workday?"
- "API format from backend? REST, GraphQL?"

---

## Phase 2: Define Scope (3-5 min)

### State Your Assumptions

> "Based on our discussion, I'll design for:
> - Modern browsers (Chrome, Firefox, Safari, Edge)
> - React-based SPA with TypeScript
> - WebSocket for real-time streaming
> - Mobile-responsive design
> - Integration via REST API + WebSocket"

### Out of Scope

> "I'll defer:
> - Detailed CSS/styling implementation
> - Workday-specific integration details
> - Backend implementation
> - Authentication flow (assume token provided)"

---

## Phase 3: High-Level Architecture (10-15 min)

### Draw This Architecture

```text
┌────────────────────────────────────────────────────────────────┐
│                        APPLICATION SHELL                       │
├────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │   Header     │  │  Bio Form    │  │    Chat Panel        │  │
│  │  Component   │  │  Component   │  │    Component         │  │
│  └──────────────┘  ├──────────────┤  ├──────────────────────┤  │
│                    │ - Name       │  │ - Message List       │  │
│                    │ - Role       │  │ - Input Field        │  │
│                    │ - Experience │  │ - Send Button        │  │
│                    │ - Skills     │  │ - Streaming Indicator│  │
│                    └──────────────┘  └──────────────────────┘  │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Bio Preview Panel                     │  │
│  │  - Current Bio Display                                   │  │
│  │  - Version History                                       │  │
│  │  - "Update Profile" Button                               │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                       STATE MANAGEMENT                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Form State │  │ Chat State  │  │    Bio State            │  │
│  │  (local)    │  │ (messages)  │  │ (current, history)      │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                       SERVICE LAYER                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   API Client    │  │ WebSocket Client│  │  Workday Client │  │
│  │   (REST)        │  │ (Real-time)     │  │  (Profile API)  │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Walk Through the User Flow

> "Let me walk through the user journey:
>
> 1. **Initial Load:** User sees bio form with input fields
> 2. **Form Submit:** User fills form, clicks 'Generate Bio'
> 3. **Streaming Response:** Bio streams in word-by-word via WebSocket
> 4. **Chat Refinement:** User types 'Make it more informal'
> 5. **Real-time Update:** New bio streams in, replacing previous
> 6. **Satisfaction:** User clicks 'Update Profile'
> 7. **Confirmation:** API call to Workday, success feedback shown"

---

## Phase 4: Deep Dive into Components (20-25 min)

### Component 1: State Management Architecture

**This is critical for frontend interviews!**

```text
┌─────────────────────────────────────────────────────────────┐
│                     GLOBAL STATE (Context/Redux)            │
├─────────────────────────────────────────────────────────────┤
│  user: { id, name, token }                                  │
│  bio: { current, versions[], isGenerating, error }          │
│  chat: { messages[], isStreaming, connectionStatus }        │
│  ui: { activePanel, notifications[] }                       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                     LOCAL STATE (useState)                  │
├─────────────────────────────────────────────────────────────┤
│  Form: { fieldValues, validationErrors, isDirty }           │
│  ChatInput: { currentMessage, isTyping }                    │
│  BioPreview: { isEditing, localEdits }                      │
└─────────────────────────────────────────────────────────────┘
```

**State Management Options:**

| Option | Pros | Cons | When to Use |
|--------|------|------|-------------|
| React Context | Built-in, simple | Re-render issues at scale | Small-medium apps |
| Redux Toolkit | Predictable, devtools | Boilerplate | Large apps, complex state |
| Zustand | Minimal, flexible | Less ecosystem | Medium apps, simple needs |
| Jotai/Recoil | Atomic, fine-grained | Learning curve | Complex derived state |

> "For this app, I'd recommend React Context + useReducer for global state, with local useState for form inputs. The state isn't complex enough to warrant Redux."

---

### Component 2: Real-time Communication (CRITICAL!)

**WebSocket vs Server-Sent Events (SSE)**

| Feature | WebSocket | SSE |
|---------|-----------|-----|
| Direction | Bidirectional | Server → Client only |
| Complexity | Higher | Lower |
| Reconnection | Manual | Automatic |
| Binary data | Yes | No |
| Use case | Chat, interactive | Streaming responses |

> "For this use case, I'd use a hybrid approach:
> - **SSE** for streaming bio generation (server → client)
> - **REST** for sending chat commands (client → server)
>
> This is simpler than full WebSocket and sufficient for our needs."

**Streaming Implementation:**

```typescript
// Pseudo-code for streaming bio
function useBioStream() {
  const [bio, setBio] = useState('');
  const [isStreaming, setIsStreaming] = useState(false);

  const startGeneration = async (formData: FormData) => {
    setIsStreaming(true);
    setBio('');

    const eventSource = new EventSource(
      `/api/generate-bio?${new URLSearchParams(formData)}`
    );

    eventSource.onmessage = (event) => {
      const chunk = JSON.parse(event.data);
      if (chunk.done) {
        eventSource.close();
        setIsStreaming(false);
      } else {
        setBio(prev => prev + chunk.text);
      }
    };

    eventSource.onerror = () => {
      eventSource.close();
      setIsStreaming(false);
      // Handle error
    };
  };

  return { bio, isStreaming, startGeneration };
}
```

---

### Component 3: Chat Interface

**Message Types:**

```typescript
type Message = {
  id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: Date;
  status: 'sending' | 'sent' | 'error';
  metadata?: {
    bioVersion?: number;
    tokens?: number;
  };
};

type ChatState = {
  messages: Message[];
  isStreaming: boolean;
  error: string | null;
};
```

**Optimistic Updates:**

> "For better UX, I'd implement optimistic updates:
>
> 1. User sends message → immediately show in chat (status: 'sending')
> 2. API call in background
> 3. On success → update status to 'sent'
> 4. On failure → show error, allow retry"

---

### Component 4: Error Handling (THEY MENTIONED THIS!)

**Error States to Handle:**

```text
┌─────────────────────────────────────────────────────────────┐
│                      ERROR TAXONOMY                         │
├─────────────────┬───────────────────────────────────────────┤
│ Network Errors  │ - Connection lost                         │
│                 │ - Timeout                                 │
│                 │ - Server unreachable                      │
├─────────────────┼───────────────────────────────────────────┤
│ API Errors      │ - 400: Invalid input                      │
│                 │ - 401: Unauthorized                       │
│                 │ - 429: Rate limited                       │
│                 │ - 500: Server error                       │
├─────────────────┼───────────────────────────────────────────┤
│ AI Errors       │ - Generation failed                       │
│                 │ - Content filtered                        │
│                 │ - Timeout (LLM slow)                      │
├─────────────────┼───────────────────────────────────────────┤
│ User Errors     │ - Validation failures                     │
│                 │ - Empty required fields                   │
└─────────────────┴───────────────────────────────────────────┘
```

**Error Handling Strategy:**

```typescript
// Error boundary for component-level errors
<ErrorBoundary fallback={<ErrorFallback />}>
  <ChatPanel />
</ErrorBoundary>

// API error handling
try {
  await sendMessage(content);
} catch (error) {
  if (error.status === 429) {
    showNotification('Rate limited. Please wait a moment.');
  } else if (error.status >= 500) {
    showNotification('Server error. Retrying...', { autoRetry: true });
  } else {
    showNotification(error.message);
  }
}
```

**User Feedback Patterns:**

| Scenario | Feedback |
|----------|----------|
| Sending message | Spinner on send button |
| Streaming bio | Typing indicator + partial text |
| Error occurred | Toast notification + retry option |
| Connection lost | Banner at top + auto-reconnect |
| Success | Green checkmark, brief message |

---

### Component 5: Performance Optimization

**Key Optimizations:**

1. **Virtualized Message List**

   > "For long chat histories, I'd use virtualization (react-window) to only render visible messages."

2. **Debounced Input**

   > "Debounce the chat input to prevent excessive re-renders while typing."

3. **Memoization**

   ```typescript
   // Memoize message components
   const MemoizedMessage = React.memo(MessageComponent);

   // Memoize expensive computations
   const bioWordCount = useMemo(() =>
     bio.split(' ').length, [bio]);
   ```

4. **Code Splitting**

   > "Lazy load the chat panel since users might not always use refinement."

   ```typescript
   const ChatPanel = React.lazy(() => import('./ChatPanel'));
   ```

---

## Phase 5: Trade-offs & UX Considerations (10-15 min)

### Key Trade-offs

| Decision | Option A | Option B | Recommendation |
|----------|----------|----------|----------------|
| State management | Context | Redux | Context (simpler for this scale) |
| Real-time | WebSocket | SSE | SSE for streaming (simpler) |
| Styling | CSS Modules | Tailwind | Depends on team preference |
| Form handling | React Hook Form | Formik | React Hook Form (lighter) |

### UX Considerations

**Loading States:**

> "Every async action needs a loading state:
>
> - Form submit → disable button, show spinner
> - Bio generation → skeleton or streaming text
> - Profile update → full-screen loader with progress"

**Accessibility:**

> "Key a11y considerations:
>
> - Announce new messages to screen readers (aria-live)
> - Keyboard navigation in chat
> - Focus management when switching panels
> - Color contrast for streaming text"

**Mobile Responsiveness:**

```text
Desktop Layout:
┌──────────┬──────────┬──────────┐
│   Form   │   Chat   │   Bio    │
└──────────┴──────────┴──────────┘

Mobile Layout:
┌──────────────────────────────────┐
│         Tab Navigation           │
├──────────────────────────────────┤
│                                  │
│      Active Panel Content        │
│                                  │
└──────────────────────────────────┘
```

---

## Phase 6: Wrap-up (5 min)

### Summarize Key Points

> "To summarize, I've designed a frontend that:
>
> 1. Provides real-time streaming UX via SSE
> 2. Manages state efficiently with Context + local state
> 3. Handles errors gracefully at every level
> 4. Optimizes performance with memoization and virtualization
> 5. Is accessible and mobile-responsive"

### Acknowledge Limitations

> "Given more time, I'd explore:
>
> - Offline support with service workers
> - Collaborative editing (multiple users)
> - Undo/redo for bio versions
> - Analytics integration"

---

## Common Follow-up Questions

### "How would you test this?"

> "Multi-level testing:
>
> 1. **Unit tests:** Individual components with React Testing Library
> 2. **Integration tests:** User flows (form → chat → save)
> 3. **E2E tests:** Cypress/Playwright for critical paths
> 4. **Visual regression:** Storybook + Chromatic"

### "How would you handle offline?"

> "Progressive enhancement:
>
> 1. Service worker caches static assets
> 2. IndexedDB stores draft bio and chat history
> 3. Background sync queues 'Update Profile' when offline
> 4. Clear UI indicator when offline"

### "What if the stream takes too long?"

> "Several strategies:
>
> 1. Show progress indicator ('Generating... 50%')
> 2. Allow cancellation with abort controller
> 3. Timeout after 30s with retry option
> 4. Show partial result with 'Continue generating' button"

### "How would you handle version history?"

> "Store versions in state:
>
> ```typescript
> bioState: {
>   current: 'Latest bio...',
>   versions: [
>     { id: 1, text: 'First version...', timestamp: Date },
>     { id: 2, text: 'After "more informal"...', timestamp: Date },
>   ],
>   activeVersion: 2
> }
> ```
>
> UI shows timeline, user can click to view/restore any version."

---

## Frontend-Specific Whiteboard Tips

### DO

- Sketch component hierarchy clearly
- Show data flow between components
- Discuss state placement decisions
- Mention specific libraries/tools
- Consider loading, error, and empty states

### DON'T

- Write actual CSS
- Get into implementation details too early
- Forget about mobile/responsive
- Ignore accessibility
- Skip error handling discussion

---

## Quick Reference: Frontend Design Checklist

```text
[ ] Clarified UX requirements (devices, browsers, a11y)
[ ] Stated tech stack assumptions
[ ] Drew component hierarchy
[ ] Explained state management approach
[ ] Covered real-time/streaming implementation
[ ] Discussed error handling strategy
[ ] Addressed performance optimizations
[ ] Considered accessibility
[ ] Mentioned testing strategy
[ ] Discussed trade-offs
[ ] Acknowledged mobile/responsive
```

---

## Comparison: Backend vs Frontend Interview

| Topic | Backend Answer | Frontend Answer |
|-------|---------------|-----------------|
| "How to scale?" | More servers, load balancer | Code splitting, lazy loading, CDN |
| "Handle errors?" | HTTP codes, retries, circuit breaker | Toast notifications, retry UI, error boundaries |
| "State management?" | Database, Redis cache | Redux/Context, local storage |
| "Real-time?" | WebSocket server, pub/sub | SSE/WebSocket client, optimistic UI |
| "Testing?" | Unit, integration, load | Unit, E2E, visual regression |

---

## Final Advice

1. **Think like a user** - Every technical decision should improve UX
2. **Show component thinking** - React/component architecture is expected
3. **State is king** - Where state lives determines your architecture
4. **Errors happen** - Always discuss how users recover from failures
5. **Performance matters** - Mention lazy loading, memoization, virtualization

Good luck!
