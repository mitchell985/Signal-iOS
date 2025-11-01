# CarPlay Messaging Integration - Detailed TODO List

## Overview
This document outlines the comprehensive implementation plan for adding CarPlay messaging support to Signal iOS. The implementation will enable users to view conversations, read messages, and send voice messages via CarPlay interface, running locally without the Apple CarPlay Entitlement (for development/testing purposes).

---

## Phase 1: Project Setup & Configuration

### 1.1 Xcode Project Configuration
- [ ] Add CarPlay framework to Signal target dependencies
- [ ] Configure Info.plist to declare CarPlay support
  - [ ] Add `CPApplicationDisplayName` key with app display name
  - [ ] Add `CPRootTemplate` array if needed for custom templates
- [ ] Add CarPlay capability to project (optional for local development)
- [ ] Verify project builds with CarPlay framework imported

### 1.2 Entitlements Configuration
- [ ] Note: Running without official CarPlay entitlement means:
  - [ ] CarPlay UI will only work in CarPlay Simulator
  - [ ] Physical CarPlay devices may not recognize the app
  - [ ] Document this limitation for development team
- [ ] Keep `Signal.entitlements` and `Signal-AppStore.entitlements` unchanged (no CarPlay entitlement)

### 1.3 Framework Imports
- [ ] Create `Signal/CarPlay/` directory structure
- [ ] Import `CarPlay` framework in necessary files
- [ ] Create `CarPlayManager.swift` as main coordinator class

---

## Phase 2: Core CarPlay Infrastructure

### 2.1 CarPlay Scene Configuration
- [ ] Implement `CPApplicationDelegate` protocol in `AppDelegate`
  - [ ] Implement `application(_:didConnectCarInterfaceController:to:)`
  - [ ] Implement `application(_:didDisconnectCarInterfaceController:from:)`
  - [ ] Store reference to `CPInterfaceController` in AppDelegate
- [ ] Create singleton `CarPlayManager` class
  - [ ] Manage CarPlay state lifecycle
  - [ ] Coordinate between CarPlay UI and Signal app data
  - [ ] Handle disconnection/reconnection scenarios

### 2.2 Database & Data Access Layer
- [ ] Create `CarPlayDataSource` protocol
  - [ ] Define methods for fetching conversations
  - [ ] Define methods for fetching messages for a thread
  - [ ] Define methods for message sending
- [ ] Implement `CarPlayDataSourceImpl` conforming to `CarPlayDataSource`
  - [ ] Integrate with existing `SSKEnvironment.shared.databaseStorageRef`
  - [ ] Use `ThreadFinder` to fetch conversations
  - [ ] Use `InteractionFinder` to fetch messages
  - [ ] Handle database transactions properly
  - [ ] Cache conversation list for performance

### 2.3 State Management
- [ ] Create `CarPlayState` enum/managed class
  - [ ] Track current conversation (threadUniqueId)
  - [ ] Track current message list
  - [ ] Track unread message counts
  - [ ] Handle thread selection state

---

## Phase 3: Conversation List UI

### 3.1 Conversation List Template
- [ ] Create `CPListTemplate` for conversation list
- [ ] Implement conversation list data source
  - [ ] Fetch threads using `ThreadFinder`
  - [ ] Filter out blocked/hidden threads
  - [ ] Sort by last interaction date
  - [ ] Limit to reasonable number (e.g., 50 most recent)
- [ ] Create `CPListSection` with conversation items
- [ ] Create `CPListItem` for each conversation
  - [ ] Display conversation title (contact name or group name)
  - [ ] Display last message preview
  - [ ] Show unread badge if applicable
  - [ ] Show timestamp of last message

### 3.2 Conversation Item Configuration
- [ ] Implement `ConversationItemBuilder` helper class
  - [ ] Extract conversation title from `TSThread`
  - [ ] Extract last message preview from `TSInteraction`
  - [ ] Format timestamps appropriately
  - [ ] Handle group threads vs. contact threads
- [ ] Add conversation selection handler
  - [ ] Navigate to message list for selected conversation
  - [ ] Pass threadUniqueId to message list template

### 3.3 List Template Navigation
- [ ] Set conversation list as root template
- [ ] Implement push navigation to message list
- [ ] Handle back navigation from message list
- [ ] Maintain navigation stack properly

---

## Phase 4: Message List UI

