# Implementation Plan for StarLabs Discord Bot Enhancements

Based on my analysis of the existing codebase and your requirements, here's a comprehensive implementation plan for enhancing the Discord bot:

## 1. Message Filtering System

### Current Implementation

- Basic filtering exists (empty messages and level-up notifications are filtered)
- No comprehensive filtering for spam, short messages, or meaningless content

### Implementation Plan

1. **Create a new module**: `src/model/discord/message_filter.py`

   - Implement functions to filter messages based on:
     - Length (configurable minimum character count)
     - Content quality (blacklist/whitelist of words)
     - User IDs (whitelist/blacklist)
     - Message type (exclude system messages, etc.)

2. **Add configuration options** in `config.yaml`:

```yaml
MESSAGE_FILTER:
  ENABLED: true
  MIN_MESSAGE_LENGTH: 5
  BLACKLISTED_WORDS: ["lol", "ok", "yes", "no", "haha"]
  WHITELISTED_USERS: []
  BLACKLISTED_USERS: []
  ALWAYS_RESPOND_TO_QUESTIONS: true
```

3. **Integrate with `DiscordChatter` class**:
   - Modify the message processing logic in `start_chatting()` to use the new filter
   - Add a check before processing messages to determine if they're worth responding to

## 2. Enhanced Human-like Responses

### Current Implementation

- Basic system prompts exist for casual Discord chat style
- Limited personality or communication style options
- No emoji insertion in responses
- No randomization of greetings/phrases

### Implementation Plan

1. **Expand prompt system** in `src/model/gpt/prompts.py`:

   - Create multiple personality templates (friendly, humorous, serious, etc.)
   - Add more varied examples for each communication style
   - Include instructions for natural emoji usage

2. **Add configuration for communication styles**:

```yaml
CHAT_STYLE:
  MODE: "friendly" # Options: friendly, humorous, serious, neutral
  EMOJI_FREQUENCY: 30 # Percentage chance of including emojis
  TYPING_MISTAKES: 10 # Percentage chance of including minor typos
```

3. **Implement response post-processing**:

   - Create a function to add random greetings/endings based on selected style
   - Add natural pauses (using Discord's typing indicators)
   - Insert emojis at appropriate places

4. **Create a library of phrases** in `data/phrases/`:
   - `greetings.txt` - Various ways to start a message
   - `transitions.txt` - Phrases to connect thoughts
   - `endings.txt` - Ways to end messages
   - `reactions.txt` - Emotional reactions to use

## 3. User-Specific Communication

### Current Implementation

- Bot can respond to replies to its messages
- No specific targeting of individual users
- No custom message files per user

### Implementation Plan

1. **Create user-specific message system**:

   - Add a directory structure: `data/user_messages/{user_id}.txt`
   - Implement a message loader that reads from these files

2. **Add configuration options**:

```yaml
USER_SPECIFIC:
  ENABLED: true
  TARGET_USERS: ["user_id_1", "user_id_2"]
  DEFAULT_RESPONSE_CHANCE: 70 # Percentage chance to respond to targeted users
  MESSAGE_DIRECTORY: "data/user_messages/"
```

3. **Modify `DiscordChatter` class**:

   - Add logic to check if a message is from a targeted user
   - Implement priority response system for targeted users
   - Add ability to use custom messages from files

4. **Create a user tracking system**:
   - Track when the bot last responded to specific users
   - Implement natural conversation patterns (don't respond to every message)

## 4. Enhanced Emoji Reactions

### Current Implementation

- Basic reaction functionality exists
- Reactions are applied to specific message IDs
- No natural timing or filtering

### Implementation Plan

1. **Create a new module**: `src/model/discord/smart_reactions.py`

   - Implement functions to:
     - Analyze message content for reaction-worthiness
     - Select appropriate reactions based on message content
     - Apply natural timing between reactions

2. **Add configuration options**:

```yaml
REACTIONS:
  ENABLED: true
  TARGET_CHANNELS: ["channel_id_1", "channel_id_2"]
  REACTION_CHANCE: 30 # Percentage chance to react to eligible messages
  MIN_MESSAGE_LENGTH: 20 # Only react to messages longer than this
  REACTION_DELAY: [3, 15] # Random delay in seconds before reacting
  PREFERRED_REACTIONS: ["❤️", "👍", "😊", "🔥"] # Default reactions to use
```

3. **Implement message scanning**:

   - Create a background task that periodically scans target channels
   - Filter messages based on length and content
   - Apply reactions with natural timing

4. **Add content-aware reactions**:
   - Analyze message sentiment to choose appropriate reactions
   - Use different reactions for questions vs. statements
   - Implement a system to avoid repetitive reaction patterns

## 5. Configuration and Logging Improvements

### Current Implementation

- YAML-based configuration
- Basic logging with loguru
- Limited configuration options for AI behavior

### Implementation Plan

1. **Enhance configuration system**:

   - Add all new configuration options
   - Implement configuration validation
   - Add ability to reload configuration without restart

2. **Improve logging system**:

   - Add detailed logging for all bot actions
   - Create log categories (reactions, messages, errors)
   - Implement log rotation and archiving

3. **Create a statistics tracking system**:
   - Track messages sent, reactions added
   - Monitor response times and patterns
   - Generate reports on bot activity

## Implementation Timeline

1. **Phase 1: Message Filtering and Configuration (1-2 days)**

   - Implement basic message filtering
   - Enhance configuration system
   - Improve logging

2. **Phase 2: Enhanced Responses (2-3 days)**

   - Implement communication styles
   - Add phrase libraries
   - Create response post-processing

3. **Phase 3: User-Specific Communication (1-2 days)**

   - Implement user targeting
   - Create message file system
   - Add priority response logic

4. **Phase 4: Smart Reactions (1-2 days)**

   - Implement reaction timing
   - Add content-aware reactions
   - Create channel scanning system

5. **Phase 5: Testing and Refinement (2-3 days)**
   - Test all features in live environment
   - Refine response patterns
   - Optimize performance

## Technical Considerations

1. **Maintain Existing Architecture**:

   - Keep the current structure of the codebase
   - Ensure backward compatibility
   - Use the existing configuration system

2. **Performance Optimization**:

   - Implement caching for frequently used data
   - Optimize API calls to Discord
   - Ensure efficient use of ChatGPT API

3. **Error Handling**:
   - Implement robust error handling
   - Add automatic recovery mechanisms
   - Ensure graceful degradation if services are unavailable
