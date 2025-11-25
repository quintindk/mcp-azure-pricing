# Docker Setup for Azure Pricing MCP Server

This document explains how to run the Azure Pricing MCP Server in a Docker container using stdio transport.

## Prerequisites

- Docker installed on your system
- Docker Compose (optional, but recommended)

## Building the Docker Image

### Using Docker

```bash
docker build -t azure-pricing-mcp:latest .
```

### Using Docker Compose

```bash
docker-compose build
```

## Running the Server

### Using Docker

```bash
docker run -i --rm \
  -e MCP_DEBUG=false \
  -e LOG_LEVEL=INFO \
  azure-pricing-mcp:latest
```

### Using Docker Compose

```bash
docker-compose up
```

To run in detached mode:

```bash
docker-compose up -d
```

## Environment Variables

You can configure the server using environment variables:

| Variable | Description | Default | Options |
|----------|-------------|---------|---------|
| `MCP_TRANSPORT` | Transport type for the MCP server | `stdio` | `stdio`, `http`, `sse` |
| `MCP_DEBUG` | Enable debug logging | `false` | `true`, `false` |
| `LOG_LEVEL` | Logging level | `INFO` | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |
| `MCP_HOST` | Host address (HTTP/SSE only) | `0.0.0.0` | Any valid IP |
| `MCP_PORT` | Port number (HTTP/SSE only) | `8080` | 1-65535 |
| `AZURE_RETAIL_PRICES_URL` | Azure Retail Prices API URL | `https://prices.azure.com/api/retail/prices` | Any valid URL |
| `AZURE_API_VERSION` | Azure API version | `2023-01-01-preview` | API version string |

### Setting Environment Variables

#### With Docker (stdio transport)

```bash
docker run -i --rm \
  -e MCP_TRANSPORT=stdio \
  -e MCP_DEBUG=true \
  -e LOG_LEVEL=DEBUG \
  azure-pricing-mcp:latest
```

#### With Docker (HTTP transport)

```bash
docker run --rm \
  -e MCP_TRANSPORT=http \
  -e MCP_DEBUG=false \
  -e MCP_HOST=0.0.0.0 \
  -e MCP_PORT=8080 \
  -p 8080:8080 \
  azure-pricing-mcp:latest
```

#### With Docker Compose

Edit the `docker-compose.yml` file or create a `.env` file:

```env
MCP_TRANSPORT=stdio
MCP_DEBUG=false
LOG_LEVEL=INFO
```

## Using with Claude Desktop

To use this Docker container with Claude Desktop, you need to configure it in your Claude Desktop config file.

### macOS

Edit `~/Library/Application\ Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "azure-pricing": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e", "MCP_DEBUG=false",
        "-e", "LOG_LEVEL=INFO",
        "azure-pricing-mcp:latest"
      ]
    }
  }
}
```

### Windows

Edit `%APPDATA%/Claude/claude_desktop_config.json` with the same configuration as above.

### Linux

Edit `~/.config/Claude/claude_desktop_config.json` with the same configuration as above.

## Troubleshooting

### Viewing Logs

Since the server communicates via stdio, logs are sent to stderr. You can view them using:

```bash
docker-compose logs -f
```

Or with Docker:

```bash
docker logs <container-id>
```

### Debug Mode

To enable debug mode for more verbose logging:

```bash
docker run -i --rm \
  -e MCP_DEBUG=true \
  -e LOG_LEVEL=DEBUG \
  azure-pricing-mcp:latest
```

### Rebuilding the Image

If you make changes to the code, rebuild the image:

```bash
docker-compose build --no-cache
```

## Available Tools

The server provides the following tools:

1. **list_service_families** - List all available Azure service families
2. **get_service_names** - Get service names within a service family
3. **get_products** - Get product names from a specific service family
4. **get_monthly_cost** - Calculate the monthly cost of a specific Azure product

## Architecture

- **Dockerfile**: Defines the container image with Python 3.11 and all dependencies
- **docker-compose.yml**: Provides easy container orchestration
- **azure_pricing_mcp_server.py**: Unified server supporting multiple transports (stdio, HTTP/SSE) with all tool implementations inline
- **config.py**: Configuration management with environment variable support
- **.dockerignore**: Excludes unnecessary files from the Docker build

The server supports configurable transport via the `MCP_TRANSPORT` environment variable (stdio, http, or sse).

## Security

The container runs as a non-root user (`mcpuser`) for security best practices.

## Notes

- The server uses stdio transport, which means it communicates via stdin/stdout
- All logs are sent to stderr to avoid interfering with stdio communication
- The container is configured with `stdin_open: true` and `tty: true` to support stdio
- No ports are exposed since communication happens via stdio