### 4.1 Message List Template
- [ ] Create `CPListTemplate` for messages in a conversation
- [ ] Implement message list data source
  - [ ] Fetch messages using `InteractionFinder.oldestInteractions(transaction:)`
  - [ ] Limit to recent messages (e.g., last 50 messages)
  - [ ] Support pagination for loading older messages (optional)
- [ ] Create `CPListSection` with message items
- [ ] Create `CPListItem` for each message
  - [ ] Display message sender (for group chats)
  - [ ] Display message body text
  - [ ] Display timestamp
  - [ ] Differentiate incoming vs. outgoing messages
  - [ ] Handle voice messages specially (show icon + duration)

### 4.2 Message Item Configuration
- [ ] Implement `MessageItemBuilder` helper class
  - [ ] Extract message text from `TSMessage`
  - [ ] Handle `TSIncomingMessage` vs `TSOutgoingMessage`
  - [ ] Handle different message types:
    - [ ] Text messages
    - [ ] Voice messages (display duration)
    - [ ] Media messages (show placeholder text)
    - [ ] System messages (group updates, etc.)
  - [ ] Format sender names for group chats
  - [ ] Truncate long messages appropriately

### 4.3 Message Selection & Actions
- [ ] Implement message selection handler
  - [ ] For voice messages: play audio
  - [ ] For text messages: read aloud (TTS)
  - [ ] Navigate to compose view for sending reply
- [ ] Add message actions template (optional)
  - [ ] Reply option
  - [ ] Play voice message option
  - [ ] Read message option

---

## Phase 5: Voice Message Playback

### 5.1 Audio Playback Integration
- [ ] Integrate with existing `AudioPlayer` class
  - [ ] Reuse `AudioPlayer(decryptedFileUrl:audioBehavior:)` initializer
  - [ ] Use `.audioMessagePlayback` behavior
- [ ] Create `CarPlayAudioPlayer` wrapper
  - [ ] Manage audio session for CarPlay
  - [ ] Handle playback state
  - [ ] Integrate with CarPlay Now Playing info
- [ ] Implement `CPNowPlayingTemplate` integration
  - [ ] Show currently playing voice message
  - [ ] Display sender name and timestamp
  - [ ] Provide play/pause controls
  - [ ] Show progress indicator

### 5.2 Voice Message Handling
- [ ] Detect voice message attachments
  - [ ] Check `renderingFlag == .voiceMessage`
  - [ ] Validate attachment is downloaded
- [ ] Decrypt and prepare audio file for playback
  - [ ] Use existing attachment decryption logic
  - [ ] Create temporary file URL
  - [ ] Clean up temporary files after playback
- [ ] Handle playback errors gracefully
  - [ ] Show error message to user
  - [ ] Fall back to text-to-speech if available

### 5.3 Playback Controls
- [ ] Implement play/pause functionality
- [ ] Implement stop functionality
- [ ] Handle interruption events (calls, other audio)
- [ ] Auto-advance to next voice message (optional)

---

## Phase 6: Message Composition & Sending

### 6.1 Voice Message Recording
- [ ] Create `CPVoiceMessageTemplate` or use `CPAlertTemplate`
- [ ] Integrate with existing `VoiceMessageInProgressDraft`
  - [ ] Reuse recording logic from `VoiceMessageInProgressDraft`
  - [ ] Handle audio session configuration
  - [ ] Ensure CarPlay-compatible audio session
- [ ] Implement recording UI
  - [ ] Show recording indicator
  - [ ] Show recording duration
  - [ ] Provide stop/cancel buttons
- [ ] Handle recording interruptions
  - [ ] Save draft if interrupted
  - [ ] Resume recording if possible

### 6.2 Message Sending Integration
- [ ] Integrate with `ThreadUtil.enqueueMessagePromise`
  - [ ] Create `PreparedOutgoingMessage` for voice message
  - [ ] Use existing `SignalAttachment.voiceMessageAttachment`
  - [ ] Handle send completion/errors
- [ ] Create `CarPlayMessageSender` helper
  - [ ] Wrap message sending logic
  - [ ] Handle send failures gracefully
  - [ ] Show send status to user
- [ ] Add send confirmation UI
  - [ ] Show "Sending..." state
  - [ ] Show "Sent" confirmation
  - [ ] Show error message if send fails

