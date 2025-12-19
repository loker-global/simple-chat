# Simple Chat - Product Requirements Document (PRD)

## 1. Overview

### 1.1 Product Vision
Simple Chat is a modern, intuitive chat interface designed for seamless communication between users and conversational AI agents, robots, or other users. The application prioritizes a clean, responsive user experience with intelligent text input handling and comprehensive multimedia support.

### 1.2 Target Users
- End users seeking to interact with AI agents or chatbots
- Developers integrating conversational AI into their applications
- Organizations deploying customer support or virtual assistant solutions

### 1.3 Success Metrics
- User engagement time and message frequency
- Message delivery success rate (99.9% target)
- Text area responsiveness (<50ms resize latency)
- Support for 95%+ of common message formats

---

## 2. Core Features

### 2.1 Auto-Expanding Text Area (PRIMARY FOCUS)

#### 2.1.1 Overview
The text input area dynamically expands as users type, providing an optimal writing experience without manual resizing or scrolling within a confined space.

#### 2.1.2 Functional Requirements

**FR-2.1.2.1: Vertical Growth on Content**
- The text area MUST expand vertically when content exceeds the current visible area
- Expansion occurs in two scenarios:
  1. **Line Breaks**: When user presses Enter/Return (traditional behavior)
  2. **Horizontal Overflow** (PRIMARY FOCUS): When typing a continuous long text string that exceeds the width of the text area
- Growth must be smooth and non-disruptive to the typing flow

**FR-2.1.2.2: Dynamic Height Calculation**
- Initial height: 1 line (approximately 40-48px depending on font size)
- Minimum height: 1 line (must not shrink below initial state)
- Maximum height: 8 lines (approximately 320-384px)
- Growth increment: Seamless pixel-based expansion, not stepped line-by-line
- Beyond maximum height: Enable internal scrolling within the text area

**FR-2.1.2.3: Auto-Shrink on Content Deletion**
- When user deletes content, the text area MUST shrink accordingly
- Shrinking should mirror the expansion behavior (smooth, immediate)
- Must return to minimum height (1 line) when all content is cleared

**FR-2.1.2.4: Text Wrapping Behavior**
- Text MUST wrap at word boundaries by default (word-wrap: break-word)
- For extremely long words (>50 characters), allow character-level breaking
- Wrapping must trigger height expansion when content flows to a new line
- Preserve whitespace for code snippets or formatted text when pasted

#### 2.1.3 Technical Specifications

**TS-2.1.3.1: Implementation Approach**
```
Primary Method: CSS-based with JavaScript measurement fallback
- Use contenteditable div OR textarea with dynamic height calculation
- Monitor input/change events to recalculate height
- Measure scrollHeight vs clientHeight to determine expansion need
```

**TS-2.1.3.2: Performance Requirements**
- Height recalculation must occur within 16ms (60fps) of user input
- No visible jank or layout shift during expansion
- Debounce resize calculations if necessary (max 50ms delay)
- Use CSS transitions for smooth expansion (150-200ms duration)

**TS-2.1.3.3: Responsive Behavior**
- Text area width must adapt to container width (100% - padding/margins)
- On mobile devices (<768px): Consider full-width implementation
- On tablets (768px-1024px): Maintain responsive padding
- On desktop (>1024px): Maximum width constraint (e.g., 800px)

#### 2.1.4 Edge Cases and Constraints

**EC-2.1.4.1: Paste Operations**
- When pasting large amounts of text, expand immediately to accommodate
- If pasted content exceeds maximum height, show scrollbar immediately
- Preserve formatting of pasted content where possible

**EC-2.1.4.2: Browser Compatibility**
- Must work on Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- Graceful degradation on older browsers (fixed height with scroll)

**EC-2.1.4.3: Accessibility**
- Maintain proper ARIA labels for screen readers
- Ensure keyboard navigation works correctly (Tab, Shift+Tab)
- Announce height changes to assistive technologies
- Maintain minimum touch target size on mobile (44x44px)

