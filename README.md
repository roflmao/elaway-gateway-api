# Elaway Gateway API

A Node.js/TypeScript REST API gateway that provides a simplified interface to interact with Elaway electric vehicle charging infrastructure. This service handles the complex OAuth2 authentication flow and provides straightforward endpoints to control charging sessions.

## Features

- **Automated OAuth2 Authentication**: Handles the complete Elaway authentication flow using Puppeteer
- **Token Management**: Automatic token caching and expiration handling
- **RESTful API**: Simple HTTP endpoints to manage charging sessions
- **Charger Status**: Retrieve detailed information about your charging point
- **Session Control**: Start and stop charging sessions programmatically
- **Docker Support**: Production-ready containerized deployment
- **TypeScript**: Fully typed codebase with strict type checking
- **Input Validation**: Environment configuration validated with Joi schemas

### Known Limitations

- Token refresh not yet implemented (full re-authentication occurs on expiration)
- No authentication required for API endpoints (secure your deployment accordingly)

## Prerequisites

- **Node.js** >= 14.0.0
- **Docker** (recommended for deployment)
- **Elaway Account**: Valid username and password
- **Client Secrets**: Must be obtained using mitmproxy (see Setup section)

## Installation

### Option 1: Docker (Recommended)

1. Copy the example Docker Compose file:
   ```bash
   cp docker-compose.yml.example docker-compose.yml
   ```

2. Edit `docker-compose.yml` and fill in your credentials:
   ```yaml
   services:
     app:
       image: ghcr.io/roflmao/elaway-gateway-api:latest
       environment:
         - ELAWAY_USER=your-email@example.com
         - ELAWAY_PASSWORD=your-password
         - ELAWAY_CLIENT_ID=1
         - ELAWAY_CLIENT_SECRET=your-client-secret
         - CLIENT_ID=your-auth0-client-id
         - PORT=3000
       ports:
         - "3000:3000"
   ```

3. Start the container:
   ```bash
   docker compose up -d
   ```

### Option 2: Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/elaway-gateway-api.git
   cd elaway-gateway-api
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env` file:
   ```bash
   PORT=3000
   ELAWAY_USER=your-email@example.com
   ELAWAY_PASSWORD=your-password
   ELAWAY_CLIENT_ID=1
   ELAWAY_CLIENT_SECRET=your-client-secret
   CLIENT_ID=your-auth0-client-id
   ```

4. Run in development mode:
   ```bash
   npm run dev
   ```

   Or build and run in production mode:
   ```bash
   npm run build
   npm start
   ```

## Configuration

All configuration is done via environment variables:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | No | `3000` | HTTP server port |
| `ELAWAY_USER` | **Yes** | - | Elaway account email address |
| `ELAWAY_PASSWORD` | **Yes** | - | Elaway account password |
| `ELAWAY_CLIENT_ID` | No | `"1"` | Elaway API client ID |
| `ELAWAY_CLIENT_SECRET` | **Yes** | - | Elaway API client secret |
| `CLIENT_ID` | **Yes** | - | Auth0 client ID for OAuth flow |
| `POLLING_INTERVAL` | No | `60000` | Polling interval in milliseconds (currently unused) |
| `AMPECO_API_URL` | No | `https://no.eu-elaway.charge.ampeco.tech/api/v1/app` | Ampeco API base URL |

### Obtaining Client Secrets

For legal reasons, client secrets are not provided in this repository. You must obtain them by intercepting traffic from the official Elaway mobile app:

