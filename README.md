# Azure Pricing MCP Server 💰

A Model Context Protocol (MCP) server that provides tools for querying Azure retail pricing information using the Azure Retail Prices API.

## 🚀 Quick Start

1. **Clone/Download** this repository
2. **Run setup**: `setup.ps1` (Windows PowerShell) or `python setup.py` (Cross-platform)
3. **Configure Claude Desktop** (see [QUICK_START.md](QUICK_START.md))
4. **Ask Claude**: "What's the price of a Standard_D2s_v3 VM in East US?"

## ✨ Features

- **🔍 Azure Price Search**: Search for Azure service prices with flexible filtering
- **⚖️ Service Comparison**: Compare prices across different regions and SKUs
- **💡 Cost Estimation**: Calculate estimated costs based on usage patterns
- **💰 Savings Plan Information**: Get Azure savings plan pricing when available
- **🌍 Multi-Currency**: Support for multiple currencies (USD, EUR, etc.)
- **📊 Real-time Data**: Uses live Azure Retail Prices API

## 🛠️ Tools Available

| Tool | Description | Example Use |
|------|-------------|-------------|
| `azure_price_search` | Search Azure retail prices with filters | Find VM prices in specific regions |
| `azure_price_compare` | Compare prices across regions/SKUs | Compare storage costs across regions |
| `azure_cost_estimate` | Estimate costs based on usage | Calculate monthly costs for 8hr/day usage |
| `azure_discover_skus` | Discover available SKUs for a service | Find all VM types for a service |
| `azure_sku_discovery` | Intelligent SKU discovery with fuzzy matching | "Find app service plans" or "web app pricing" |
| `azure_price_architecture` | Price a whole architecture/app as one monthly bill | Estimate monthly cost of a multi-service application |

## 📋 Installation

### Automated Setup (Recommended)
```bash
# Windows PowerShell
.\setup.ps1

# Cross-platform (Python)
python setup.py
```

### Manual Setup
```bash
# Create virtual environment
python -m venv .venv

# Activate virtual environment
.venv\Scripts\activate  # Windows
source .venv/bin/activate  # Linux/Mac

# Install dependencies
pip install -r requirements.txt
```

## 🔧 Configuration

Add to your Claude Desktop config file:

```json
{
  "mcpServers": {
    "azure-pricing": {
      "command": "python",
      "args": ["-m", "azure_pricing_server"],
      "cwd": "/path/to/azure_pricing"
    }
  }
}
```

## 💬 Example Queries

Once configured with Claude, you can ask:

- **Basic Pricing**: "What's the price of Azure SQL Database?"
- **Comparisons**: "Compare VM prices between East US and West Europe"
- **Cost Estimation**: "Estimate costs for running a D4s_v3 VM 12 hours per day"
- **Savings**: "What are the reserved instance savings for virtual machines?"
- **GPU Pricing**: "Show me all GPU-enabled VMs with pricing"
- **Service Discovery**: "Find all App Service plan pricing" or "What storage options are available?"
- **SKU Discovery**: "Show me all web app hosting plans"
- **Architecture Pricing**: "Estimate the monthly bill for a D2s_v3 VM plus 500 GB of Hot LRS blob storage"

## 🧪 Testing

Test setup and connectivity:
```bash
# Windows PowerShell
.\test_setup.ps1

# Cross-platform test
python -m azure_pricing_server --test
```

## 📚 Documentation

- **[QUICK_START.md](QUICK_START.md)** - Step-by-step setup guide
- **[USAGE_EXAMPLES.md](USAGE_EXAMPLES.md)** - Detailed usage examples and API responses
- **[config_examples.json](config_examples.json)** - Example configurations for Claude Desktop and VS Code

## 🔌 API Integration

This server uses the official Azure Retail Prices API:
- **Endpoint**: `https://prices.azure.com/api/retail/prices`
- **Version**: `2023-01-01-preview` (supports savings plans)
- **Authentication**: None required (public API)
- **Rate Limits**: Generous limits for retail pricing data

## 🌟 Key Features

### Smart Filtering
- Filter by service name, family, region, SKU
- Support for partial matches and contains operations
- Case-sensitive filtering for precise results

### Cost Optimization
- Automatic savings plan detection
- Reserved instance pricing comparisons
- Multi-region cost analysis
- Intelligent SKU discovery for finding the best pricing options

### Architecture / Application Pricing
The `azure_price_architecture` tool turns a list of line items into a single
monthly bill, designed to be driven by an agent pricing a whole application:

- **Deterministic math** – subtotals (`unit_price × quantity ÷ unit_multiplier`)
  and monthly/yearly totals are computed in code, not by the model, so the
  arithmetic is reliable across many line items.
- **Unit-aware** – the per-unit block in `unitOfMeasure` (e.g. `"100 Hours"`,
  `"10K"`) is parsed so storage, bandwidth, and per-hour meters all price
  correctly. `quantity` is given in the meter's base unit (hours, GB, etc.).
- **Tier-aware** – graduated meters select the tier applicable to the quantity.
- **Never guesses** – line items matching multiple distinct meters are returned
  as `ambiguous` with candidate meters (and excluded from the total); resolve
  them with `meter_name`, `product_name`, or `unit`. Use `arm_sku_name` to match
  resources by their ARM name (e.g. `Standard_D2s_v3`).

The agent owns architecture interpretation, SKU selection, and usage
assumptions; the MCP owns accurate price lookup and the math.

### Developer Friendly
- Comprehensive error handling
- Detailed logging for troubleshooting
- Flexible parameter support
- Cross-platform setup scripts (PowerShell and Python)

## 🤝 Contributing

This project follows the Spec-Driven Development (SDD) methodology. Contributions are welcome!

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details

## 🙋‍♂️ Support

- Check [QUICK_START.md](QUICK_START.md) for setup issues
- Review [USAGE_EXAMPLES.md](USAGE_EXAMPLES.md) for query patterns
- Open an issue for bugs or feature requests

---

*Built with the Model Context Protocol (MCP) for seamless integration with Claude and other AI assistants.*