**EC-2.1.4.4: IME (Input Method Editor) Support**
- Properly handle Asian language inputs (Chinese, Japanese, Korean)
- Expansion should account for composition events
- Do not expand during intermediate composition states

---

### 2.2 Multi-Media Message Support

#### 2.2.1 Overview
The chat interface supports sending and receiving diverse message types beyond plain text, enabling rich, multimedia conversations.

#### 2.2.2 Supported Message Types

**MT-2.2.2.1: Text Messages**
- Plain text with UTF-8 encoding
- Emoji support (full Unicode emoji set)
- Markdown rendering (optional):
  - Bold: **text** or __text__
  - Italic: *text* or _text_
  - Code: `code` or ```code block```
  - Links: Auto-detect and hyperlink URLs
- Maximum length: 10,000 characters per message

**MT-2.2.2.2: Image Messages**
- Supported formats: JPEG, PNG, GIF, WebP, SVG
- Maximum file size: 10MB per image
- Features:
  - Thumbnail preview in chat thread
  - Click to expand to full size (modal or lightbox)
  - Alt text support for accessibility
  - Loading states (skeleton/spinner)
  - Progressive loading for large images
- Display: Auto-fit to message container width, maintain aspect ratio
- Upload methods: 
  - Drag and drop into text area
  - Click to browse file system
  - Paste from clipboard (Ctrl+V / Cmd+V)

**MT-2.2.2.3: Video Messages**
- Supported formats: MP4, WebM, MOV, AVI
- Maximum file size: 100MB per video
- Features:
  - Inline video player with controls (play, pause, seek, volume, fullscreen)
  - Poster image/thumbnail before playback
  - Duration indicator
  - Loading states
  - Streaming support for large files (progressive download)
- Display: Fixed height (e.g., 240px) or responsive based on container
- Playback: Auto-pause when scrolled out of view (optional)

**MT-2.2.2.4: Voice Notes / Audio Messages**
- Supported formats: MP3, WAV, OGG, M4A, WebM (audio)
- Maximum file size: 25MB per audio file
- Features:
  - Inline audio player with waveform visualization (optional)
  - Playback controls: play/pause, seek, playback speed (1x, 1.5x, 2x)
  - Duration and current time display
  - Loading states
- Recording capability (optional):
  - Browser-based audio recording using MediaRecorder API
  - Real-time recording indicator
  - Maximum recording duration: 5 minutes
  - Preview before sending

**MT-2.2.2.5: Document Messages (PDF and others)**
- Supported formats: PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX, TXT, CSV
- Maximum file size: 50MB per document
- Features:
  - File name, size, and type icon display
  - Download button/link
  - PDF preview (embedded viewer or first-page thumbnail)
  - Virus scanning before upload (recommended)
  - File metadata display (page count for PDFs, word count for docs)
- Display: Card-style preview with icon, filename, and size

**MT-2.2.2.6: Link Previews**
- Auto-detect URLs in text messages
- Fetch and display rich previews (Open Graph metadata):
  - Page title
  - Description
  - Thumbnail image
  - Source domain
- Timeout for preview generation: 5 seconds
- Fallback to plain link if preview fails

#### 2.2.3 Message Composition UI

**UI-2.2.3.1: Attachment Button**
- Icon-based button (paperclip or plus icon) next to text area
- Click to open file picker with type filters
- Support multi-select for batch uploads (up to 10 files)

**UI-2.2.3.2: Drag and Drop Zone**
- Entire text area acts as drop zone
- Visual indicator when dragging files over the area
- Overlay message: "Drop files to upload"

**UI-2.2.3.3: Preview Before Sending**
- Show thumbnail/preview of attached media below text area
- Allow removal of attachments before sending (X button)
- Display file name and size
- Show upload progress bar for each file

