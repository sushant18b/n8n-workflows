# A/B Testing Chatbot

A sophisticated A/B testing chatbot system integrated with Telegram that manages user onboarding through multiple test groups using n8n workflow automation.

---

## 📋 Overview

This workflow runs an **A/B test** where users are split into different groups and receive different onboarding experiences. It tracks user interactions, logs events, and analyzes the effectiveness of different conversational flows.

---

## 🚀 Setup Instructions (How to Run Locally)

### Prerequisites
- n8n instance (self-hosted or cloud)
- Telegram Bot Token (from BotFather)
- Supabase project with database set up
- Statsig account with API access

### Step-by-Step Setup

1. **Clone or Import the Workflow**
   - Import this workflow into your n8n instance
   - Access your n8n dashboard and create a new workflow

2. **Set Up Telegram Bot**
   - Create a bot using [Telegram BotFather](https://t.me/botfather)
   - Copy the generated bot token
   - Set up webhook or use polling to receive messages

3. **Configure Supabase**
   - Create two tables: `users` and `events` (schema details below)
   - Generate an API key and anon key from your Supabase project settings
   - Note the project URL

4. **Set Up Statsig**
   - Create an account on [Statsig](https://statsig.com)
   - Create an A/B test experiment for user group assignment
   - Generate API keys for initialization and event logging
   - Get the URLs for group allocation and event logging endpoints

5. **Configure Credentials in n8n**
   - Add Telegram credentials with your bot token
   - Add Supabase credentials with URL and API keys
   - Add Statsig credentials with API key
   - Add custom HTTP nodes with Statsig endpoints

6. **Activate the Workflow**
   - Enable the Telegram trigger node
   - Test with a message to your bot
   - Monitor workflow executions in the n8n dashboard

---

## 🔐 Environment Variables

Create a `.env` file or configure these in your n8n credentials manager:

```env
# Telegram
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_supabase_anon_key_here

# Statsig
STATSIG_API_KEY=your_statsig_api_key_here
STATSIG_INITIALIZE_URL=https://api.statsig.com/v1/initialize
STATSIG_LOG_EVENT_URL=https://api.statsig.com/v1/log_event
```

### Environment Variable Details

| Variable | Description | Example |
|----------|-------------|---------|
| `TELEGRAM_BOT_TOKEN` | Bot authentication token from BotFather | `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11` |
| `SUPABASE_URL` | Your Supabase project URL | `https://abcdefgh.supabase.co` |
| `SUPABASE_ANON_KEY` | Public API key for Supabase | `eyJhbGc...` |
| `STATSIG_API_KEY` | API key for Statsig authentication | `secret_abc123...` |
| `STATSIG_INITIALIZE_URL` | Endpoint to assign users to test/control groups | `https://api.statsig.com/v1/initialize` |
| `STATSIG_LOG_EVENT_URL` | Endpoint to log user events for analysis | `https://api.statsig.com/v1/log_event` |

---

## 🏗️ Architecture Overview

### High-Level Flow

```
Telegram User Message
        ↓
    Telegram Trigger (n8n)
        ↓
    Query User in Supabase
        ↓
    Switch Node (Route Decision)
        ├─→ New User → Initialize Group (Statsig) → Create User → Log Event → Send Welcome
        ├─→ Test Group User → Get Last Event → Log Progression → Send Next Step
        └─→ Control Group User → Send Standard Message
        ↓
    Response Back to User
```

### Key Components

1. **Telegram Trigger**: Entry point that captures incoming user messages
2. **Supabase Query**: Looks up user in database to determine their status
3. **Switch Router**: Routes users to appropriate execution path based on group assignment
4. **Statsig Integration**: Assigns new users to test/control groups and tracks events
5. **Response Handler**: Sends personalized messages based on user group and progression state

---

## 🔧 Tools and Services Used and Why

### **Telegram**
Telegram provides reliable real-time messaging through webhooks and an easy-to-use Bot API. It eliminates the need for custom message infrastructure and handles user authentication seamlessly. Chosen for its simplicity and global user base.

### **Supabase**
Supabase offers a PostgreSQL-based backend with a straightforward REST API and real-time capabilities. It provides reliable data persistence for user profiles and event logs without requiring complex database management. Used for storing user state and tracking user progression through the onboarding flow.

### **Statsig**
Statsig is a feature management and A/B testing platform that handles statistical significance calculations and experiment management out-of-the-box. It provides APIs for group assignment and event tracking, eliminating the need to build custom experiment infrastructure. Chosen to ensure statistically valid A/B test results and built-in analytics dashboards.

### **n8n**
n8n is a low-code workflow automation platform that integrates multiple services without writing backend code. It handles orchestration of API calls, conditional routing, and event logging elegantly through a visual interface. Used to connect all services together and manage the complex multi-step onboarding flow.

---

## 💡 Assumptions and Trade-offs

This project operates with the following key considerations:

**No Explicit Assumptions**: The workflow is designed to work flexibly with any Telegram bot, Supabase database, and Statsig experiment setup without imposing specific business logic assumptions.

**Design Trade-offs**:
- **Synchronous Execution**: All API calls are executed sequentially, prioritizing reliability over maximum throughput
- **Simple Group Assignment**: Uses Statsig's built-in group assignment without custom stratification or user property-based segmentation
- **Event-Driven State**: User progression is tracked through discrete events rather than a continuous state machine, making it easier to analyze but potentially missing nuanced user interactions
- **No Persistence of Errors**: Failed API calls are not automatically retried; manual intervention may be required for reliability

---

## ⏱️ Time Breakdown per Section

| Phase | Duration | Notes |
|-------|----------|-------|
| **Architecture & Design** | 24 hours | Planned workflow structure, data schema, and integration points |
| **Telegram & Supabase Integration** | 4 hours | Relatively straightforward; good documentation available |
| **Statsig API Integration** | 5-6 hours | Limited official documentation; required trial-and-error for endpoint configuration and payload structure |
| **Workflow Implementation** | 4 hours | Translating architecture into n8n nodes and connecting services |
| **Testing & Deployment** | 2 hours | End-to-end testing, credential setup, and workflow activation |
| **Total** | **39-40 hours** | — |

---

## 🔄 n8n Workflow Architecture

### **Entry Point: Telegram Trigger**
- Listens for incoming Telegram messages
- Captures user data (ID, username, message content)
- Initiates the workflow for each message

### **Core Logic Nodes**

#### 1. **Get a row** (User Lookup)
- Queries Supabase `User` table to check if user exists
- Uses Telegram user ID as the lookup key
- Always outputs data (even if no match found)

#### 2. **Switch Node** (A/B Test Router)
Routes users based on three conditions:
- **Condition 1**: New user (empty `user_id`) → Initialization path
- **Condition 2**: User in "Test" group → Test onboarding path
- **Condition 3**: User in "Control" group → Control onboarding path

---

## 🚀 Execution Paths

### **Path 1: New User (Initialization)**
1. `Initialize Group` → Calls Statsig API to assign user to test/control group
2. `Create User` → Saves new user to Supabase with assigned group and metadata
3. `Log Event` → Logs "new_user" event to Statsig for tracking
4. `Create Event` → Saves event record to Supabase database
5. `Send a text message` → Sends conditional welcome based on group assignment

### **Path 2: Existing "Test" Group User**
1. `Latest Event of User` → Retrieves user's last event from Supabase
2. `Log Event 1` → Logs progression event to Statsig
3. `Create Event 1` → Creates next progression event in Supabase
4. `Send a text message2` → Sends next onboarding step with dynamic content

**Dynamic Message Logic:**
- `new_user` → "Let's get started. What are you interested in?"
- `onboarding_user` → "Which category do you prefer?"
- `onboarding_step_2` → "You're all set! You can now start using the bot!"

### **Path 3: Existing "Control" Group User**
1. `Send a text message1` → Sends standard welcome message

---

## 🔗 Data Integrations

| Service | Purpose |
|---------|---------|
| **Telegram** | User messaging and notifications |
| **Supabase** | Stores users, events, and user state |
| **Statsig** | A/B testing framework - assigns groups & tracks experiments |

---

## 📊 Key Features

✅ **A/B Test Experiment Tracking** - Assign users to test/control groups  
✅ **User State Management** - Track user lifecycle and progression  
✅ **Multi-step Onboarding Flow** - Dynamic conversation paths  
✅ **Event Logging** - Comprehensive event tracking for analysis  
✅ **Group-specific Messaging** - Different messages for different variants  
✅ **Real-time Analytics** - Monitor experiment performance  

---

## 🗄️ Database Schema

### Users Table
- `user_id`: Telegram user ID (primary key)
- `username`: Telegram username
- `message`: User's first message
- `group`: Test assignment (Test/Control)

### Events Table
- `user_id`: Reference to user
- `event_name`: Event type (new_user, onboarding_user, onboarding_step_2, onboarding_completed)
- `created_at`: Timestamp of event

---

## 📈 Workflow Status

- **Active**: false (currently inactive)
- **Execution Order**: v1
- **Binary Mode**: separate

---

## 🎯 Use Cases

- Testing different onboarding messages for user engagement
- Analyzing which conversation flow converts better
- A/B testing product recommendations
- Optimizing user retention through targeted messaging
