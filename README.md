# Azure Pricing MCP Server

[![Docker Hub](https://img.shields.io/docker/v/quintindk/azure-pricing-mcp?label=docker&logo=docker)](https://hub.docker.com/r/quintindk/azure-pricing-mcp)
[![Docker Pulls](https://img.shields.io/docker/pulls/quintindk/azure-pricing-mcp)](https://hub.docker.com/r/quintindk/azure-pricing-mcp)
[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Model Context Protocol (MCP) server that provides programmatic access to Azure resource pricing information. Query Azure pricing data through a simple, structured workflow using the Azure Retail Prices API.

**Perfect for**: Cost estimation, pricing comparisons, budget planning, and integrating Azure pricing into your AI workflows.

## Features

- 🚀 Query Azure pricing data through a simple, structured workflow
- 💰 Get real-time pricing information from the Azure Retail Prices API
- 🔍 Navigate through Azure service families, service names, and products
- 📊 Calculate monthly costs for Azure resources
- 🐳 Available as a Docker image for easy deployment
- 🔓 No Azure account or credentials required (uses public pricing API)

## Quick Start with Docker

The easiest way to get started is using Docker:

```bash
# Pull the latest image
docker pull quintindk/azure-pricing-mcp:latest

# Run with stdio transport (for MCP client integration)
docker run --rm -e MCP_TRANSPORT=stdio quintindk/azure-pricing-mcp:latest

# Run with HTTP transport (for web-based access)
docker run --rm -p 8080:8080 -e MCP_TRANSPORT=http quintindk/azure-pricing-mcp:latest
```

The server will start and be ready to accept MCP requests through your configured client.

## MCP Client Configuration

Configure your MCP client to connect to this server. The configuration depends on the transport mode:

### For HTTP/SSE Transport

Add to your MCP client configuration:

```json
{
  "azure-pricing": {
    "serverUrl": "http://localhost:8080/sse"
  }
}
```

### For stdio Transport (Docker)

Add to your MCP client configuration:

```json
{
  "azure-pricing": {
    "command": "docker",
    "args": ["run", "--rm", "-i", "-e", "MCP_TRANSPORT=stdio", "quintindk/azure-pricing-mcp:latest"]
  }
}
```

## Alternative: Python Installation

If you prefer to run from source:

1. **Clone the repository:**
   ```bash
   git clone https://github.com/quintindk/mcp-azure-pricing.git
   cd mcp-azure-pricing
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python -m venv .venv
   ```
   * Windows:
     ```bash
     .venv\Scripts\activate
     ```
   * macOS/Linux:
     ```bash
     source .venv/bin/activate
     ```

3. **Install the dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the server:**
   ```bash
   python azure_pricing_mcp_server.py
   ```

## How It Works

The MCP server provides a structured four-step workflow for accessing Azure pricing information:

1. **Get service families** - Retrieve the list of available Azure service families
2. **Get service names** - Get service names within a specific family
3. **Get products** - Get products associated with a service
4. **Calculate monthly costs** - Calculate the monthly cost for a specific product

### Available Endpoints (HTTP Mode)

When running in HTTP mode, the server exposes:

- `GET /sse`: Server-Sent Events endpoint for MCP communication
- `GET /tools`: Lists the available tools in the MCP server

## MCP Tools Reference

The server provides four main tools that form a logical workflow for querying Azure pricing:

### 1. list_service_families

**Description**: Lists all available service families in Azure according to Microsoft's official documentation.

### 2. get_service_names

**Description**: Gets all unique service names within a specified service family.

**Parameters**:
- `service_family`: The service family to query (e.g., 'Compute', 'Storage')
- `region`: Azure region (default: 'westeurope')
- `max_results`: Maximum number of results to process

### 3. get_products

**Description**: Gets product names from a specific service family.

**Parameters**:
- `service_family`: The service family to query
- `region`: Azure region (default: 'westeurope')
- `type`: Price type (optional, e.g., 'Consumption', 'Reservation')
- `service_name`: Service name to filter by (optional)
- `product_name_contains`: Filter products whose name contains this text (optional)
- `limit`: Maximum number of products to return (optional)

### 4. get_monthly_cost

**Description**: Calculates the monthly cost of a specific Azure product.

**Parameters**:
- `product_name`: Exact name of the product (e.g., 'Azure App Service Premium v3 Plan')
- `region`: Azure region (default: 'westeurope')
- `monthly_hours`: Number of hours per month (default: 730)
- `type`: Price type (optional, e.g., 'Consumption')

## Error Handling

The MCP server includes a robust error handling system that:

- Provides descriptive error messages when resources cannot be found
- Properly handles Azure API errors
- Logs detailed information for debugging purposes

### Common Error Scenarios

- **Product not found**: When a product name doesn't exist in the specified region
- **Service family not found**: When an invalid service family is specified
- **API rate limits**: When the Azure Retail Prices API rate limits are exceeded
- **Network errors**: When the server cannot connect to the Azure API

## Limitations

* Prices are estimates based on public information from the Azure Retail Prices API
* Does not include all possible discounts, account-specific offers, or additional costs like taxes or support
* The Azure Retail Prices API has rate limits that can affect performance with a high volume of requests
* Prices may vary depending on the region and currency selected
* Not all Azure resources are available in all regions

## Building from Source

If you want to build your own Docker image:

```bash
# Clone the repository
git clone https://github.com/quintindk/mcp-azure-pricing.git
cd mcp-azure-pricing

# Build the Docker image
docker build -t azure-pricing-mcp:local .

# Run your local build
docker run --rm -p 8080:8080 -e MCP_TRANSPORT=http azure-pricing-mcp:local
```

The Dockerfile is optimized for both `amd64` and `arm64` architectures and includes multi-stage caching for faster builds.

## Contributing

Contributions are welcome! Here's how you can contribute to this project:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add some amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.