**UI-2.2.3.4: Send Button**
- Enabled only when:
  - Text content is present (trimmed length > 0), OR
  - At least one valid attachment is ready
- Disabled during upload or when no content
- Visual states: default, hover, active, disabled
- Keyboard shortcut: Enter to send (Shift+Enter for new line)

#### 2.2.4 Message Display (Received Messages)

**MD-2.2.4.1: Message Bubble Layout**
- User messages: Right-aligned, distinct background color (e.g., blue)
- Agent/bot messages: Left-aligned, neutral background color (e.g., gray)
- Consistent padding: 12px horizontal, 8px vertical
- Border radius: 12-16px for modern look
- Maximum width: 70% of container width

**MD-2.2.4.2: Message Metadata**
- Timestamp: Display below each message or message group
- Sender name/avatar: For multi-user conversations
- Delivery status indicators (sent, delivered, read) - optional
- Message reactions (emoji reactions) - optional

**MD-2.2.4.3: Media Rendering**
- Images: Full-width within message bubble, rounded corners
- Videos: Aspect-ratio preserved, controls overlay
- Audio: Compact player bar with waveform
- Documents: Card with icon, download option
- Loading states: Skeleton screens or spinners

**MD-2.2.4.4: Grouping and Threading**
- Consecutive messages from same sender within 2 minutes: Group together
- Reduce vertical spacing between grouped messages
- Show timestamp only on first or last message of group

---

## 3. User Experience Requirements

### 3.1 Message Flow

**UX-3.1.1: Sending Messages**
1. User types in auto-expanding text area
2. Text area grows as content increases (up to max height)
3. User can optionally attach media via button or drag-and-drop
4. Preview of attachments appears below text area
5. User clicks Send button or presses Enter
6. Message appears immediately in chat thread (optimistic UI)
7. Loading indicator shown during upload/processing
8. Confirmation (checkmark, status change) when delivered

**UX-3.1.2: Receiving Messages**
1. New message appears at bottom of chat thread
2. Auto-scroll to newest message if user is near bottom (within 100px)
3. If user has scrolled up, show "New messages" indicator/button
4. Smooth entrance animation (fade-in, slide-up)
5. Media loads progressively (show skeleton → content)

### 3.2 Responsive Design

**RD-3.2.1: Mobile (320px - 767px)**
- Text area: Full width minus 16px margins
- Send button: Always visible (sticky or inline)
- Attachments: Stack vertically
- Message bubbles: Max width 85% of screen
- Optimize touch targets (minimum 44x44px)

**RD-3.2.2: Tablet (768px - 1023px)**
- Text area: Full width minus 24px margins
- Side-by-side attachment previews (2 per row)
- Message bubbles: Max width 75% of screen

**RD-3.2.3: Desktop (1024px+)**
- Text area: Max width 800px, centered or left-aligned
- Attachment previews: 3-4 per row
- Message bubbles: Max width 70% of screen
- Hover states for interactive elements

### 3.3 Accessibility

**A11Y-3.3.1: Keyboard Navigation**
- Tab through all interactive elements (text area, send button, attachments)
- Enter to send message (Shift+Enter for new line)
- Escape to cancel file upload or close previews
- Arrow keys to navigate through sent messages (optional)

**A11Y-3.3.2: Screen Reader Support**
- Semantic HTML (main, section, article, button, input)
- ARIA labels for icon-only buttons
- Announce new messages to screen readers
- Alt text for all images
- Transcripts or captions for video/audio (optional)

**A11Y-3.3.3: Visual Accessibility**
- Minimum contrast ratio: 4.5:1 for text, 3:1 for UI components (WCAG AA)
- Support for user font size preferences (rem-based sizing)
- Focus indicators on all interactive elements
- No reliance on color alone to convey information

### 3.4 Performance

**PERF-3.4.1: Load Time**
- Initial page load: <2 seconds on 4G connection
- Time to interactive: <3 seconds
- Lazy load message history (virtualized scrolling for 1000+ messages)