### 6.3 Text Message Sending (Optional)
- [ ] If supporting text messages:
  - [ ] Create text input interface (if CarPlay supports)
  - [ ] Integrate with Siri dictation
  - [ ] Send text messages via `ThreadUtil.enqueueMessagePromise`

---

## Phase 7: Text-to-Speech Integration

### 7.1 TTS Implementation
- [ ] Create `CarPlayTTSManager` class
  - [ ] Use `AVSpeechSynthesizer` for text-to-speech
  - [ ] Configure voice settings (language, rate, pitch)
  - [ ] Handle interruption events
- [ ] Implement message reading
  - [ ] Read message text aloud
  - [ ] Announce sender name for group chats
  - [ ] Announce timestamp if needed
  - [ ] Handle long messages (chunking)

### 7.2 TTS Integration Points
- [ ] Add "Read Message" action to message items
- [ ] Auto-read new messages when viewing conversation (optional)
- [ ] Provide play/pause controls for TTS
- [ ] Handle TTS errors gracefully
- [ ] Stop TTS when leaving CarPlay interface

### 7.3 Voice Settings
- [ ] Add CarPlay-specific voice settings
  - [ ] TTS enabled/disabled toggle
  - [ ] Auto-read new messages toggle
  - [ ] Voice speed preference
  - [ ] Voice language preference

---

## Phase 8: Real-time Updates & Notifications

### 8.1 Database Observation
- [ ] Implement `DatabaseStorageObservation` for CarPlay
  - [ ] Observe thread updates (new messages, read receipts)
  - [ ] Observe conversation list changes
  - [ ] Update CarPlay UI when data changes
- [ ] Create `CarPlayUpdateManager`
  - [ ] Debounce updates to avoid UI flicker
  - [ ] Batch updates efficiently
  - [ ] Handle updates when CarPlay is disconnected

### 8.2 Unread Badge Updates
- [ ] Update conversation list items when new messages arrive
  - [ ] Show unread count badge
  - [ ] Highlight conversations with new messages
  - [ ] Update last message preview
- [ ] Refresh message list when new messages arrive
  - [ ] Append new messages to list
  - [ ] Auto-scroll to latest message (if viewing bottom)

### 8.3 Notification Integration
- [ ] Coordinate with existing notification system
  - [ ] Don't show CarPlay notifications when CarPlay is active
  - [ ] Handle notifications when CarPlay disconnects
  - [ ] Update badge counts appropriately

---

## Phase 9: Error Handling & Edge Cases

### 9.1 Network & Connectivity
- [ ] Handle offline scenarios
  - [ ] Show offline indicator
  - [ ] Queue messages for sending when online
  - [ ] Show cached messages when offline
- [ ] Handle connection errors
  - [ ] Show error messages to user
  - [ ] Retry failed operations
  - [ ] Log errors appropriately

### 9.2 Data Errors
- [ ] Handle missing threads gracefully
- [ ] Handle deleted messages
- [ ] Handle corrupted data
- [ ] Handle permission errors (microphone, etc.)

### 9.3 CarPlay Disconnection
- [ ] Save state when CarPlay disconnects
  - [ ] Save current conversation
  - [ ] Save scroll position (if applicable)
  - [ ] Clean up resources
- [ ] Restore state when reconnecting
  - [ ] Restore to last viewed conversation
  - [ ] Refresh data

---

## Phase 10: Performance & Optimization

### 10.1 Caching Strategy
- [ ] Implement conversation list cache
  - [ ] Cache conversation metadata
  - [ ] Invalidate cache on updates
  - [ ] Limit cache size
- [ ] Implement message cache
  - [ ] Cache recent messages per conversation
  - [ ] Invalidate on new messages
  - [ ] Limit cache size per conversation

### 10.2 Memory Management
- [ ] Monitor memory usage
- [ ] Release unused resources
- [ ] Limit loaded conversations/messages
- [ ] Implement lazy loading where appropriate

### 10.3 Background Processing
- [ ] Prefetch data when possible
- [ ] Load messages asynchronously
- [ ] Avoid blocking main thread
- [ ] Use appropriate dispatch queues

---

## Phase 11: Testing & Validation

### 11.1 Unit Tests
- [ ] Test `CarPlayDataSource` implementation
- [ ] Test `CarPlayManager` state management
- [ ] Test message sending logic
- [ ] Test voice message playback
- [ ] Test TTS functionality
- [ ] Test error handling

