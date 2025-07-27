# StarLabs Discord Automation Tool - AI Documentation

## Introduction

StarLabs Discord is a powerful and flexible Discord automation tool designed to manage multiple Discord accounts, automate interactions, and provide AI-powered chat capabilities. This tool allows users to perform various Discord-related tasks programmatically, including server management, message sending, profile customization, and AI-driven conversations.

The tool is built with multi-threading capabilities, allowing for parallel processing of multiple accounts simultaneously. It also includes features like automatic retries, proxy support, and secure file handling to ensure reliable and efficient operation.

## Project Architecture

### High-Level Overview

The StarLabs Discord automation tool follows a modular architecture with clear separation of concerns:

1. **Entry Point**: `main.py` serves as the entry point, setting up logging and initiating the main process.

2. **Process Management**: `process.py` handles the main processing logic, including account selection, threading, and task execution.

3. **Core Modules**:

   - `src/model/discord/`: Contains Discord-specific functionality
   - `src/model/gpt/`: Handles GPT integration for AI chat capabilities
   - `src/utils/`: Provides utility functions for configuration, file handling, etc.

4. **Data Management**:
   - `data/accounts.xlsx`: Stores Discord account information
   - `data/messages/`: Contains message templates
   - `data/pictures/`: Stores profile pictures

### Component Breakdown

#### Discord Module (`src/model/discord/`)

This module contains various components for interacting with Discord:

- **Account Management**:

  - `account_editor.py`: Handles profile customization (name, username, password, profile picture)
  - `token_checker.py`: Verifies Discord tokens and checks account status

- **Server Interaction**:

  - `get_all_servers.py`: Retrieves server information
  - `inviter.py`: Handles server invitations
  - `leave_guild.py`: Manages leaving Discord servers

- **Chat Functionality**:

  - `chatter.py`: Manages AI-powered chat interactions
  - `send_chat_message.py`: Handles sending messages to Discord channels

- **UI Interaction**:

  - `button_presser.py`: Automates button interactions
  - `reaction_presser.py`: Manages emoji reactions

- **Captcha Handling**:
  - `captcha/capsolver.py`: Integrates with captcha solving services
  - `captcha/nocaptcha.py`: Handles captcha bypass techniques

#### GPT Module (`src/model/gpt/`)

This module handles the integration with OpenAI's GPT models:

- `gpt.py`: Core functionality for interacting with OpenAI API
- `prompts.py`: Contains system prompts for different chat scenarios

#### Utilities (`src/utils/`)

Provides various utility functions:

- `config.py`: Handles configuration loading and management
- `constants.py`: Defines constants used throughout the application
- `reader.py`: Handles file reading operations
- `writer.py`: Manages file writing operations
- `output.py`: Handles console output formatting
- `check_github_version.py`: Checks for updates from GitHub

## Key Features

### Multi-Threading and Parallel Processing

The tool supports concurrent execution of tasks across multiple accounts:

```python
# From process.py
semaphore = asyncio.Semaphore(value=threads)
tasks = []

# Create tasks for each account
for account in shuffled_accounts:
    tasks.append(asyncio.create_task(launch_wrapper(account)))

await asyncio.gather(*tasks)
```

### Automatic Retries

The tool includes a robust retry mechanism for handling failures:

```python
# From process.py
async def wrapper(function, config: src.utils.config.Config, *args, **kwargs):
    attempts = config.SETTINGS.ATTEMPTS
    for attempt in range(attempts):
        result = await function(*args, **kwargs)
        if isinstance(result, tuple) and result and isinstance(result[0], bool):
            if result[0]:
                return result
        elif isinstance(result, bool):
            if result:
                return True

        if attempt < attempts - 1:  # Don't sleep after the last attempt
            pause = random.randint(
                config.SETTINGS.PAUSE_BETWEEN_ATTEMPTS[0],
                config.SETTINGS.PAUSE_BETWEEN_ATTEMPTS[1],
            )
            logger.info(
                f"Sleeping for {pause} seconds before next attempt {attempt+1}/{config.SETTINGS.ATTEMPTS}..."
            )
            await asyncio.sleep(pause)

    return result
```

### AI Chat Integration

