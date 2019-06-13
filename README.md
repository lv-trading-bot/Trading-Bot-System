# Trading_Bot_System

# PORT
- DB-Server: 3001
- ML-Server: 3002
- Gekko: 3003
- Manager: 3004
- UI: 3005
- Gekko Igniter: 3006 (dockerhost)
# Các bước để chạy toàn bộ hệ thống
- Build Gekko với tên image, ví dụ ở đây dùng tên `gekko_image`
    - `docker build -t gekko_image .`
- Thiết lập file config và chạy docker-compose, ở dưới đã hướng dẫn
- Chạy GekkoIgniter
    - Yêu cầu có `node` và `docker`
    - Tạo file `config.js`, 
    Example:
    ```
    var config = {};

    config.debug = true;
    config.silent = false;

    config.production = true;
    config.loggerAdapter = 'file';

    config.gekkoImageName = "test_gekko_config";
    config.networkName = "trading_bot_system_local"

    config.nameConfigInGekko = "config_by_gekko_igniter";

    module.exports = config;

    ```
    - set biến môi trường `PORT=3006`, `LIVE_TRADE_MANAGER_BASE_API=http://live_trading_manager:3004`
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
- Set biến môi trường `AUTHENTICATION_TOKEN`
- Set biến môi trường `MONGO_INITDB_ROOT_USERNAME` và `MONGO_INITDB_ROOT_PASSWORD`, đây là root user của mongodb
- `docker-compose up -d`

## Lưu ý:
- Lúc dừng toàn bộ hệ thống thì down từng node, không dùng `docker-compose down` dẫn đến network nội bộ bị xóa làm cho các gekko container không thể kết nối vào hệ thống mới sau khi start lại.
- User của mongodb chỉ được tạo 1 lần duy nhất lúc tạo volume, những lần sau đều sử dụng lại user ban đầu (lưu ý để truyền vào cho đúng cho live-trading-manager và db-server ở những lần chạy lại)