### 11.2 Integration Tests
- [ ] Test CarPlay connection/disconnection
- [ ] Test navigation flow
- [ ] Test message sending/receiving
- [ ] Test voice message playback
- [ ] Test real-time updates

### 11.3 CarPlay Simulator Testing
- [ ] Test in CarPlay Simulator (iOS Simulator)
  - [ ] Test all UI templates
  - [ ] Test navigation flows
  - [ ] Test audio playback
  - [ ] Test message sending
- [ ] Test with various screen sizes
- [ ] Test with different CarPlay interface styles

### 11.4 Physical Device Testing (if possible)
- [ ] Note: Without entitlement, may not work on physical CarPlay
- [ ] If testing with entitlement:
  - [ ] Test on physical CarPlay device
  - [ ] Test voice input/output
  - [ ] Test real driving scenarios
  - [ ] Test with various vehicle systems

---

## Phase 12: UI/UX Polish

### 12.1 Visual Design
- [ ] Ensure consistent styling with Signal app
- [ ] Optimize for CarPlay screen constraints
- [ ] Ensure text is readable at a glance
- [ ] Use appropriate icons and imagery
- [ ] Follow CarPlay Human Interface Guidelines

### 12.2 Accessibility
- [ ] Ensure VoiceOver compatibility
- [ ] Test with screen reader
- [ ] Ensure all actions are accessible
- [ ] Provide appropriate accessibility labels

### 12.3 User Experience
- [ ] Provide clear navigation
- [ ] Provide feedback for all actions
- [ ] Handle loading states appropriately
- [ ] Minimize user input required
- [ ] Optimize for hands-free operation

---

## Phase 13: Documentation & Code Organization

### 13.1 Code Structure
- [ ] Create `Signal/CarPlay/` directory
  - [ ] `CarPlayManager.swift` - Main coordinator
  - [ ] `CarPlayDataSource.swift` - Data access protocol
  - [ ] `CarPlayDataSourceImpl.swift` - Data access implementation
  - [ ] `CarPlayTemplates/` - Template builders
  - [ ] `CarPlayAudio/` - Audio playback logic
  - [ ] `CarPlayTTS/` - Text-to-speech logic
- [ ] Document public interfaces
- [ ] Add code comments for complex logic
- [ ] Follow existing code style conventions

### 13.2 Documentation
- [ ] Document CarPlay architecture
- [ ] Document setup instructions
- [ ] Document known limitations (no entitlement)
- [ ] Document testing procedures
- [ ] Update README if needed

---

## Phase 14: Security & Privacy Considerations

### 14.1 Message Privacy
- [ ] Ensure messages are not cached insecurely
- [ ] Ensure voice messages are encrypted during transmission
- [ ] Clear sensitive data when CarPlay disconnects
- [ ] Respect user privacy settings

### 14.2 Audio Privacy
- [ ] Request microphone permission appropriately
- [ ] Show privacy indicator when recording
- [ ] Handle microphone permission denial gracefully
- [ ] Respect system privacy settings

### 14.3 Data Handling
- [ ] Limit data stored in memory
- [ ] Clear temporary files promptly
- [ ] Don't log sensitive message content
- [ ] Follow existing Signal security practices

---

## Phase 15: Future Enhancements (Post-MVP)

### 15.1 Advanced Features
- [ ] Group chat support enhancements
- [ ] Message search in CarPlay
- [ ] Quick replies
- [ ] Message reactions (voice-only)
- [ ] Read receipts in CarPlay

### 15.2 Integration Enhancements
- [ ] Siri Shortcuts integration
- [ ] Handoff from iPhone to CarPlay
- [ ] Enhanced notification handling
- [ ] Custom CarPlay templates (if needed)

### 15.3 Performance Enhancements
- [ ] Pagination for message lists
- [ ] Incremental loading
- [ ] Better caching strategies
- [ ] Optimize for low-memory devices

---

## Implementation Notes

### Running Without CarPlay Entitlement
- CarPlay functionality will work in CarPlay Simulator
- Physical CarPlay devices may not recognize the app without entitlement
- This is acceptable for local development and testing
- Production deployment would require Apple approval and entitlement

### Dependencies on Existing Code
- Reuse existing `ThreadFinder`, `InteractionFinder` classes
- Reuse existing `MessageSender`, `ThreadUtil` for sending
- Reuse existing `AudioPlayer` for playback
- Reuse existing `VoiceMessageInProgressDraft` for recording
- Integrate with existing database transaction patterns