The tool integrates with OpenAI's GPT models to generate human-like chat messages:

```python
# From src/model/gpt/gpt.py
def ask_chatgpt(api_key: str, model: str, user_message: str, prompt: str, proxy: str = "") -> tuple[bool, str]:
    """
    Send a message to ChatGPT and get a response.
    """
    # ... implementation details ...
```

The AI chat functionality is configured to mimic casual Discord users:

```python
# From src/model/gpt/prompts.py
BATCH_MESSAGES_SYSTEM_PROMPT = """You are a casual Discord chat participant in your teens/early 20s. Your task is to analyze chat messages and contribute ONE natural message.

Rules:
- Send only ONE message, 3-15 words long
- Look at the general topic being discussed
- Add a single relevant thought to the conversation
...
"""
```

### Account Management

The tool provides comprehensive account management features:

```python
# From src/model/discord/account_editor.py
async def change_name(self) -> bool:
    # ... implementation details ...

async def change_username(self) -> bool:
    # ... implementation details ...

async def change_password(self) -> bool:
    # ... implementation details ...

async def change_profile_picture(self) -> bool:
    # ... implementation details ...
```

### Token Verification

The tool includes functionality to verify Discord tokens and check account status:

```python
# From src/model/discord/token_checker.py
async def token_checker(account: Account, config: Config, client: AsyncSession) -> tuple[bool, str] | bool:
    # ... implementation details ...
```

## Installation and Setup

### Prerequisites

- Python 3.11.6 or higher
- Excel file with Discord accounts
- Valid Discord tokens
- (Optional) Proxies
- OpenAI API key for AI Chat features

### Installation Steps

1. Clone the repository:

```bash
git clone https://github.com/0xStarLabs/StarLabs-Discord.git
cd StarLabs-Discord
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Configure your settings in `config.yaml`

### Configuration

#### accounts.xlsx Structure

| Column        | Example   | Description              |
| ------------- | --------- | ------------------------ |
| DISCORD_TOKEN | token1    | Discord account token    |
| PROXY         | proxy1    | Proxy address (optional) |
| USERNAME      | user1     | Account username         |
| STATUS        | VALID     | Account status           |
| PASSWORD      | pass1     | Current password         |
| NEW_PASSWORD  | newpass1  | New password for update  |
| NEW_NAME      | name1     | New display name         |
| NEW_USERNAME  | username1 | New username             |
| MESSAGES_FILE | messages1 | Custom messages file     |

#### config.yaml Settings

The `config.yaml` file contains various configuration options:

```yaml
SETTINGS:
  # discord token use for press reaction task etc, for getting emojis info
  DISCORD_TOKEN_FOR_PARSING: ""
  # proxy for parsing. format: username:password@ip:port
  PROXY_FOR_PARSING: ""

  # number of concurrent threads
  THREADS: 1

  # number of retries for ANY action
  ATTEMPTS: 5

  # shuffle accounts in random order
  SHUFFLE_ACCOUNTS: true

  # account range.
  # BY DEFAULT: [0, 0] - all accounts
  # [3, 5] - only 3 4 5 accounts
  # [7, 7] - only 7 account
  ACCOUNTS_RANGE: [0, 0]

  # ... other settings ...
```

## Usage Guide

### Basic Usage

1. Prepare your files:

   - Fill `accounts.xlsx` with tokens and account data
   - Configure `config.yaml` with desired settings
   - Add message templates to `data/messages/`
   - Add profile pictures to `data/pictures/`

2. Run the bot:

```bash
python main.py
```

3. Select a task from the menu that appears

### Available Tasks

The tool provides various tasks that can be selected from the menu:

1. **AI Chatter**: Engages in AI-powered conversations in Discord channels
2. **Server Inviter**: Joins Discord servers using invite links
3. **Button Interaction**: Automates clicking buttons in Discord UI
4. **Reaction Management**: Adds reactions to messages
5. **Profile Customization**: Changes name, username, password, or profile picture
6. **Message Management**: Sends messages to Discord channels
7. **Token Verification**: Checks the validity of Discord tokens
8. **Server Management**: Leaves servers, lists servers, or checks presence in servers

### AI Chat Configuration

The AI chat functionality can be configured in the `config.yaml` file:

```yaml
AI_CHATTER:
  # discord server id where bot will send messages
  GUILD_ID: ""

  # discord channel id where bot will send messages
  CHANNEL_ID: ""

  # if a bot is asked something, what is the probability that it will answer or ignore the question
  ANSWER_PERCENTAGE: 50

  # How many percent of texts will be REPLY to other users' texts
  # the rest of the sms will just be sent to the chat.
  # if 0, the bot will answer to no one
  # if 100, the bot will send only replies to other users' sms.
  REPLY_PERCENTAGE: 50

  # number of messages to send
  MESSAGES_TO_SEND_PER_ACCOUNT: [3, 5]

  # ... other settings ...