**PERF-3.4.2: Runtime Performance**
- Maintain 60fps during scrolling and typing
- Debounce text area resize calculations (max 50ms)
- Optimize media rendering (lazy load images outside viewport)
- Limit DOM nodes (virtualize long conversation threads)

**PERF-3.4.3: Network Efficiency**
- Compress media uploads (images: WebP conversion, videos: streaming)
- Resume interrupted uploads (chunked upload with retry)
- WebSocket or Server-Sent Events for real-time message delivery
- Request throttling to prevent API abuse

---

## 4. Technical Architecture

### 4.1 Frontend Components

**Component Hierarchy:**
```
ChatContainer
├── MessageList
│   ├── MessageBubble (user)
│   │   ├── TextContent
│   │   ├── MediaContent (Image/Video/Audio/Document)
│   │   └── MessageMetadata
│   └── MessageBubble (agent)
│       └── ...
└── MessageComposer
    ├── AutoExpandingTextArea
    ├── AttachmentButton
    ├── AttachmentPreviewList
    │   └── AttachmentPreview[]
    └── SendButton
```

### 4.2 State Management

**State Structure:**
```javascript
{
  messages: [
    {
      id: string,
      sender: 'user' | 'agent',
      timestamp: Date,
      content: {
        text?: string,
        attachments?: [
          {
            type: 'image' | 'video' | 'audio' | 'document',
            url: string,
            metadata: object
          }
        ]
      },
      status: 'sending' | 'sent' | 'delivered' | 'read' | 'failed'
    }
  ],
  composerState: {
    text: string,
    pendingAttachments: [],
    uploadProgress: {}
  },
  uiState: {
    isAtBottom: boolean,
    unreadCount: number
  }
}
```

### 4.3 Auto-Expanding Text Area Implementation

**Approach 1: Textarea with Dynamic Height (Recommended)**
```javascript
// Pseudo-code
function adjustTextAreaHeight(textarea) {
  // Store current scroll position to prevent jump
  const scrollPos = textarea.scrollTop;
  
  // Temporarily set to minimal height to get accurate scrollHeight
  textarea.style.height = '1px';
  
  // Calculate new height based on content
  const newHeight = Math.min(
    Math.max(textarea.scrollHeight, MIN_HEIGHT),
    MAX_HEIGHT
  );
  
  // Apply new height with transition
  textarea.style.height = newHeight + 'px';
  
  // Restore scroll position if needed
  if (scrollPos > 0) textarea.scrollTop = scrollPos;
  
  // Enable scroll if content exceeds max height
  textarea.style.overflowY = 
    textarea.scrollHeight > MAX_HEIGHT ? 'auto' : 'hidden';
}

// Attach to input event
textarea.addEventListener('input', () => adjustTextAreaHeight(textarea));
```

**Approach 2: ContentEditable Div with Shadow Measurement**
```javascript
// Pseudo-code
function adjustEditableHeight(editableDiv) {
  // Create hidden mirror div with same styling
  const mirror = createMirrorElement(editableDiv);
  mirror.textContent = editableDiv.textContent || ' ';
  
  // Measure mirror height
  const contentHeight = mirror.offsetHeight;
  
  // Apply clamped height
  const newHeight = Math.min(
    Math.max(contentHeight, MIN_HEIGHT),
    MAX_HEIGHT
  );
  
  editableDiv.style.height = newHeight + 'px';
  
  // Cleanup
  mirror.remove();
}
```

**CSS for Smooth Transitions:**
```css
.auto-expanding-textarea {
  min-height: 40px;
  max-height: 320px;
  resize: none;
  overflow-y: hidden;
  transition: height 0.15s ease-out;
  word-wrap: break-word;
  white-space: pre-wrap;
}
```

### 4.4 File Upload Architecture

