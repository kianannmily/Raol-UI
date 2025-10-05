# Raol APIs

<div align="center">
  <img src="https://github.com/raolbyte/raolbyte/blob/main/banner.png" alt="Raol APIs Logo" width="120" height="120">
  
  **Simple and easy to use API with Discord Bot Integration.**
  
  [![Version](https://img.shields.io/badge/version-BETA%207.1.0-blue.svg)](https://github.com/raolbyte/Raol-UI)
  [![Node.js](https://img.shields.io/badge/node.js-%3E%3D%2018.0.0-green.svg)](https://nodejs.org/)
  [![License](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
  [![Status](https://img.shields.io/badge/status-online-brightgreen.svg)](https://raol-apis.vercel.app)
  [![Discord](https://img.shields.io/badge/discord-bot%20ready-7289da.svg)](https://discord.com/developers/applications)
</div>

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Discord Bot Integration](#discord-bot-integration)
- [Configuration](#configuration)
- [API Key Management](#api-key-management)
- [Rate Limiting](#rate-limiting)
- [Maintenance Mode](#maintenance-mode)
- [License](#license)

## Installation

### Prerequisites

- Node.js >= 18.0.0
- npm or yarn package manager

### Clone the Repository

```bash
git clone https://github.com/raolbyte/Raol-UI.git
cd Raol-UI
```

### Install Dependencies

```bash
npm install
```

### Environment Setup

Copy the example environment file and configure it:

```bash
cp .env.example .env
```

Edit the `.env` file with your configuration:

```env
# Discord Bot Configuration
# Get your bot token from https://discord.com/developers/applications
DISCORD_TOKEN=

# Server Configuration
PORT=3000
```

**Note:** The Discord bot is optional. If you don't provide a `DISCORD_TOKEN`, the API will work normally without Discord integration.

## Quick Start

### Development Mode

```bash
npm start
```

### Production Mode

```bash
npm run build
npm run production
```

The server will start on port 3000 (or the port specified in your environment variables).

## Discord Bot Integration

### Features

The Discord bot provides powerful API management through slash commands:

- **`/stats`** - View real-time API statistics with time period support
- **`/maintenance`** - Toggle maintenance mode on/off
- **`/apikey`** - Manage API keys (add, delete, toggle, list)
- **`/endpoint`** - Manage API endpoints (add, delete, list, scan)

### Setting Up Discord Bot

1. **Create a Discord Application:**
   - Go to [Discord Developer Portal](https://discord.com/developers/applications)
   - Create a new application
   - Go to "Bot" section and create a bot
   - Copy the bot token

2. **Configure Bot Permissions:**
   - Enable "Message Content Intent" in Bot settings
   - Invite bot to your server with appropriate permissions

3. **Add Token to Environment:**
   ```env
   DISCORD_TOKEN=your_actual_bot_token_here
   ```

### Discord Commands

#### `/help`
Show available commands and help information with detailed descriptions.

#### `/stats [action] [time]`
View API statistics with advanced options:
- **Actions:**
  - `start_auto` - Start auto-updating stats (updates every 30 seconds)
  - `stop_auto` - Stop auto-updating stats
  - `view` - View current stats (default)
- **Time Periods:**
  - `5m` - Last 5 minutes
  - `15m` - Last 15 minutes  
  - `30m` - Last 30 minutes
  - `1h` - Last hour
  - `6h` - Last 6 hours
  - `12h` - Last 12 hours
  - `1d` - Last day
  - `3d` - Last 3 days
  - `7d` - Last week

#### `/activity [action] [status]`
Manage bot activity status:
- `set_custom <status>` - Set custom bot status text
- `reset_auto` - Reset to automatic status rotation
- `show_current` - Show current activity status

#### `/maintenance [action]`
Toggle maintenance mode:
- `on` - Enable maintenance mode
- `off` - Disable maintenance mode

#### `/apikey [subcommand]`
Advanced API key management with categories:
- **Add:** `add <key> <name> <category> <ratelimit>`
  - Categories: `free`, `premium`, `vip`, `admin`
  - Rate limits: `100/minute`, `500/minute`, `1000/minute`, `5000/day`, `10000/day`, `50000/day`, `unlimited`
- **Delete:** `delete <key>` - Delete API key
- **Toggle:** `toggle <enable/disable>` - Enable/disable API key requirement
- **List:** `list` - List all API keys with details

#### `/endpoint [subcommand]`
Complete API endpoint management system:
- **Add:** `add <name> <filename> <category> <method> [description] [parameters] [optional_parameters]`
  - **Name:** Display name for documentation (e.g., "Weather API", "Translate Text")
  - **Filename:** File name for endpoint (e.g., "weather", "translate")
  - Categories: `ai`, `maker`, `random`, `tools`, `games`, `social`, `news`, `custom`
  - Methods: `GET`, `POST`, `PUT`, `DELETE`
  - Parameters: Required parameters (comma-separated)
  - Optional Parameters: Optional parameters (comma-separated)
  - Auto-generates endpoint files and updates documentation
- **Delete:** `delete <filename> <category>` - Delete endpoint and remove from docs
- **List:** `list` - Show all available endpoints
- **Scan:** `scan` - Scan folder structure for existing endpoints

### Auto Stats Updates

The bot automatically updates statistics every 30 seconds when auto-stats is enabled via `/stats start_auto`. The stats are updated in the same channel where the command was used, with message editing to prevent spam.

### Endpoint Management

The Discord bot provides complete endpoint lifecycle management:

#### Creating Endpoints
```bash
/endpoint add name:"Weather API" filename:weather category:tools method:GET description:Get weather information parameters:location optional_parameters:format,units
```

This command will:
- **Display Name:** "Weather API" (shown in documentation)
- **Filename:** "weather" (creates `src/api/tools/weather.js`)
- **Path:** `/tools/weather`
- Auto-generate parameter validation
- Update `settings.json` for documentation
- Make the endpoint immediately available at `/docs`

#### Generated Template Features
- **No Comments**: Clean code without `//` comments
- **Smart Validation**: Required parameters are validated, optional ones are not
- **Auto Examples**: Intelligent parameter examples based on parameter names
- **Complete Documentation**: Full response with examples and usage
- **Error Handling**: Proper error responses and logging

#### Example Generated Code
```javascript
import axios from "axios"
import { createApiKeyMiddleware } from "../../middleware/apikey.js"

export default (app) => {
  app.get("/tools/weather", createApiKeyMiddleware(), async (req, res) => {
    try {
      const location = req.query.location
      const format = req.query.format
      const units = req.query.units
      
      if (!location) {
        return res.status(400).json({ 
          status: false, 
          error: "location is required" 
        })
      }
      
      res.status(200).json({
        status: true,
        message: "Get weather information",
        endpoint: "/tools/weather",
        method: "GET",
        required_parameters: {
          "location": "New York"
        },
        optional_parameters: {
          "format": "example_format",
          "units": "example_units"
        },
        example: {
          url: "https://your-domain.com/tools/weather?location=New York&format=example_format&units=example_units",
          method: "GET",
          query: {
            "location": "New York",
            "format": "example_format",
            "units": "example_units"
          }
        }
      })
      
    } catch (error) {
      console.error("tools/weather API Error:", error)
      res.status(500).json({ 
        status: false, 
        error: error.message || "Internal server error" 
      })
    }
  })
}
```

### Bot Activity Management

The Discord bot features intelligent activity management:
- **Automatic Rotation:** Bot status changes every 30 seconds between different activities
- **Custom Status:** Set custom status text using `/activity set_custom <text>`
- **Activity Types:** Playing, Watching, Listening, Competing
- **Auto Reset:** Use `/activity reset_auto` to return to automatic rotation

## Configuration

### Settings.json Structure

The `src/settings.json` file controls all aspects of your API service:

```json
{
  "name": "Raol Api'S",
  "version": "v7.1.0",
  "description": "Experience the next generation of API documentation...",
  "maintenance": {
    "enabled": false
  },
  "bannerImage": "/src/banner.jpg",
  "previewImage": "/src/preview.png",
  "header": {
    "status": "Online"
  },
  "apiSettings": {
    "creator": "RaolByte",
    "requireApikey": false,
    "apikey": {
      "your-api-key": {
        "rateLimit": "5000/day",
        "enabled": true
      }
    }
  },
  "categories": [
    {
      "name": "Category Name",
      "items": [
        {
          "name": "API Endpoint Name",
          "desc": "Description of the endpoint",
          "path": "/endpoint/path",
          "status": "ready",
          "params": {
            "param1": "Parameter description"
          }
        }
      ]
    }
  ]
}
```

### Configuration Options

#### Basic Settings
- `name` - Your API service name
- `version` - Current version number
- `description` - Brief description of your service
- `bannerImage` - Path to banner image
- `previewImage` - Path to preview image

#### Maintenance Mode
```json
"maintenance": {
  "enabled": true
}
```

#### API Settings
```json
"apiSettings": {
  "creator": "Your Team Name",
  "requireApikey": false,
  "apikey": {
    "api-key-name": {
      "rateLimit": "5000/day",
      "enabled": true
    }
  }
}
```

#### Rate Limit Formats
- `"unlimited"` - No rate limiting
- `"100/minute"` - 100 requests per minute
- `"1000/hour"` - 1000 requests per hour
- `"5000/day"` - 5000 requests per day

## API Key Management

### Enabling API Key Authentication

Set `requireApikey` to `true` in your `settings.json`:

```json
"apiSettings": {
  "requireApikey": true,
  "apikey": {
    "your-secret-key": {
      "rateLimit": "1000/day",
      "enabled": true
    }
  }
}
```

### Using API Keys

When API keys are required, include them in your requests:

```bash
curl "http://localhost:3000/ai/luminai?text=Hello&apikey=your-secret-key"
```

### API Key Responses

#### No API Key Provided (when required)
```json
{
  "status": false,
  "creator": "RaolByte",
  "error": "API key required",
  "message": "Please provide a valid API key in the query parameters"
}
```

#### Invalid API Key
```json
{
  "status": false,
  "creator": "RaolByte",
  "error": "Invalid API key",
  "message": "The provided API key is not valid or does not exist"
}
```

## Rate Limiting

### Global Rate Limiting
- **Default:** 50 requests per minute per IP
- **Window:** 1 minute
- **Bypass:** When `requireApikey` is `false`, API endpoints bypass global rate limiting

### API Key Rate Limiting
- **Configurable per key**
- **Formats:** `unlimited`, `100/minute`, `1000/hour`, `5000/day`
- **Tracking:** Per API key, not per IP

### Rate Limit Responses

#### Global Rate Limit Exceeded
```json
{
  "status": false,
  "creator": "RaolByte",
  "error": "Rate limit exceeded",
  "message": "You have exceeded the rate limit for this API key"
}
```

## Maintenance Mode

Enable maintenance mode to temporarily disable API access:

```json
"maintenance": {
  "enabled": true
}
```

### Maintenance Mode Behavior
- **API Endpoints:** Return 503 status with maintenance message
- **Documentation:** Shows maintenance page
- **Bypass Paths:** `/api/settings`, `/assets/`, `/src/`, `/support`

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Issues:** [GitHub Issues](https://github.com/raolbyte/Raol-UI/issues)
- **Contact:** [Support Page](https://whatsapp.com/channel/0029Vb6n9HIDJ6H6oibRvv1D)

---

<div align="center">
  <p>Made with ❤️ by <strong>RaolByte</strong></p>
  <p>
    <a href="https://github.com/raolbyte/Raol-UI">⭐ Star this repo</a> •
    <a href="https://github.com/raolbyte/Raol-UI/issues">🐛 Report Bug</a> •
    <a href="https://github.com/raolbyte/Raol-UI/pulls">💡 Request Feature</a>
  </p>
</div>
