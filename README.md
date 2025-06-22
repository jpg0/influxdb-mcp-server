[![MseeP Badge](https://mseep.net/pr/idoru-influxdb-mcp-server-badge.jpg)](https://mseep.ai/app/idoru-influxdb-mcp-server)

# InfluxDB MCP Server

[![smithery badge](https://smithery.ai/badge/@idoru/influxdb-mcp-server)](https://smithery.ai/server/@idoru/influxdb-mcp-server)

A Model Context Protocol (MCP) server that exposes access to an InfluxDB instance using the InfluxDB OSS API v2. Mostly built with Claude Code.

## Features

This MCP server provides:

- **Resources**: Access to organization, bucket, and measurement data
- **Tools**: Write data, execute queries, and manage database objects
- **Prompts**: Templates for common Flux queries and Line Protocol format

## Resources

The server exposes the following resources:

1. **Organizations List**: `influxdb://orgs`
   - Displays all organizations in the InfluxDB instance

2. **Buckets List**: `influxdb://buckets`
   - Shows all buckets with their metadata

3. **Bucket Measurements**: `influxdb://bucket/{bucketName}/measurements`
   - Lists all measurements within a specified bucket

4. **Query Data**: `influxdb://query/{orgName}/{fluxQuery}`
   - Executes a Flux query and returns results as a resource

## Tools

The server provides these tools:

1. `write-data`: Write time-series data in line protocol format
   - Parameters: org, bucket, data, precision (optional)

2. `query-data`: Execute Flux queries
   - Parameters: org, query

3. `create-bucket`: Create a new bucket
   - Parameters: name, orgID, retentionPeriodSeconds (optional)

4. `create-org`: Create a new organization
   - Parameters: name, description (optional)

## Prompts

The server offers these prompt templates:

1. `flux-query-examples`: Common Flux query examples
2. `line-protocol-guide`: Guide to InfluxDB line protocol format

## Configuration

The server requires these environment variables:

- `INFLUXDB_TOKEN` (required): Authentication token for the InfluxDB API
- `INFLUXDB_URL` (optional): URL of the InfluxDB instance (defaults to `http://localhost:8086`)
- `INFLUXDB_ORG` (optional): Default organization name for certain operations

## Installation

### Installing via Smithery

To install InfluxDB MCP Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@idoru/influxdb-mcp-server):

```bash
npx -y @smithery/cli install @idoru/influxdb-mcp-server --client claude
```

### Option 1: Run with npx (recommended)

```bash
# Run directly with npx
INFLUXDB_TOKEN=your_token npx influxdb-mcp-server
```

### Option 2: Install globally

```bash
# Install globally
npm install -g influxdb-mcp-server

# Run the server
INFLUXDB_TOKEN=your_token influxdb-mcp-server
```

### Option 3: From source

```bash
# Clone the repository
git clone https://github.com/idoru/influxdb-mcp-server.git
cd influxdb-mcp-server

# Install dependencies
npm install

# Run the server
INFLUXDB_TOKEN=your_token npm start
```

You can also start the server with Streamable HTTP transport by providing the `--http` option with an optional port number (defaults to 3000). This mode uses an internal Express.js server:

```bash
# Start with Streamable HTTP transport on default port 3000
INFLUXDB_TOKEN=your_token npm start -- --http

# Start with Streamable HTTP transport on a specific port
INFLUXDB_TOKEN=your_token npm start -- --http 8080
```

If you installed globally or are using npx, you can run:
```bash
INFLUXDB_TOKEN=your_token influxdb-mcp-server --http
# or
INFLUXDB_TOKEN=your_token influxdb-mcp-server --http 8080
```

## Running with Docker Compose

This project includes a `docker-compose.yml` file to easily run the MCP server. This setup assumes you have an **existing InfluxDB instance** that the MCP server will connect to.

**Prerequisites:**
- Docker and Docker Compose installed.
- An existing InfluxDB instance (v2.x or compatible) accessible from where you run Docker Compose.

**Configuration:**

1.  **Environment Variables:**
    The MCP server is configured using environment variables. You **must** set these in a `.env` file in the project root directory or provide them when running `docker compose`.

    Create a `.env` file in the project root with the following content:
    ```env
    # .env - Configure to connect to your EXISTING InfluxDB
    INFLUXDB_URL=http://your-influxdb-host:8086
    INFLUXDB_TOKEN=your_influxdb_api_token
    INFLUXDB_ORG=your_influxdb_organization_name

    # Optional: Specify the host port for the MCP server (defaults to 3000)
    # MCP_PORT=3001
    ```
    **Important:**
    - `INFLUXDB_URL`: The full URL of your existing InfluxDB instance (e.g., `http://localhost:8086` if running locally, or `http://influxdb.example.com:8086`).
    - `INFLUXDB_TOKEN`: An API token for your InfluxDB instance that has the necessary permissions for the MCP server to operate (read/write data, manage orgs/buckets if those tools are used).
    - `INFLUXDB_ORG`: The InfluxDB organization the MCP server will primarily interact with.
    - `MCP_PORT` (Optional): If you want to expose the MCP server on a different host port than the default `3000`.

2.  **Network Considerations:**
    If your InfluxDB instance is running in Docker on a custom network, you might need to adjust the `docker-compose.yml` to connect the `mcp-server` service to that same network. See comments in `docker-compose.yml` for guidance. If InfluxDB is running on the host or elsewhere, ensure `INFLUXDB_URL` is resolvable from within the Docker container (e.g., use `host.docker.internal` for host services from Docker Desktop).

**Usage:**

1.  **Start the MCP Server:**
    Navigate to the project root directory (where `docker-compose.yml` and your `.env` file are) and run:
    ```bash
    docker compose up --build
    ```
    Or, to run in detached mode:
    ```bash
    docker compose up --build -d
    ```
    This command builds the MCP server Docker image (if it's not already built or if the `Dockerfile` has changed) and starts the `mcp-server` service.

2.  **Accessing the MCP Server:**
    The MCP server will be accessible at `http://localhost:<MCP_PORT_OR_3000>/mcp`. For example, if `MCP_PORT` is not set in your `.env` file, it will be `http://localhost:3000/mcp`.

3.  **Stopping the MCP Server:**
    To stop the server, press `Ctrl+C` in the terminal where `docker compose up` is running (if not in detached mode), or run:
    ```bash
    docker compose down
    ```

## Integration with Claude for Desktop

Add the server to your `claude_desktop_config.json`:

### Using npx (recommended)

```json
{
  "mcpServers": {
    "influxdb": {
      "command": "npx",
      "args": ["influxdb-mcp-server"],
      "env": {
        "INFLUXDB_TOKEN": "your_token",
        "INFLUXDB_URL": "http://localhost:8086",
        "INFLUXDB_ORG": "your_org"
      }
    }
  }
}
```

### If installed locally

```json
{
  "mcpServers": {
    "influxdb": {
      "command": "node",
      "args": ["/path/to/influxdb-mcp-server/src/index.js"],
      "env": {
        "INFLUXDB_TOKEN": "your_token",
        "INFLUXDB_URL": "http://localhost:8086",
        "INFLUXDB_ORG": "your_org"
      }
    }
  }
}
```

## Code Structure

The server code is organized into a modular structure:

- `src/`
  - `index.js` - Main server entry point
  - `config/` - Configuration related files
    - `env.js` - Environment variable handling
  - `utils/` - Utility functions
    - `influxClient.js` - InfluxDB API client
    - `loggerConfig.js` - Console logger configuration
  - `handlers/` - Resource and tool handlers
    - `organizationsHandler.js` - Organizations listing
    - `bucketsHandler.js` - Buckets listing
    - `measurementsHandler.js` - Measurements listing
    - `queryHandler.js` - Query execution
    - `writeDataTool.js` - Data write tool
    - `queryDataTool.js` - Query tool
    - `createBucketTool.js` - Bucket creation tool
    - `createOrgTool.js` - Organization creation tool
  - `prompts/` - Prompt templates
    - `fluxQueryExamplesPrompt.js` - Flux query examples
    - `lineProtocolGuidePrompt.js` - Line protocol guide

This structure allows for better maintainability, easier testing, and clearer separation of concerns.

## Testing

The repository includes comprehensive integration tests that:

- Spin up a Docker container with InfluxDB
- Populate it with sample data
- Test all MCP server functionality

To run the tests:

```bash
npm test
```

## License

MIT
