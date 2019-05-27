# Trading_Bot_System

# PORT
- DB-Server: 3001
- ML-Server: 3002
- Gekko: 3003
- Manager: 3004\
# Các bước để chạy docker-compose up
- Chuẩn bị file `config.js` cho `DB-Server`
Example:
    ```
    var config = {};

    config.beginAt = "2018-02-01 00:00:00";

    config.pairs = [
        {
            exchange: "binance",
            currency: "USDT",
            asset: "BTC",
        }
    ]

    config.adapterDatabase = "mongo";
    config.mongo = {
        connectionString: process.env.MONGO_URL,
        dbName:  "db_candles_of_cryptocurrency_docker"
    }

    config.debug = true;
    config.silent = false;

    config.production = true;
    config.loggerAdapter = 'file';

    module.exports = config;
    ```
- Sửa file `tin-config-paper-trading.js` trong thư mục Gekko để cấu hình cặp chuẩn bị chạy
- Chuẩn bị file `config.js` cho `live-trading-manager`
Example:
    ```
    let config = {};

    config.mongodb = {
        connectionString: process.env.MONGO_URL || "mongodb://localhost:27017",
        dbName:  "db_live_trading_manager"
    }

    config.pairs = {
        listPairs: [
            {
                asset: "BTC",
                currency: "USDT",
                candleSize: 60
            }
        ]
    }

    config.machine_learning_api = {
        base: process.env.ML_SERVER_BASE_API || "http://localhost:3002",
        live: "/live"
    }

    config.production = true;
    config.loggerAdapter = 'file';

    module.exports = config;
    ```
- Sửa file `config.py` trong thư mục `ML-For-Trading-Bot` nếu như có cấu hình khác.
- `docker-compose up -d`
# Các bước để thêm 1 gekko mới
- clone source gekko sang thư mục khác
- xóa hết file trong thư mục `logs` và `save_info`
- sửa file `tin-config-paper-trading.js`
- chạy lệnh `docker build . -t <some_thing>`
- chạy lệnh `docker run -d -e LIVE_TRADE_MANAGER_BASE_API=http://trading-bot-system_live_trading_manager_1:3004 --name <some_thing> -v <Đường dẫn đến thư mục logs>:/usr/src/app/logs --network trading-bot-system_local <some_thing>`