**Upload Flow:**
1. User selects file(s) → Validation (size, type)
2. Generate preview/thumbnail (for images/videos)
3. Upload to server/storage (chunked for large files)
4. Receive URL/ID from server
5. Include URL in message payload
6. Send message

**Upload Service API:**
```
POST /api/upload
Content-Type: multipart/form-data

Response:
{
  success: true,
  file: {
    id: string,
    url: string,
    type: string,
    size: number,
    metadata: object
  }
}
```

---

## 5. Edge Cases and Error Handling

### 5.1 Text Area Edge Cases

**EC-5.1.1: Rapid Input**
- Debounce resize calculations to prevent performance issues
- Ensure final state is always accurate (no partial updates)

**EC-5.1.2: Copy-Paste Large Content**
- Immediately expand to accommodate pasted content
- If exceeds max height, scroll to bottom of text area

**EC-5.1.3: Font Size Changes**
- Recalculate height when user changes browser font size
- Use ResizeObserver to detect layout changes

**EC-5.1.4: Multi-line Paste with Mixed Content**
- Preserve line breaks from pasted content
- Expand appropriately for all lines

### 5.2 Media Upload Edge Cases

**EC-5.2.1: Network Failure During Upload**
- Show retry button with error message
- Allow user to continue typing and queue message
- Retry upload automatically (exponential backoff)

**EC-5.2.2: Unsupported File Type**
- Show clear error message: "File type not supported"
- List supported formats
- Reject file before upload attempt

**EC-5.2.3: File Size Exceeds Limit**
- Show error: "File too large. Maximum size: XX MB"
- Suggest compression or alternative formats
- Prevent upload attempt

**EC-5.2.4: Corrupted or Invalid Media File**
- Detect during validation (magic number check)
- Show error: "File appears to be corrupted"
- Allow user to try another file

### 5.3 Message Delivery Edge Cases

**EC-5.3.1: Message Fails to Send**
- Show error indicator on message bubble
- Provide retry button
- Allow deletion of failed message

**EC-5.3.2: Duplicate Message Prevention**
- Implement idempotency keys (client-generated message ID)
- Deduplicate on server based on message ID

**EC-5.3.3: Connection Loss**
- Show offline indicator
- Queue outgoing messages
- Auto-send when connection restored

---

## 6. Security and Privacy

### 6.1 File Upload Security

**S-6.1.1: File Validation**
- Verify file type using magic numbers, not just extension
- Scan uploaded files for malware/viruses
- Sanitize file names (remove special characters, limit length)

**S-6.1.2: Storage Security**
- Store uploaded files in secure, access-controlled storage
- Generate signed URLs with expiration for accessing media
- Implement rate limiting on upload endpoints

**S-6.1.3: Content Moderation**
- Optional: Scan images for inappropriate content
- Optional: Moderate text for profanity or harmful content

### 6.2 Data Privacy

**P-6.2.1: Message Encryption**
- Encrypt messages in transit (HTTPS/WSS)
- Optional: End-to-end encryption for sensitive conversations

**P-6.2.2: Data Retention**
- Define message retention policies (e.g., 30/60/90 days)
- Allow users to delete their messages
- Automatic deletion of media files after retention period

**P-6.2.3: User Consent**
- Obtain consent before accessing microphone (for voice notes)
- Obtain consent before accessing file system
- Clear privacy policy for data usage

---

## 7. Non-Functional Requirements

### 7.1 Scalability
- Support 10,000+ concurrent users
- Handle message throughput of 1000 messages/second
- Store and serve 1TB+ of media files

### 7.2 Reliability
- 99.9% uptime SLA
- Message delivery guarantee (at-least-once delivery)
- Automatic failover for backend services

### 7.3 Browser Compatibility
- Chrome 90+ (full support)
- Firefox 88+ (full support)
- Safari 14+ (full support, some media limitations)
- Edge 90+ (full support)
- Mobile browsers: iOS Safari 14+, Chrome Mobile 90+