1. Install [mitmproxy](https://mitmproxy.org/)
2. Configure your iOS/Android device to use mitmproxy as a proxy
3. Install mitmproxy's CA certificate on your device
4. Open the Elaway app and log in
5. Capture the OAuth requests to extract:
   - `ELAWAY_CLIENT_SECRET`
   - `CLIENT_ID`

## API Endpoints

Base URL: `http://localhost:3000` (or your configured host/port)

### Get Charger Status

```http
GET /charger/
```

Retrieves detailed information about your charging point and its current status.

**Response Example:**
```json
{
  "id": "charger-id",
  "name": "My Charger",
  "evses": [...],
  "status": "Available"
}
```

**Error Responses:**
- `500 Internal Server Error` - Failed to fetch charger information

### Start Charging Session

```http
POST /charger/start
```

Starts a new charging session on the configured EVSE.

**Request Body:** None (uses pre-configured EVSE ID)

**Response Example:**
```json
{
  "sessionId": "session-123",
  "status": "Starting",
  "evseId": "evse-456"
}
```

**Error Responses:**
- `500 Internal Server Error` - Failed to start session (includes error details)

### Stop Charging Session

```http
POST /charger/stop
```

Stops the currently active charging session.

**Request Body:** None (automatically finds current session)

**Response Example:**
```json
{
  "status": "Stopped",
  "sessionId": "session-123"
}
```

**Error Responses:**
- `404 Not Found` - No active charging session found
- `500 Internal Server Error` - Failed to stop session

## Usage Examples

### Using cURL

```bash
# Check charger status
curl http://localhost:3000/charger/

# Start charging
curl -X POST http://localhost:3000/charger/start

# Stop charging
curl -X POST http://localhost:3000/charger/stop
```

### Using JavaScript/Fetch

```javascript
// Get charger status
const status = await fetch('http://localhost:3000/charger/').then(r => r.json());
console.log(status);

// Start charging
const startResult = await fetch('http://localhost:3000/charger/start', {
  method: 'POST'
}).then(r => r.json());

// Stop charging
const stopResult = await fetch('http://localhost:3000/charger/stop', {
  method: 'POST'
}).then(r => r.json());
```

### Using Python

```python
import requests

base_url = "http://localhost:3000"

# Get charger status
status = requests.get(f"{base_url}/charger/").json()
print(status)

# Start charging
start = requests.post(f"{base_url}/charger/start").json()

# Stop charging
stop = requests.post(f"{base_url}/charger/stop").json()
```

## Development

### Available Scripts

- `npm run dev` - Start development server with hot reload
- `npm run build` - Build TypeScript to JavaScript
- `npm start` - Run production build
- `npm run lint` - Check code with Biome linter
- `npm run lint:fix` - Auto-fix linting issues
- `npm run clean` - Remove build artifacts

### Project Structure

```
elaway-gateway-api/
├── src/
│   ├── main.ts              # Application entry point
│   ├── config.ts            # Environment configuration & validation
│   ├── auth.ts              # OAuth2 authentication flow
│   └── charger/
│       └── chargerRouter.ts # Express router for charger endpoints
├── dockerfile               # Multi-stage Docker build
├── docker-compose.yml.example
├── package.json
├── tsconfig.json
├── biome.json              # Linter/formatter configuration
└── README.md
```

### Code Style

This project uses [Biome](https://biomejs.dev/) for linting and formatting:

- Run `npm run lint` to check for issues
- Run `npm run lint:fix` to automatically fix issues
- Git hooks (via Lefthook) enforce linting on commit

### Building Docker Images

```bash
# Build local image
docker build -t elaway-gateway-api .

# Run local image
docker run -p 3000:3000 --env-file .env elaway-gateway-api
```

Official images are automatically built and published to GitHub Container Registry on release:
- `ghcr.io/roflmao/elaway-gateway-api:latest`
- `ghcr.io/roflmao/elaway-gateway-api:v1.x.x`

## Troubleshooting

### Authentication Issues

**Problem:** "Failed to authenticate" or constant re-authentication

**Solutions:**
- Verify your `ELAWAY_USER` and `ELAWAY_PASSWORD` are correct
- Check that `CLIENT_ID` and `ELAWAY_CLIENT_SECRET` are valid
- Delete `src/tokens.json` to force fresh authentication
- Ensure your Elaway account is active and has access to a charger

### Puppeteer/Chrome Issues

**Problem:** "Could not find Chrome" or browser launch failures

**Solutions:**
- Ensure Chrome is installed (Docker image includes it automatically)
- For local development on Linux, install Chrome or Chromium:
  ```bash
  # Ubuntu/Debian
  wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
  sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
  sudo apt-get update
  sudo apt-get install google-chrome-stable
  ```

### Port Already in Use

**Problem:** "Error: listen EADDRINUSE: address already in use :::3000"

**Solutions:**
- Change the `PORT` environment variable to an available port
- Stop the conflicting process using port 3000
- For Docker: Change the host port mapping in `docker-compose.yml`

### No Active Session Found

**Problem:** 404 error when stopping charging

**Solutions:**
- Verify a charging session is actually running
- Check the charger status with `GET /charger/` first
- Ensure the session wasn't stopped manually via the Elaway app

### Token Expiration

**Problem:** API calls fail after extended period of inactivity

**Solutions:**
- Token refresh is not yet implemented
- Restart the application to trigger re-authentication
- Monitor logs for "Token expired" messages

## Security Considerations

- **No API Authentication**: The gateway API endpoints have no authentication. Deploy behind a firewall or add authentication middleware if needed.
- **Credential Storage**: Tokens are stored in plaintext in `src/tokens.json`. Secure file permissions accordingly.
- **Environment Variables**: Never commit `.env` files or `docker-compose.yml` with real credentials to version control.
- **Reverse-Engineered Secrets**: Client secrets are obtained through mitmproxy. Use at your own discretion and comply with Elaway's Terms of Service.

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork the repository** and create a feature branch
2. **Follow the code style** using Biome (`npm run lint`)
3. **Test your changes** thoroughly
4. **Write clear commit messages**
5. **Submit a pull request** with a description of your changes

### Development Guidelines

- Use TypeScript strict mode
- Add proper error handling for all async operations
- Validate all external inputs
- Update README.md if adding new features or configuration
- Ensure Docker build succeeds
- Test with real Elaway hardware when possible

### Priority TODOs

- Implement automatic token refresh functionality
- Add API authentication/authorization
- Add unit and integration tests
- Add health check endpoint
- Support multiple chargers/EVSEs
- Add WebSocket support for real-time status updates

## License

This project is provided as-is for educational and personal use. Please review Elaway's Terms of Service before using this software to control their infrastructure.

## Acknowledgments

- Built on the Ampeco charging platform API
- Uses Auth0 for OAuth2 authentication
- Inspired by the need for a simpler API to integrate Elaway chargers with home automation systems
