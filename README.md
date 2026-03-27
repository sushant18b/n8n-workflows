# A/B Testing Chatbot

A sophisticated A/B testing chatbot system integrated with Telegram that manages user onboarding through multiple test groups using n8n workflow automation.

---

## 📋 Overview

This workflow runs an **A/B test** where users are split into different groups and receive different onboarding experiences. It tracks user interactions, logs events, and analyzes the effectiveness of different conversational flows.

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

## 🔐 Credentials Used

- **Telegram API**: Bot authentication for message handling
- **Supabase API**: Database access credentials
- **Statsig API Key**: A/B testing and experiment configuration

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