### 7.4 Internationalization (i18n)
- Support RTL languages (Arabic, Hebrew)
- Proper Unicode handling for all languages
- Date/time formatting based on user locale

---

## 8. Future Enhancements (Out of Scope for v1)

### 8.1 Advanced Features
- Message editing and deletion
- Message reactions (emoji)
- Typing indicators
- Read receipts
- Message search
- Conversation threads/replies
- @mentions and notifications
- Code syntax highlighting in messages

### 8.2 Rich Input
- Slash commands (/help, /clear, etc.)
- @mentions autocomplete
- Emoji picker
- GIF search and insertion
- Voice-to-text transcription

### 8.3 Advanced Media
- Screen sharing
- Live video chat
- Collaborative whiteboard
- Location sharing

---

## 9. Success Criteria

The Simple Chat application will be considered successful when:

1. **Auto-Expanding Text Area Performance**
   - 95% of users experience <50ms resize latency
   - Zero reported issues with text overflow or content clipping
   - Smooth expansion on all supported browsers

2. **Multi-Media Support**
   - 99% successful upload rate for all supported file types
   - <5% error rate for media rendering
   - Average upload time <10 seconds for files <10MB

3. **User Satisfaction**
   - User satisfaction score >4.5/5
   - <2% bounce rate from the chat interface
   - Average session duration >5 minutes

4. **Performance Benchmarks**
   - Page load time <2 seconds (P50)
   - Time to interactive <3 seconds (P50)
   - Maintain 60fps during all interactions

5. **Reliability**
   - 99.9% uptime
   - <0.1% message delivery failure rate
   - Zero data loss incidents

---

## 10. Implementation Phases

### Phase 1: Core Chat with Auto-Expanding Text Area (MVP)
- Basic message send/receive
- Auto-expanding text area (primary focus feature)
- Text-only messages
- Real-time message delivery
- Responsive design (mobile, tablet, desktop)

### Phase 2: Image and Document Support
- Image upload, preview, and display
- Document upload and download
- Drag-and-drop support
- Preview before sending

### Phase 3: Video and Audio Support
- Video upload and inline playback
- Audio upload and playback
- Voice note recording (optional)
- Waveform visualization for audio

### Phase 4: Polish and Optimization
- Performance optimizations (virtualized scrolling, lazy loading)
- Accessibility improvements (WCAG AA compliance)
- Error handling refinements
- Link previews
- Message grouping and threading

---

## 11. Appendix

### 11.1 Glossary
- **Auto-Expanding Text Area**: Input field that dynamically adjusts height based on content
- **Message Bubble**: Visual container for individual messages in chat thread
- **Optimistic UI**: Immediately showing user action results before server confirmation
- **Virtualized Scrolling**: Rendering only visible messages for performance with large datasets

### 11.2 References
- Material Design Guidelines for Chat UI
- WCAG 2.1 Accessibility Standards
- Web Content Accessibility Guidelines (WCAG)
- MDN Web Docs: Textarea, ContentEditable, File API, MediaRecorder API

### 11.3 Open Questions
1. Should we support message reactions (emoji reactions)? → Future enhancement
2. Maximum number of simultaneous file uploads? → Proposed: 10 files
3. Support for animated GIFs in addition to static images? → Yes, covered under image support
4. Should we compress images on client-side before upload? → Recommended for images >2MB to optimize upload time while staying within 10MB limit
5. Handling of very large videos (>100MB)? → Reject with clear error message, suggest compression

---

## 12. Contact and Feedback

For questions, suggestions, or feedback regarding this PRD, please contact:
- Product Manager: [Contact information will be added when team assignments are finalized]
- Engineering Lead: [Contact information will be added when team assignments are finalized]
- Design Lead: [Contact information will be added when team assignments are finalized]

**Document Version**: 1.0  
**Last Updated**: 2025-12-19  
**Status**: Draft → Pending Review → Approved
