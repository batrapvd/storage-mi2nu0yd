# 🤖 AI PROMPT TEMPLATES FOR BUILDING XAUUSD TRADING BOT

## HOW TO USE THIS DOCUMENT
Copy each prompt below to an AI assistant (Claude, GPT-4, etc.) to build each component of the trading system. Follow the order provided for best results.

---

## PROMPT 1: Create Windows Installation Script

```
I need you to create a Windows batch script (install.bat) and a Python script (setup.py) for setting up an automated trading bot on Windows.

Requirements:
1. Check if Python 3.9+ is installed
2. Create virtual environment named "venv"
3. Install these packages: MetaTrader5, fastapi, uvicorn[standard], redis, httpx, pandas, numpy, websockets, google-generativeai, python-dotenv
4. Create folder structure: logs/, data/, config/
5. Generate a default config.json file
6. Test MT4/MT5 connection
7. Display success message with next steps

The script should handle errors gracefully and provide clear error messages if something fails.

Also create requirements.txt with exact versions.
```

---

## PROMPT 2: Build MT4 Connector Module

```
Create a Python module (mt4_connector.py) that connects to MetaTrader 5 and provides trading functionality.

Class: MT4Bridge

Required Methods:
1. __init__(): Initialize connection to MT5
2. get_account_info(): Return balance, equity, margin, etc.
3. get_market_data(symbol="XAUUSD"): Get current price, spread
4. get_ohlc_bars(timeframe, count): Get historical bars
5. calculate_indicators(data): Calculate RSI, SMA, MACD
6. place_order(order_type, volume=0.01, sl=None, tp=None): Place market order
7. close_position(ticket): Close specific position
8. modify_position(ticket, sl=None, tp=None): Modify SL/TP
9. get_positions(): Get all open positions
10. get_history(days=7): Get trade history

Include:
- Error handling for connection loss
- Automatic reconnection logic
- Validation for all parameters
- Logging for all operations
- Symbol specifications for XAUUSD

The module should be production-ready with proper exception handling.
```

---

## PROMPT 3: Create FastAPI Backend Server

```
Create a FastAPI server (main.py) that provides REST API and WebSocket for the trading bot.

Features needed:
1. Connect to MT4 using mt4_connector module
2. Connect to Redis (URL: redis://default:AYEXAAIncDJmMGM2Y2FmZjRmOWQ0YTliOTRhMzkzZTNiMTI3NTY4OHAyMzMwNDc@smart-redfish-33047.upstash.io:6379)
3. Background task that scans market every 10 seconds
4. WebSocket for real-time updates

API Endpoints:
- GET /api/account - Account information
- GET /api/market/data - Current XAUUSD market data
- GET /api/market/indicators - Technical indicators
- GET /api/positions - Open positions
- GET /api/trades/history - Trade history
- POST /api/trade/place - Place new order
- POST /api/trade/close/{ticket} - Close position
- POST /api/trade/modify/{ticket} - Modify position
- GET /api/stats/today - Today's statistics
- GET /api/config - Get configuration
- POST /api/config - Update configuration
- WS /ws - WebSocket for real-time data

Background Scanner:
- Runs every 10 seconds
- Fetches XAUUSD data
- Calculates indicators (RSI, SMA 20/50/200, MACD)
- Stores in Redis with keys:
  - market:latest
  - market:history (list, max 1440 items)
  - indicators:latest
- Broadcasts via WebSocket

Include CORS middleware for frontend access.
```

---

## PROMPT 4: Build AI Trading Agent

```
Create an AI trading agent (ai_agent.py) that makes trading decisions using Gemini and OpenRouter APIs.

Class: AITradingAgent

Features:
1. Support multiple AI models (Gemini, OpenRouter)
2. Automatic model rotation on API limits
3. Generate trading decisions based on market analysis
4. Risk management validation

Methods:
1. __init__(redis_client): Initialize with Redis connection
2. analyze_market(data): Analyze market conditions
3. build_prompt(market_data, account_info, positions): Create detailed prompt
4. call_gemini(prompt, api_key): Call Gemini API
5. call_openrouter(prompt, api_key, model): Call OpenRouter API
6. get_decision(market_data, account_info): Get trading decision
7. validate_decision(decision): Validate AI response
8. execute_decision(decision): Send to MT4

AI Prompt Structure:
- Current XAUUSD price and spread
- Technical indicators (RSI, SMA, MACD)
- Support/Resistance levels
- Account balance and equity
- Open positions (max 3)
- Risk rules (0.01 lot, max daily loss $50)
- Request JSON response with: action, confidence, sl, tp, reasoning

Decision Validation:
- Confidence must be >70%
- Risk/Reward ratio minimum 1:2
- Check position limits
- Verify margin requirements

Include comprehensive error handling and logging.
```

---

## PROMPT 5: Create Risk Manager