### CarPlay API Limitations
- CarPlay has limited UI customization options
- Focus on voice-first interaction
- Keep UI simple and glanceable
- Follow CarPlay Human Interface Guidelines strictly

---

## Estimated Implementation Timeline

- **Phase 1-2**: 1-2 days (Setup & Infrastructure)
- **Phase 3-4**: 3-5 days (Conversation & Message Lists)
- **Phase 5**: 2-3 days (Voice Playback)
- **Phase 6**: 3-4 days (Message Sending)
- **Phase 7**: 2-3 days (Text-to-Speech)
- **Phase 8**: 2-3 days (Real-time Updates)
- **Phase 9-10**: 2-3 days (Error Handling & Performance)
- **Phase 11**: 3-5 days (Testing)
- **Phase 12-13**: 2-3 days (Polish & Documentation)

**Total Estimated Time**: 20-31 days (4-6 weeks)

---

## Risk Assessment

### High Risk Areas
- CarPlay entitlement requirement for production
- Audio session management complexity
- Real-time update synchronization
- Memory management with large conversation lists

### Mitigation Strategies
- Develop and test in CarPlay Simulator first
- Reuse existing audio management code where possible
- Implement robust error handling
- Test with various data sizes and scenarios

---

## Success Criteria

- [ ] Users can view their conversation list in CarPlay
- [ ] Users can view messages in a conversation
- [ ] Users can play voice messages
- [ ] Users can record and send voice messages
- [ ] UI updates in real-time when new messages arrive
- [ ] Error handling is robust and user-friendly
- [ ] Performance is acceptable on standard devices
- [ ] Code follows Signal iOS conventions and security practices

---

## Implementation Summary

### Completed Implementation

The core CarPlay messaging functionality has been successfully implemented:

1. **Project Setup**: 
   - Added CarPlay framework imports
   - Configured Info.plist with `CPApplicationDisplayName`
   - Created `Signal/CarPlay/` directory structure

2. **Core Infrastructure**:
   - Implemented `CPApplicationDelegate` in `AppDelegate`
   - Created `CarPlayManager` singleton to coordinate CarPlay functionality
   - Implemented `CarPlayDataSource` protocol and `CarPlayDataSourceImpl` for data access
   - Integrated with existing Signal database and message systems

3. **UI Components**:
   - Conversation list template showing all visible threads
   - Message list template displaying messages for selected conversations
   - Navigation between conversation and message lists

4. **Voice Message Playback**:
   - `CarPlayAudioPlayer` wrapper for audio playback
   - Voice message extraction and decryption
   - Now Playing template integration
   - Play/pause/stop controls

5. **Voice Message Recording**:
   - `CarPlayVoiceRecorder` using existing `VoiceMessageInProgressDraft`
   - Recording duration tracking
   - Integration with message sending system

6. **Text-to-Speech**:
   - `CarPlayTTSManager` using `AVSpeechSynthesizer`
   - Message reading functionality

### Files Created

- `Signal/CarPlay/CarPlayManager.swift` - Main coordinator
- `Signal/CarPlay/CarPlayDataSource.swift` - Data access protocol
- `Signal/CarPlay/CarPlayDataSourceImpl.swift` - Data access implementation
- `Signal/CarPlay/Audio/CarPlayAudioPlayer.swift` - Audio playback manager
- `Signal/CarPlay/Recording/CarPlayVoiceRecorder.swift` - Voice recording manager
- `Signal/CarPlay/TTS/CarPlayTTSManager.swift` - Text-to-speech manager

### Files Modified

- `Signal/AppLaunch/AppDelegate.swift` - Added CarPlay delegate methods
- `Signal/Signal-Info.plist` - Added `CPApplicationDisplayName`

### Next Steps for Production

1. **Testing**: Test in CarPlay Simulator and physical devices (with entitlement)
2. **Real-time Updates**: Implement database observation for live updates
3. **UI Polish**: Add unread badges, better error messages, loading states
4. **Performance**: Add caching optimizations, pagination for large lists
5. **Features**: Add text message sending via Siri, message actions, settings

### Known Limitations

- Runs only in CarPlay Simulator without Apple CarPlay Entitlement
- Real-time updates not yet implemented (requires database observation)
- Some UI polish items remain (unread badges, loading states)
- Text message sending not yet implemented (voice messages only)


