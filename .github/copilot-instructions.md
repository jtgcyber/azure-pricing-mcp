# Copilot Instructions

Azure Pricing MCP Server: a Model Context Protocol (MCP) server exposing tools that query the public
[Azure Retail Prices API](https://prices.azure.com/api/retail/prices). No Azure auth is required; the API is public.

## Setup & running

```powershell
# Setup (creates .venv and installs requirements.txt)
.\setup.ps1            # Windows PowerShell (primary)
python setup.py        # cross-platform

# Run the server (stdio MCP transport)
python -m azure_pricing_server
```

This is a Windows-first repo: PowerShell scripts (`setup.ps1`, `test_setup.ps1`) are the primary tooling.
The server speaks MCP over stdio — it is launched by an MCP client (Claude Desktop / VS Code), not run interactively.

## Testing

There is no pytest suite or test runner. Tests are standalone ad-hoc scripts run directly, each hitting the live
Azure API:

```powershell
python test_mcp.py          # exercises search_azure_prices directly
python test_mcp_server.py   # invokes handle_call_tool for azure_price_search
.\test_setup.ps1            # verifies venv + connectivity
```

Note: `project.json` scripts (`test_server.py`, `test_api.py`) and the README's `--test` flag reference
entry points that do not exist — prefer the scripts above. The `debug_*.py`, `simulate_mcp_call.py`,
`find_app_service.py`, and `exact_mcp_handler_test.py` files are throwaway debugging scripts.

## Architecture

Nearly all logic lives in `azure_pricing_server.py` (~1400 lines). Two module-level singletons drive everything:

- `server = Server("azure-pricing")` — the MCP server. Tools are declared in `handle_list_tools()`
  (`@server.list_tools()`) and dispatched by name in `handle_call_tool()` (`@server.call_tool()`).
- `pricing_server = AzurePricingServer()` — holds the `aiohttp` session and all business logic. It is an
  async context manager; `handle_call_tool` wraps every call in `async with pricing_server:` to open/close
  the HTTP session per request.

Adding/changing a tool means editing in three places: the `Tool(...)` schema in `handle_list_tools()`, an
`elif name == ...` branch in `handle_call_tool()`, and the method on `AzurePricingServer`.

`__main__.py` is the module entry point; it calls `azure_pricing_server.main()`, which runs the server over
`stdio_server()`.

## Key conventions

- **Querying Azure**: filters are built as OData `$filter` strings (`" and ".join(filter_conditions)`) and
  passed to `_make_request`, which has built-in retry/backoff on HTTP 429. Always wrap user-supplied filter
  values with `_escape_odata_literal()` to avoid OData injection.
- **Pagination**: the Retail Prices API ignores `$top` and returns ~1000 items per page with a `NextPageLink`.
  Use `AzurePricingServer._paginate()` (not a single `_make_request`) to fetch complete results; it caps at
  `MAX_PAGES` and returns `(items, truncated)`. Reading only the first page silently drops data.
- **ARM vs display SKU names**: `armSkuName` holds the ARM name (e.g. `Standard_D2s_v3`); `skuName` is a
  display name (e.g. `D2s v3`). A meter's identity is `meterName` + `productName` + `unitOfMeasure`; rows
  differing only by `tierMinimumUnits` are pricing tiers, not separate meters.
- **Discounts are opt-in**: pricing reflects Azure's actual retail prices by default. `azure_price_search`
  and `azure_price_architecture` only apply a discount when `discount_percentage` is passed. (`get_customer_discount`
  remains a hardcoded-10% stub but is no longer auto-applied.)
- **`azure_price_architecture`** is the deterministic bill engine: it does unit-aware math and sums totals in
  code, and marks line items `ambiguous`/`not_found` rather than guessing. The agent owns architecture
  interpretation and usage quantities; the MCP owns price lookup and arithmetic.
- Tool/method results are returned as `list[TextContent]`; errors are caught in `handle_call_tool` and
  returned as text rather than raised.
- The API version is pinned to `2023-01-01-preview` (`DEFAULT_API_VERSION`) because it is required for
  savings-plan data.