```
Create a risk management module (risk_manager.py) for the trading bot.

Class: RiskManager

Core Functions:
1. Check if trading is allowed based on:
   - Daily loss limit ($50)
   - Consecutive losses (max 3)
   - Number of open positions (max 3)
   - Account drawdown
   - Margin level

2. Position Management:
   - Calculate stop loss (50 points default)
   - Calculate take profit (100 points default)
   - Validate risk/reward ratio (min 1:2)
   - Apply trailing stop (activate at 30 points profit, trail by 20 points)

3. Statistics Tracking:
   - Daily P/L
   - Win rate
   - Average win/loss
   - Consecutive wins/losses
   - Maximum drawdown

Methods:
- can_trade(account_info, positions, stats): Return bool with reason
- calculate_position_size(account_balance): Return 0.01 (fixed)
- calculate_stops(entry_price, direction): Return SL and TP
- should_apply_trailing(position, current_price): Check if trailing needed
- update_statistics(trade_result): Update Redis stats
- get_risk_metrics(): Return current risk status

Store all data in Redis with appropriate keys and TTL.
```

---

## PROMPT 6: Build Redis Data Manager

```
Create a Redis client module (redis_client.py) for data management.

Class: RedisManager

Connection:
- URL: redis://default:AYEXAAIncDJmMGM2Y2FmZjRmOWQ0YTliOTRhMzkzZTNiMTI3NTY4OHAyMzMwNDc@smart-redfish-33047.upstash.io:6379
- Use SSL/TLS
- Connection pooling
- Auto-reconnect

Data Operations:
1. Market Data:
   - save_market_snapshot(data): Store with timestamp
   - get_latest_market(): Get current data
   - get_market_history(count): Get historical data
   
2. Positions:
   - save_position(position_data): Store open position
   - get_open_positions(): Get all open positions
   - remove_position(ticket): Remove closed position
   
3. Trading:
   - save_trade(trade_data): Store completed trade
   - get_trade_history(period): Get trade history
   - update_daily_stats(stats): Update statistics
   
4. Configuration:
   - get_config(key): Get configuration value
   - save_config(key, value): Store configuration
   - get_ai_models(): Get AI model configs
   
5. Real-time:
   - publish_event(channel, data): Publish to channel
   - subscribe(channel): Subscribe to updates

Key Structure:
- market:latest - Current market data (JSON)
- market:history - Historical data (List)
- positions:open - Open positions (Set)
- position:{ticket} - Position details (Hash)
- trades:history - Trade history (List)
- stats:daily:{date} - Daily statistics (Hash)
- config:trading - Trading parameters (Hash)
- config:ai:models - AI models (List)

Include error handling and data validation.
```

---

## PROMPT 7: Create Windows Startup Script

```
Create a Windows batch script (start_bot.bat) and Python launcher (launcher.py) to start the trading bot.

start_bot.bat should:
1. Display ASCII art logo
2. Check if virtual environment exists
3. Activate virtual environment
4. Check if MT5 is running
5. Test Redis connection
6. Display current configuration
7. Start the FastAPI server with uvicorn
8. Handle graceful shutdown

launcher.py should:
1. Verify all dependencies are installed
2. Check MT5 connection
3. Test Redis connectivity
4. Load and validate configuration
5. Initialize all modules
6. Start background tasks:
   - Market scanner (10-second interval)
   - AI decision maker
   - Risk monitor
   - WebSocket broadcaster
7. Start FastAPI with proper host/port
8. Display real-time logs
9. Handle Ctrl+C for clean shutdown

Include color-coded console output for different log levels.
```

---

## PROMPT 8: Build Frontend Dashboard

```
Create a Next.js 14 dashboard for the trading bot with TypeScript and Tailwind CSS.

Project Structure:
/app
  /dashboard
    page.tsx - Main dashboard
  /config
    page.tsx - Configuration page
  /api
    [...route].ts - API routes
/components
  /charts
    TradingChart.tsx - Real-time price chart
    PnLChart.tsx - Profit/Loss chart
  /trading
    PositionsList.tsx - Open positions
    QuickTrade.tsx - Manual trade buttons
    TradeHistory.tsx - Historical trades
  /config
    AIModelConfig.tsx - AI model settings
    RiskSettings.tsx - Risk parameters
  /shared
    WebSocketProvider.tsx - WS connection
    StatsCards.tsx - Metric cards

Features:
1. Real-time XAUUSD chart with candlesticks
2. Live position monitoring with P/L
3. Account balance and metrics
4. AI decision log with reasoning
5. Manual trade execution
6. Configuration management
7. WebSocket for live updates

Use:
- lightweight-charts for charting
- shadcn/ui for components
- react-query for data fetching
- zustand for state management
- sonner for notifications

WebSocket Integration:
- Auto-reconnect logic
- Real-time price updates
- Position change notifications
- AI decision alerts

The dashboard should be responsive and support dark mode.
```

---

## PROMPT 9: Create Docker Setup for Frontend

