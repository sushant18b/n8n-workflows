# HealthBot - Meal Tracking Chatbot

A sophisticated meal tracking and health monitoring chatbot system integrated with Telegram that helps users log, view, edit, and manage their daily meals using n8n workflow automation.

---

## 📋 Overview

This workflow provides a comprehensive health tracking solution where users can interact with a Telegram bot to manage their daily meal intake. It enables users to log meals, view their meal history, edit meal entries, and delete meals. The system tracks user activity and maintains a persistent record of all meal data.

---

## 🚀 Setup Instructions (How to Run Locally)

### Prerequisites
- n8n instance (self-hosted or cloud)
- Telegram Bot Token (from BotFather)
- Supabase project with database set up
- Active Telegram account

### Step-by-Step Setup

1. **Clone or Import the Workflow**
   - Import this workflow into your n8n instance
   - Access your n8n dashboard and create a new workflow
   - Import the `HealthBot Workflow.json` file

2. **Set Up Telegram Bot**
   - Create a bot using [Telegram BotFather](https://t.me/botfather)
   - Copy the generated bot token
   - Set up webhook or use polling to receive messages

3. **Configure Supabase**
   - Create two tables: `Users` and `Meals` (schema details below)
   - Generate an API key and anon key from your Supabase project settings
   - Note the project URL

4. **Configure Credentials in n8n**
   - Add Telegram credentials with your bot token
   - Add Supabase credentials with URL and API keys
   - Test the connections

5. **Activate the Workflow**
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
```

### Environment Variable Details

| Variable | Description | Example |
|----------|-------------|---------|
| `TELEGRAM_BOT_TOKEN` | Bot authentication token from BotFather | `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11` |
| `SUPABASE_URL` | Your Supabase project URL | `https://abcdefgh.supabase.co` |
| `SUPABASE_ANON_KEY` | Public API key for Supabase | `eyJhbGc...` |

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
        ├─→ New User → Create User → Send Welcome Menu
        ├─→ Log Meal Command → Create Meal Entry → Confirmation
        ├─→ View Meals Command → Retrieve Meals → Display List
        ├─→ Edit Meal Command → Update Meal Entry → Confirmation
        ├─→ Delete Meal Command → Remove Meal → Confirmation
        └─→ Stop Command → Deactivate User
        ↓
    Response Back to User
```

### Key Components

1. **Telegram Trigger**: Entry point that captures incoming user messages
2. **Supabase Query**: Looks up user in database to determine their status
3. **Switch Router**: Routes users to appropriate execution path based on command
4. **Meal Management**: Create, read, update, delete operations on meal records
5. **Response Handler**: Sends contextual messages based on user actions and meal data

---

## 🔧 Tools and Services Used and Why

### **Telegram**
Telegram provides reliable real-time messaging through webhooks and an easy-to-use Bot API. It eliminates the need for custom message infrastructure and handles user authentication seamlessly. Chosen for its simplicity, global user base, and excellent bot framework.

### **Supabase**
Supabase offers a PostgreSQL-based backend with a straightforward REST API and real-time capabilities. It provides reliable data persistence for user profiles and meal logs without requiring complex database management. Used for storing user state and tracking meal entries with timestamps.

### **n8n**
n8n is a low-code workflow automation platform that integrates multiple services without writing backend code. It handles orchestration of API calls, conditional routing, and data transformation elegantly through a visual interface. Used to connect Telegram and Supabase together and manage the complex multi-step meal tracking flow.

---

## 💡 Assumptions and Trade-offs

This project operates with the following key considerations:

**No Explicit Assumptions**: The workflow is designed to work flexibly with any Telegram bot and Supabase database setup without imposing specific business logic assumptions.

**Design Trade-offs**:
- **Synchronous Execution**: All API calls are executed sequentially, prioritizing reliability over maximum throughput
- **Daily Meal Tracking**: Meals are filtered by creation date (today's date), resetting each day automatically
- **Simple User Management**: Uses basic active/inactive status without role-based access control
- **Text-based Interaction**: All meal input is text-based, requiring users to type meal names
- **No Nutritional Data**: The system stores meal names only, without calorie or nutritional information
- **No Persistence of Errors**: Failed API calls are not automatically retried; manual intervention may be required for reliability

---

## ⏱️ Time Breakdown per Section

| Phase | Duration | Notes |
|-------|----------|-------|
| **Architecture & Design** | 16 hours | Planned workflow structure, data schema, and meal tracking logic |
| **Telegram Integration** | 3 hours | Setting up bot triggers and message routing |
| **Supabase Setup** | 3 hours | Database schema design and API configuration |
| **CRUD Operations Implementation** | 6 hours | Create, read, update, delete meal functionality |
| **User State Management** | 2 hours | User creation, activation, and deactivation |
| **Testing & Deployment** | 2 hours | End-to-end testing, credential setup, and workflow activation |
| **Total** | **32 hours** | — |

---

## 🔄 n8n Workflow Architecture

### **Entry Point: Telegram Trigger**
- Listens for incoming Telegram messages
- Captures user data (ID, username, message content)
- Initiates the workflow for each message

### **Core Logic Nodes**

#### 1. **Get a row** (User Lookup)
- Queries Supabase `Users` table to check if user exists
- Uses Telegram user ID as the lookup key
- Always outputs data (even if no match found)

#### 2. **Switch1 Node** (Primary Router)
Routes users based on multiple conditions:
- **Condition 1**: New user (empty `user_id`) → User initialization path
- **Condition 2**: Menu commands (Log, View, Edit, Delete) → Meal operations path
- **Condition 3**: Invalid/unknown commands → Error handling path
- **Condition 4**: User inactive → Prevent further operations
- **Condition 5**: Stop command → User deactivation path

---

## 🚀 Execution Paths

### **Path 1: New User (Initialization)**
1. `Create New User` → Saves new user to Supabase with active status
2. `Send message to new user` → Sends welcome menu with all available options

### **Path 2: Log Meal**
1. `Meal Context` → Sends prompt asking for meal name
2. `Create Meal` → Creates meal entry in Supabase with timestamp
3. `Meal Context1` → Sends confirmation message with logged meal

### **Path 3: View Meals**
1. `If` → Checks if command is "View Meals"
2. `Get Meals` → Retrieves all meals logged today by user
3. `View Meals` → Sends formatted list of meals to user
4. `Update User1` → Logs user action for tracking

### **Path 4: Edit Meal**
1. `Meal Context` → Sends instructions for edit format (Old, New)
2. `Update Meal` → Parses comma-separated input and updates meal name
3. `Meal Context1` → Sends confirmation of meal update

### **Path 5: Delete Meal**
1. `Meal Context` → Sends prompt for meal name to delete
2. `Delete Meal` → Removes meal from today's records
3. `Meal Context1` → Sends deletion confirmation

### **Path 6: Stop/Deactivate**
1. `Send a text message` → Sends goodbye message
2. `Update User` → Sets `is_user_active` to False, preventing future commands

---

## 🔗 Data Integrations

| Service | Purpose |
|---------|---------|
| **Telegram** | User messaging and command interface |
| **Supabase** | Stores users and meal records with timestamps |

---

## 📊 Key Features

✅ **User Registration** - Automatic new user creation on first interaction  
✅ **Meal Logging** - Quick meal entry with timestamp tracking  
✅ **Meal History** - View all meals logged today with formatting  
✅ **Edit Meals** - Update meal entries with simple comma-separated format  
✅ **Delete Meals** - Remove meals from daily records  
✅ **User State Management** - Track user activity and engagement  
✅ **Daily Reset** - Automatically filters meals by date  
✅ **Interactive Menu** - Emoji-enhanced keyboard interface

---

## 🗄️ Database Schema

### Users Table
- `user_id`: Telegram user ID (primary key)
- `username`: Telegram username
- `is_user_active`: Boolean flag (True/False) indicating user status
- `last_event_action`: Stores last command executed (Log Meal, View Meals, etc.)
- `created_at`: Timestamp of user registration

### Meals Table
- `id`: Auto-incrementing primary key
- `user_id`: Reference to user
- `meal_name`: Name of the meal logged
- `created_at`: Timestamp of meal entry
- `updated_at`: Last modification timestamp

---

## 📈 Workflow Status

- **Active**: true (workflow is currently active)  
- **Execution Order**: v1  
- **Binary Mode**: separate

---

## 🎯 Use Cases

- Tracking daily meal intake for health monitoring  
- Building meal history for dietary analysis  
- Quick meal logging without complex interfaces  
- User engagement through simple chatbot interaction  
- Meal data collection for health research  
- Personal nutrition awareness and accountability

---

## 📝 Usage Examples

### Logging a Meal
```
User: /start
Bot: Welcome to your Health Tracker 🥗
     Choose an option below:
     🍽 Log Meal | 📊 View Meals | ✏ Edit Meal | 🗑 Delete Meal

User: 🍽 Log Meal
Bot: What meal did you eat? 🍽

User: Grilled Chicken with Rice
Bot: ✅ Meal logged!
     🍽 Grilled Chicken with Rice has been added to your meals today
```

### Viewing Meals
```
User: 📊 View Meals
Bot: Today's Meals 🍽
     1. Grilled Chicken with Rice
     2. Apple
     3. Green Salad
```

### Editing a Meal
```
User: ✏ Edit Meal
Bot: Send the old meal and the new meal separated by a comma.
     Example: Chicken Rice, Pasta

User: Grilled Chicken with Rice, Roasted Chicken with Vegetables
Bot: ✅ Meal updated!
     Grilled Chicken with Rice → Roasted Chicken with Vegetables
```

### Deleting a Meal
```
User: 🗑 Delete Meal
Bot: Send the meal name to delete.
     Example: Pasta

User: Apple
Bot: ✅ Meal deleted!
     Apple has been removed from your meals.
```

---

## 🛠️ Troubleshooting

### Bot doesn't respond
- Verify Telegram bot token is correct in credentials
- Check webhook is properly configured
- Ensure n8n workflow is active

### Meals not saving
- Verify Supabase credentials are correct
- Check that Meals table exists with proper schema
- Ensure user is active (is_user_active = True)

### Cannot view meals
- Confirm Get Meals node is filtering by current date correctly
- Check user_id filter is matching Telegram user ID
- Verify Supabase API key has read permissions

---

## 🚀 Future Enhancements

- Add nutritional information tracking (calories, protein, carbs)
- Implement meal categories (breakfast, lunch, dinner, snacks)
- Add health metrics tracking (weight, water intake)
- Create meal analysis and recommendations
- Add reminders for meal logging
- Implement multi-user household tracking
- Add meal photo/image support
- Integration with nutrition databases (USDA, etc.)

---

## 📞 Support and Contributions

For issues, feature requests, or contributions, please visit the repository and create an issue or pull request.

---

**Created by**: sushant18b
**Last Updated**: 2026-03-29
**License**: MIT