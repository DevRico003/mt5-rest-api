# MT5 REST API

Turns MetaTrader 5 into a REST API server for algorithmic trading.

## Requirements

- MetaTrader 5 (64-bit)
- Visual C++ Redistributable 2017: https://aka.ms/vs/15/release/vc_redist.x64.exe

## Installation

1. Clone repo to any folder on your PC
2. Copy `MQL5` folder to MT5 Data folder (`File → Open Data Folder` or `Ctrl+Shift+D`)
3. Copy all DLLs from `MQL5/Libraries/` to the same location
4. In MT5 Navigator (`Ctrl+N`), right-click `Expert Advisors → RestApi → Modify`
5. Press `F7` to compile
6. Drag EA onto a chart

## EA Configuration

| Parameter | Description | Example |
|-----------|-------------|---------|
| host | Listen address | `http://YOUR_SERVER_IP` |
| port | Port number | `6542` |
| AuthToken | Authentication token | `your-secret-token` |

**Important:** On Windows, use your server IP instead of `http://0.0.0.0`

## API Endpoints

All endpoints require `Authorization` header with your token.

### Account & Info

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/info` | GET | Account details, balance, equity, margin |
| `/balance` | GET | Balance, equity, margin info |
| `/symbols/{name}` | GET | Symbol info including ask/bid prices |

### Positions & Orders

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/positions` | GET | List all open positions |
| `/positions/{id}` | GET | Get position by ID |
| `/orders` | GET | List pending orders |
| `/orders/{id}` | GET | Get order by ID |
| `/history` | GET | List order history |
| `/history/{id}` | GET | Get history order by ID |
| `/deals` | GET | List deals (query: `offset`, `limit`) |
| `/deals/{id}` | GET | Get deal by ID |

### Historical Data (OHLCV)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/candles/{symbol}` | GET | Historical candlestick data |

**Query Parameters:**
- `timeframe`: M1, M5, M15, M30, H1, H4, D1, W1, MN1 (default: H1)
- `count`: Number of candles, 1-1000 (default: 100)
- `start_pos`: Start position, 0 = current candle (default: 0)

**Example:**
```bash
curl -H "Authorization: your-token" "http://localhost:6542/candles/EURGBP?timeframe=H1&count=100"
```

**Response:**
```json
{
  "symbol": "EURGBP",
  "timeframe": "H1",
  "count": 100,
  "candles": [
    {
      "time": "2025-12-02T22:00:00.000Z",
      "open": 0.87971,
      "high": 0.87994,
      "low": 0.87966,
      "close": 0.87985,
      "tick_volume": 1241,
      "spread": 0,
      "real_volume": 0
    }
  ]
}
```

### Trading

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/trade` | POST | Execute trading actions |

## Trade Examples

### Open Buy
```json
{
  "symbol": "EURUSD",
  "actionType": "ORDER_TYPE_BUY",
  "volume": 0.1,
  "stoploss": 1.09000,
  "takeprofit": 1.11000,
  "comment": "Buy order"
}
```

### Open Sell
```json
{
  "symbol": "EURUSD",
  "actionType": "ORDER_TYPE_SELL",
  "volume": 0.1,
  "stoploss": 1.11000,
  "takeprofit": 1.09000,
  "comment": "Sell order"
}
```

### Pending Orders
```json
// Buy Limit
{ "symbol": "EURUSD", "actionType": "ORDER_TYPE_BUY_LIMIT", "price": 1.08000, "volume": 0.1, "stoploss": 1.07000, "takeprofit": 1.10000 }

// Sell Limit
{ "symbol": "EURUSD", "actionType": "ORDER_TYPE_SELL_LIMIT", "price": 1.12000, "volume": 0.1, "stoploss": 1.13000, "takeprofit": 1.10000 }

// Buy Stop
{ "symbol": "EURUSD", "actionType": "ORDER_TYPE_BUY_STOP", "price": 1.12000, "volume": 0.1, "stoploss": 1.11000, "takeprofit": 1.14000 }

// Sell Stop
{ "symbol": "EURUSD", "actionType": "ORDER_TYPE_SELL_STOP", "price": 1.08000, "volume": 0.1, "stoploss": 1.09000, "takeprofit": 1.06000 }
```

### Position Management
```json
// Close position by ID
{ "actionType": "POSITION_CLOSE_ID", "id": 123456789 }

// Partial close
{ "actionType": "POSITION_PARTIAL", "id": 123456789, "volume": 0.05 }

// Modify position
{ "actionType": "POSITION_MODIFY", "id": 123456789, "stoploss": 1.08500, "takeprofit": 1.11500 }

// Cancel pending order
{ "actionType": "ORDER_CANCEL", "id": 123456789 }
```

## Trade Response

```json
{
  "error": 10009,
  "description": "TRADE_RETCODE_DONE",
  "order_id": 405895526,
  "volume": 0.1,
  "price": 1.13047,
  "bid": 1.13038,
  "ask": 1.13047,
  "function": "CRestApi::tradingModule"
}
```

**Common Return Codes:**
- `10009` - TRADE_RETCODE_DONE (Success)
- `10018` - TRADE_RETCODE_MARKET_CLOSED
- `10016` - TRADE_RETCODE_INVALID_STOPS
- `10019` - TRADE_RETCODE_NO_MONEY

Full list: https://www.mql5.com/en/docs/constants/errorswarnings/enum_trade_return_codes

## Building from Source

To compile the C++ DLL:
1. Open `mt5-rest.sln` in Visual Studio 2017+
2. Select `Release x64`
3. Build (`Ctrl+Shift+B`)
4. Output: `/x64/Release/mt5-rest.dll`

## References

- DateTime format: https://www.mql5.com/en/docs/basis/types/integer/datetime
- Enums: https://www.mql5.com/en/docs/constants
- Trade codes: https://www.mql5.com/en/docs/constants/errorswarnings/enum_trade_return_codes