```

## AI Integration Details

### OpenAI API Integration

The tool integrates with OpenAI's API to generate human-like chat messages:

```python
# From src/model/gpt/gpt.py
def ask_chatgpt(api_key: str, model: str, user_message: str, prompt: str, proxy: str = "") -> tuple[bool, str]:
    """
    Send a message to ChatGPT and get a response.
    """
    # ... implementation details ...
```

### System Prompts

The tool uses carefully crafted system prompts to guide the AI's responses:

```python
# From src/model/gpt/prompts.py
REFERENCED_MESSAGES_SYSTEM_PROMPT = """You are a casual Discord chat participant in your teens/early 20s. Your task is to continue the conversation naturally based on the previous messages.

Rules:
- Keep responses between 3-15 words
- Stay in context of the conversation
- Be super casual and informal, like a real Discord user
- Sometimes start sentences with lowercase
- Use minimal punctuation (occasional commas are ok)
- Use slang sparingly (only use "lol", "ngl", "fr", "tbh" in about 20% of messages)
- Don't overuse exclamation marks
- Never mention that you're an AI
- It's ok to make small typos sometimes
- Mix up your style - don't be repetitive with phrases

# ... more details ...
```

### Chat Strategies

The tool implements different chat strategies:

1. **Reply to Mentions**: Responds to messages that mention the bot's account
2. **Reply to Conversations**: Joins existing conversations by replying to messages
3. **New Message Generation**: Starts new conversations based on chat history

## Troubleshooting and Best Practices

### Common Issues

1. **Token Validation Failures**:

   - Ensure Discord tokens are valid and not expired
   - Check if accounts are locked or quarantined

2. **Rate Limiting**:

   - Increase pause times between actions
   - Use more proxies to distribute requests

3. **Captcha Challenges**:
   - Configure captcha solving services properly
   - Reduce suspicious activity patterns

### Best Practices

1. **Account Management**:

   - Rotate accounts regularly
   - Use different proxies for each account
   - Maintain natural usage patterns

2. **AI Chat Configuration**:

   - Set reasonable message frequencies
   - Balance reply percentages for natural interaction
   - Customize prompts for specific use cases

3. **Proxy Usage**:
   - Use high-quality proxies
   - Distribute accounts across different proxy regions
   - Monitor proxy performance

## Security Considerations

### Token Security

Discord tokens provide full access to accounts. Ensure:

1. Tokens are stored securely
2. Access to the `accounts.xlsx` file is restricted
3. Tokens are not exposed in logs or shared environments

### API Key Management

OpenAI API keys should be protected:

1. Do not hardcode API keys in the configuration
2. Rotate API keys periodically
3. Monitor API usage for unauthorized access

## Legal and Ethical Considerations

This tool is provided for educational purposes only. Users should:

1. Comply with Discord's Terms of Service
2. Respect community guidelines and server rules
3. Obtain proper authorization before automating actions on accounts
4. Use the tool responsibly and ethically

## Conclusion

The StarLabs Discord automation tool provides a powerful framework for automating Discord interactions with AI capabilities. By following the guidelines in this documentation, users can effectively leverage the tool's features while maintaining security and compliance with platform policies.

Remember that automation tools should be used responsibly and in accordance with Discord's Terms of Service. The developers of this tool are not responsible for any misuse or violations resulting from its use.

---

_Disclaimer: This tool is for educational purposes only. Use at your own risk and in accordance with Discord's Terms of Service._
