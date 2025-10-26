# Telex-X-Mastra

> A2A (Agent-to-Agent) protocol integration between Mastra and Telex for intelligent workflow automation

![Node.js](https://img.shields.io/badge/node-%3E%3D20.9.0-brightgreen)
![TypeScript](https://img.shields.io/badge/TypeScript-5.9.3-blue)
![License](https://img.shields.io/badge/license-ISC-blue)

## Overview

Telex-X-Mastra is an AI-powered weather information system that implements the A2A (Agent-to-Agent) protocol for seamless integration between Mastra AI agents and the Telex platform. It features an intelligent weather agent capable of providing real-time weather data, multi-language support, and weather-based activity recommendations through a JSON-RPC 2.0 compliant API.

## Features

- **AI-Powered Weather Agent** - Intelligent conversational agent using Google Gemini 2.0 Flash
- **Multi-Language Support** - Automatic translation of non-English location names
- **Real-Time Weather Data** - Integration with Open-Meteo APIs for accurate weather information
- **A2A Protocol Compliance** - Full JSON-RPC 2.0 implementation for agent-to-agent communication
- **Workflow Orchestration** - Multi-step workflows for complex weather planning tasks
- **Conversation Memory** - Persistent storage using LibSQL for maintaining context
- **Evaluation Framework** - Built-in scorers for quality assurance and agent performance
- **RESTful API** - Complete API with Swagger UI documentation
- **Activity Recommendations** - Weather-based suggestions for optimal activity planning
- **Tool Integration** - Extensible tool system for external data sources

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        A2A Protocol Layer                    │
│                     (JSON-RPC 2.0 API)                       │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                      Mastra Core Engine                      │
│  ┌────────────┐  ┌──────────┐  ┌──────────────────────┐   │
│  │   Agents   │  │ Workflows│  │    Evaluators        │   │
│  │            │  │          │  │  (Scorers)           │   │
│  └─────┬──────┘  └────┬─────┘  └──────────────────────┘   │
│        │              │                                      │
│  ┌─────▼──────────────▼─────┐  ┌──────────────────────┐   │
│  │       Tools               │  │    Memory Store      │   │
│  │  (Weather, etc.)          │  │     (LibSQL)         │   │
│  └───────────────────────────┘  └──────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                   External Services                          │
│  ┌──────────────────┐  ┌────────────────────────────┐      │
│  │ Open-Meteo       │  │  Google Generative AI      │      │
│  │ Weather API      │  │  (Gemini 2.0 Flash)        │      │
│  └──────────────────┘  └────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## Prerequisites

- **Node.js** >= 20.9.0
- **npm** or **pnpm** package manager
- **Google Generative AI API Key** (for Gemini model access)

## Installation

1. Clone the repository:
```bash
git clone https://github.com/hngprojects/mastra-x-telex.git
cd telex-x-mastra
```

2. Install dependencies:
```bash
npm install
# or
pnpm install
```

3. Configure environment variables:
```bash
cp .env.example .env
```

Edit `.env` and add your API key:
```env
GOOGLE_GENERATIVE_AI_API_KEY=your_google_api_key_here
```

## Configuration

The project uses a centralized configuration in `src/mastra/index.ts`:

- **Workflows**: Weather planning workflow
- **Agents**: Weather agent with Gemini 2.0 Flash
- **Scorers**: Tool call, completeness, and translation evaluators
- **Storage**: LibSQL in-memory database
- **Logger**: Pino logger with debug level
- **Server**: OpenAPI docs and Swagger UI enabled

## Usage

### Starting the Server

Development mode with hot reload:
```bash
npm run dev
```

Production build and start:
```bash
npm run build
npm run start
```

The server will start on `http://localhost:4111` (default Mastra port).

### API Endpoints

- **A2A Agent Endpoint**: `POST /a2a/agent/:agentId`
- **Swagger UI**: `http://localhost:4111/api-docs`
- **Health Check**: `GET /health` (if configured)

## A2A Protocol API

### Request Format

#### Option 1: A2A `message/send` (Recommended)

```bash
curl -X POST http://localhost:4111/a2a/agent/weatherAgent \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "request-001",
    "method": "message/send",
    "params": {
      "message": {
        "kind": "message",
        "role": "user",
        "parts": [
          {
            "kind": "text",
            "text": "What is the weather in Lagos?"
          }
        ],
        "messageId": "msg-001",
        "taskId": "task-001"
      },
      "configuration": {
        "blocking": true
      }
    }
  }'
```

#### Option 2: Simplified `execute` Format

```bash
curl -X POST http://localhost:4111/a2a/agent/weatherAgent \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "request-001",
    "method": "execute",
    "params": {
      "messages": [
        {
          "role": "user",
          "parts": [
            {
              "kind": "text",
              "text": "What is the weather in New York?"
            }
          ]
        }
      ]
    }
  }'
```

### Response Format

```json
{
  "jsonrpc": "2.0",
  "id": "request-001",
  "result": {
    "id": "task-001",
    "contextId": "context-uuid",
    "status": {
      "state": "completed",
      "timestamp": "2025-10-26T14:30:00.000Z",
      "message": {
        "messageId": "response-uuid",
        "role": "agent",
        "parts": [
          {
            "kind": "text",
            "text": "The current weather in Lagos is thunderstorm with a temperature of 29°C..."
          }
        ],
        "kind": "message"
      }
    },
    "artifacts": [
      {
        "artifactId": "artifact-uuid",
        "name": "weatherAgentResponse",
        "parts": [
          {
            "kind": "text",
            "text": "The current weather in Lagos..."
          }
        ]
      }
    ],
    "history": [
      {
        "kind": "message",
        "role": "user",
        "parts": [{ "kind": "text", "text": "What is the weather in Lagos?" }],
        "messageId": "msg-001",
        "taskId": "task-001"
      },
      {
        "kind": "message",
        "role": "agent",
        "parts": [{ "kind": "text", "text": "The current weather in Lagos..." }],
        "messageId": "response-uuid",
        "taskId": "task-001"
      }
    ],
    "kind": "task"
  }
}
```

### Multi-Turn Conversations

To continue a conversation, include the `contextId` and `taskId` from the previous response:

```bash
curl -X POST http://localhost:4111/a2a/agent/weatherAgent \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "request-002",
    "method": "execute",
    "params": {
      "contextId": "existing-context-id",
      "taskId": "existing-task-id",
      "messages": [
        {
          "role": "user",
          "parts": [
            {
              "kind": "text",
              "text": "What about tomorrow?"
            }
          ]
        }
      ]
    }
  }'
```

### Error Responses

#### Invalid Request (Missing JSON-RPC fields)
```json
{
  "jsonrpc": "2.0",
  "id": "request-001",
  "error": {
    "code": -32600,
    "message": "Invalid Request: jsonrpc must be \"2.0\" and id is required"
  }
}
```

#### Agent Not Found
```json
{
  "jsonrpc": "2.0",
  "id": "request-001",
  "error": {
    "code": -32602,
    "message": "Agent 'unknownAgent' not found"
  }
}
```

#### Internal Error
```json
{
  "jsonrpc": "2.0",
  "id": "request-001",
  "error": {
    "code": -32603,
    "message": "Internal error: [error details]"
  }
}
```

## Weather Agent

### Overview

The Weather Agent (`weatherAgent`) is an AI-powered conversational agent that provides accurate weather information and activity recommendations.

**Model**: Google Gemini 2.0 Flash
**Location**: `src/mastra/agents/weather-agent.ts`

### Capabilities

- Retrieves current weather data for any global location
- Automatically translates non-English location names
- Provides detailed weather information including:
  - Temperature and "feels like" temperature
  - Humidity levels
  - Wind speed and gusts
  - Current weather conditions
  - Precipitation details
- Suggests weather-appropriate activities
- Maintains conversation context with memory

### Agent Instructions

The agent follows these guidelines:

1. **Always asks for location** if not provided in the user's query
2. **Translates location names** to English for API compatibility
3. **Handles multi-part locations** (e.g., "New York, NY" → "New York")
4. **Provides concise responses** with relevant details
5. **Suggests activities** based on weather forecasts

### Example Interactions

**Simple Query:**
```
User: Lagos
Agent: The current weather in Lagos is thunderstorm with a temperature
      of 29°C, feeling like 32°C. Humidity is at 75%, and winds are
      blowing at 4 m/s. Would you like activity suggestions?
```

**Non-English Location:**
```
User: What's the weather in 东京?
Agent: [Translates to "Tokyo"] The current weather in Tokyo is partly
       cloudy with a temperature of 18°C...
```

## Weather Tool

### Overview

The Weather Tool (`weatherTool`) fetches real-time weather data from Open-Meteo APIs.

**Location**: `src/mastra/tools/weather-tool.ts`

### API Integration

1. **Geocoding API**: Converts location names to coordinates
   - Endpoint: `https://geocoding-api.open-meteo.com/v1/search`
   - Returns: Latitude, longitude, location name

2. **Weather API**: Retrieves current weather conditions
   - Endpoint: `https://api.open-meteo.com/v1/forecast`
   - Returns: Temperature, humidity, wind, conditions

### Output Schema

```typescript
{
  temperature: number,      // Current temperature in °C
  feelsLike: number,       // Apparent temperature in °C
  humidity: number,        // Relative humidity (0-100%)
  windSpeed: number,       // Wind speed in m/s
  windGust: number,        // Wind gust speed in m/s
  conditions: string,      // Human-readable weather description
  location: string         // Verified location name
}
```

### Weather Conditions

The tool supports 100+ WMO weather codes, including:

- Clear sky (0)
- Mainly clear, partly cloudy (1-3)
- Fog and depositing rime fog (45, 48)
- Drizzle (51-57)
- Rain (61-67)
- Snow (71-77, 85-86)
- Rain showers (80-82)
- Thunderstorms (95-99)

## Weather Workflow

### Overview

The Weather Workflow (`weatherWorkflow`) orchestrates a two-step process for comprehensive weather planning.

**Location**: `src/mastra/workflows/weather-workflow.ts`

### Workflow Steps

#### Step 1: Fetch Weather
- **Input**: City name
- **Process**: Queries Open-Meteo APIs
- **Output**:
  - Temperature range (max/min)
  - Precipitation probability
  - Weather conditions

#### Step 2: Plan Activities
- **Input**: Weather forecast data
- **Process**: Calls Weather Agent with formatted prompt
- **Output**: Structured activity suggestions with:
  - Morning activities
  - Afternoon activities
  - Indoor alternatives
  - Weather-specific recommendations
  - Formatted with emojis and sections

### Workflow Features

- **Structured Output**: Consistent formatting with emojis
- **Real-time Streaming**: Text streaming for responsive UX
- **Context-Aware**: Considers precipitation and temperature
- **Multi-Period Planning**: Morning and afternoon suggestions
- **Indoor Alternatives**: Backup plans for poor weather

### Usage Example

```typescript
import { weatherWorkflow } from './src/mastra/workflows/weather-workflow';

const result = await weatherWorkflow.execute({
  triggerData: {
    city: 'New York'
  }
});

console.log(result);
```

## Evaluators (Scorers)

The system includes three evaluation scorers for quality assurance:

### 1. Tool Call Appropriateness Scorer

**Purpose**: Validates that the agent calls the weather tool when appropriate
**Type**: Built-in scorer
**Mode**: Non-strict
**Location**: `src/mastra/scorers/weather-scorer.ts`

### 2. Completeness Scorer

**Purpose**: Ensures responses are comprehensive and address user queries
**Type**: Built-in scorer
**Location**: `src/mastra/scorers/weather-scorer.ts`

### 3. Translation Scorer

**Purpose**: Validates translation quality for non-English locations
**Type**: Custom multi-step LLM-as-judge scorer
**Judge Model**: Google Gemini 2.0 Flash
**Location**: `src/mastra/scorers/weather-scorer.ts`

#### Translation Scorer Logic

1. Detects if location name is in non-English language
2. Verifies correct translation to English
3. Lenient with transliteration differences
4. Outputs confidence scores

**Sampling**: All scorers use 100% sampling rate (ratio: 1)

## Project Structure

```
telex-x-mastra/
├── src/
│   └── mastra/
│       ├── agents/
│       │   └── weather-agent.ts           # Weather AI agent
│       ├── tools/
│       │   └── weather-tool.ts            # Weather data tool
│       ├── workflows/
│       │   └── weather-workflow.ts        # Weather planning workflow
│       ├── routes/
│       │   ├── a2a-agent-route.ts         # A2A protocol endpoint
│       │   └── a2a-example.md             # API documentation
│       ├── scorers/
│       │   └── weather-scorer.ts          # Evaluation scorers
│       └── index.ts                       # Mastra configuration
├── .mastra/                               # Mastra internal files
│   └── mastra.db                          # SQLite database
├── package.json                           # Dependencies
├── tsconfig.json                          # TypeScript config
├── .env                                   # Environment variables
├── .gitignore                             # Git ignore rules
└── test-a2a-request.json                  # Test request example
```

## Technologies

### Core Framework
- **[@mastra/core](https://mastra.ai)** (v0.22.2) - AI agent framework
- **[@mastra/memory](https://mastra.ai)** (v0.15.8) - Memory management
- **[@mastra/evals](https://mastra.ai)** (v0.14.1) - Evaluation framework
- **[@mastra/loggers](https://mastra.ai)** (v0.10.17) - Logging utilities
- **[@mastra/libsql](https://mastra.ai)** (v0.16.0) - SQLite storage

### Language & Validation
- **TypeScript** (v5.9.3) - Type-safe development
- **Zod** (v3.25.76) - Schema validation
- **Node.js** (>= 20.9.0) - Runtime environment

### AI & External Services
- **Google Generative AI** - Gemini 2.0 Flash model
- **Open-Meteo API** - Weather data and geocoding

### Development Tools
- **Mastra CLI** (v0.17.3) - Development and build tools
- **ESM** (ES2022) - Modern JavaScript modules

## Development

### Available Scripts

```bash
# Start development server with hot reload
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run tests (not yet implemented)
npm run test
```

### Adding a New Agent

1. Create agent file in `src/mastra/agents/`:

```typescript
import { Agent } from '@mastra/core';

export const myAgent = new Agent({
  name: 'My Agent',
  instructions: 'Agent instructions...',
  model: {
    provider: 'GOOGLE',
    name: 'gemini-2.0-flash-exp',
    toolChoice: 'auto',
  },
  tools: {
    // Add tools here
  },
});
```

2. Register in `src/mastra/index.ts`:

```typescript
import { myAgent } from './agents/my-agent';

export const mastra = new Mastra({
  agents: { myAgent, weatherAgent },
  // ...
});
```

### Adding a New Tool

1. Create tool file in `src/mastra/tools/`:

```typescript
import { createTool } from '@mastra/core';
import { z } from 'zod';

export const myTool = createTool({
  id: 'myTool',
  description: 'Tool description',
  inputSchema: z.object({
    // Define input schema
  }),
  outputSchema: z.object({
    // Define output schema
  }),
  execute: async ({ context }) => {
    // Tool logic here
  },
});
```

2. Add to agent's tools in agent configuration.

### Adding a New Workflow

1. Create workflow file in `src/mastra/workflows/`:

```typescript
import { Workflow, Step } from '@mastra/core';

const step1 = new Step({
  id: 'step1',
  execute: async ({ context }) => {
    // Step logic
  },
});

export const myWorkflow = new Workflow({
  name: 'my-workflow',
  triggerSchema: z.object({
    // Define trigger schema
  }),
});

myWorkflow.step(step1).commit();
```

2. Register in `src/mastra/index.ts`:

```typescript
import { myWorkflow } from './workflows/my-workflow';

export const mastra = new Mastra({
  workflows: { myWorkflow, weatherWorkflow },
  // ...
});
```

## Environment Variables

Create a `.env` file with the following variables:

```env
# Required
GOOGLE_GENERATIVE_AI_API_KEY=your_google_api_key_here

# Optional
PORT=4111                                  # Server port (default: 4111)
LOG_LEVEL=debug                            # Log level (debug, info, warn, error)
DATABASE_URL=:memory:                      # Database connection string
```

## API Documentation

Once the server is running, access the interactive API documentation:

- **Swagger UI**: `http://localhost:4111/api-docs`
- **OpenAPI Spec**: `http://localhost:4111/openapi.json`

The Swagger UI provides:
- Complete API endpoint documentation
- Request/response schemas
- Interactive API testing
- Example requests and responses

## Testing

### Using the Test File

A test request example is provided in `test-a2a-request.json`. Use it with curl:

```bash
curl -X POST http://localhost:4111/a2a/agent/weatherAgent \
  -H "Content-Type: application/json" \
  -d @test-a2a-request.json
```

### Manual Testing

Test the weather agent with various queries:

```bash
# Simple location query
curl -X POST http://localhost:4111/a2a/agent/weatherAgent \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "test-1",
    "method": "execute",
    "params": {
      "messages": [
        {
          "role": "user",
          "parts": [{"kind": "text", "text": "Tokyo"}]
        }
      ]
    }
  }'

# Non-English location
curl -X POST http://localhost:4111/a2a/agent/weatherAgent \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "test-2",
    "method": "execute",
    "params": {
      "messages": [
        {
          "role": "user",
          "parts": [{"kind": "text", "text": "北京"}]
        }
      ]
    }
  }'
```

## Troubleshooting

### Common Issues

1. **Agent not found error**
   - Verify agent is registered in `src/mastra/index.ts`
   - Check agent name matches the URL parameter

2. **API key errors**
   - Ensure `GOOGLE_GENERATIVE_AI_API_KEY` is set in `.env`
   - Verify the API key is valid and has proper permissions

3. **Weather data not fetching**
   - Check internet connection
   - Verify Open-Meteo APIs are accessible
   - Check location name spelling

4. **Database errors**
   - Check `.mastra/` directory permissions
   - Verify LibSQL is properly installed

5. **Port already in use**
   - Change port in `.env` file
   - Kill existing process on port 4111

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a pull request

### Coding Standards

- Use TypeScript for all new code
- Follow the existing code style
- Add JSDoc comments for public APIs
- Write descriptive commit messages
- Test your changes thoroughly

## License

ISC License - see LICENSE file for details

## Author

**Phoenix**

## Repository

[https://github.com/hngprojects/mastra-x-telex](https://github.com/hngprojects/mastra-x-telex)

## Support

For issues and questions:
- Open an issue on GitHub
- Check the [Mastra documentation](https://mastra.ai/docs)
- Review the A2A protocol specification

## Acknowledgments

- [Mastra Framework](https://mastra.ai) - AI agent framework
- [Open-Meteo](https://open-meteo.com) - Weather data APIs
- [Google Generative AI](https://ai.google.dev) - Gemini models

---

Built with Mastra and powered by Google Gemini 2.0 Flash