```
Create Docker configuration for the frontend dashboard.

Files needed:
1. Dockerfile
2. docker-compose.yml
3. .dockerignore
4. nginx.conf (for production)

Dockerfile:
- Use node:20-alpine
- Multi-stage build for optimization
- Build Next.js app
- Expose port 3000

docker-compose.yml:
- Frontend service
- Environment variables for API URL
- Volume for persistent data
- Restart policy

Environment Configuration:
NEXT_PUBLIC_API_URL=http://[WINDOWS_IP]:8000
NEXT_PUBLIC_WS_URL=ws://[WINDOWS_IP]:8000/ws

Production considerations:
- Use nginx as reverse proxy
- Enable gzip compression
- Add health checks
- Configure logging

Include instructions for:
- Local development
- Production deployment
- Environment variable management
```

---

## PROMPT 10: Create Comprehensive Configuration System

```
Create a configuration management system (config_manager.py) with UI components.

Backend (Python):
1. ConfigManager class:
   - Load from JSON file
   - Save to Redis
   - Validate all parameters
   - Hot-reload support

2. Configuration Structure:
   {
     "trading": {
       "symbol": "XAUUSD",
       "lot_size": 0.01,
       "max_positions": 3,
       "max_daily_loss": 50,
       "consecutive_loss_limit": 3,
       "check_interval": 10,
       "trailing_stop_activation": 30,
       "trailing_stop_distance": 20
     },
     "risk": {
       "min_risk_reward": 2.0,
       "default_sl_points": 50,
       "default_tp_points": 100,
       "max_spread": 30
     },
     "ai": {
       "confidence_threshold": 70,
       "models": [
         {
           "provider": "gemini",
           "model": "gemini-pro",
           "api_key": "",
           "priority": 1,
           "enabled": true
         }
       ]
     }
   }

Frontend (React):
1. Configuration UI with forms for:
   - Trading parameters (number inputs with validation)
   - Risk settings (sliders with real-time preview)
   - AI model management (add/edit/delete/reorder)
   - API key secure input (masked, encrypted storage)

2. Features:
   - Real-time validation
   - Reset to defaults
   - Import/Export configuration
   - Configuration history
   - Rollback capability

Include WebSocket updates when configuration changes.
```

---

## PROMPT 11: Create Testing and Monitoring System

```
Create a comprehensive testing and monitoring system.

1. Unit Tests (test_suite.py):
   - Test MT4 connection
   - Test order placement
   - Test risk calculations
   - Test AI prompt generation
   - Test Redis operations

2. Integration Tests:
   - End-to-end trading flow
   - WebSocket communication
   - AI decision pipeline
   - Risk management triggers

3. Paper Trading Mode:
   - Simulate trades without real money
   - Track virtual balance
   - Compare with real market
   - Performance metrics

4. Monitoring Dashboard:
   - System health status
   - API response times
   - Error rate tracking
   - Resource usage
   - Trading performance

5. Logging System:
   - Structured logging with levels
   - File rotation
   - Real-time log streaming
   - Error alerting

6. Metrics Collection:
   - Trade execution time
   - AI decision latency
   - Market data freshness
   - WebSocket connection stability

Create both Python backend monitoring and frontend dashboard components.
```

---

## PROMPT 12: Create Complete Documentation

```
Create comprehensive documentation for the trading bot system.

1. README.md:
   - Project overview
   - Quick start guide
   - Architecture diagram
   - Technology stack
   - License and disclaimer

2. SETUP_GUIDE.md:
   - Prerequisites
   - Step-by-step installation
   - Configuration guide
   - First run instructions
   - Troubleshooting

3. API_DOCUMENTATION.md:
   - All endpoints with examples
   - WebSocket events
   - Request/Response formats
   - Error codes

4. TRADING_STRATEGY.md:
   - Strategy explanation
   - Risk management rules
   - AI decision process
   - Indicator usage

5. DEPLOYMENT.md:
   - Production setup
   - Security best practices
   - Backup procedures
   - Scaling considerations

6. TROUBLESHOOTING.md:
   - Common issues and solutions
   - Error messages explained
   - Performance optimization
   - FAQ section

Include code examples, screenshots, and diagrams where appropriate.
```

---

## USAGE INSTRUCTIONS FOR AI

When using these prompts:

1. **Start with Prompt 1** - Set up the environment
2. **Follow the order** - Each component builds on previous ones
3. **Test each component** - Ensure it works before moving to next
4. **Customize as needed** - Adapt prompts to specific requirements
5. **Handle errors** - Add comprehensive error handling
6. **Security first** - Never hardcode sensitive information
7. **Document everything** - Add comments and docstrings
8. **Test thoroughly** - Use paper trading before real money

## IMPORTANT NOTES

- Always test with demo account first
- Keep lot size at 0.01 (never change)
- Monitor the bot closely for first 24 hours
- Have emergency stop mechanism ready
- Back up configuration regularly
- Keep API keys secure and rotate regularly
- Monitor API usage to avoid rate limits
- Update dependencies regularly for security

## FINAL INTEGRATION

After all components are built:
1. Run install.bat on Windows
2. Configure API keys in config.json
3. Start backend with start_bot.bat
4. Deploy frontend with Docker
5. Access dashboard at http://localhost:3000
6. Configure AI models and risk settings
7. Test with paper trading mode
8. Monitor for 24 hours before live trading
