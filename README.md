# Trading_Bot_System

## Các PORT được quy định trong hệ thống
- DB-Server: 3001
- ML-Server: 3002
- Gekko: 3003
- Manager: 3004
- UI: 3005
- Gekko Igniter: 3006 (dockerhost)
## Các bước để chạy toàn bộ hệ thống
- Thiết lập file config và chạy docker-compose, xem phần `Các bước để chạy docker-compose up`.
- Build Gekko với tên image, ví dụ ở đây dùng tên `gekko_image`.
    - `docker build -t gekko_image .`
- Chạy GekkoIgniter
    - Yêu cầu môi trường đã cài đặt có `node` và `docker`.
    - Tạo file `/GekkoIgniter/config.js`, nếu như đặt tên khác cho Gekko Image thì cần phải sửa lại trong `gekkoImageName`.
    Ví dụ:
    ```
    var config = {};

    config.debug = true;
    config.silent = false;

    config.production = true;
    config.loggerAdapter = 'file';

    // Tên của gekko image đã được build
    config.gekkoImageName = "gekko_image";

    // Tên của network hệ thống sử dụng trong Docker
    config.networkName = "trading_bot_system_local"

    // Tên file config sẽ được đặt bên trong Gekko
    config.nameConfigInGekko = "config_by_gekko_igniter";

    module.exports = config;

    ```
    - Set các biến môi trường sau trước khi chạy:
        - PORT=3006
        - LIVE_TRADE_MANAGER_BASE_API (trong container): nếu không thay đổi gì thì là `http://live_trading_manager:3004`
        - ML_BASE_API (trong container): nếu không thay đổi gì thì là `http://ml_server:3002`
        - AUTHENTICATION_TOKEN: Token dùng chung cho tất cả các thành phần trong hệ thống.
        - MONGO_INITDB_ROOT_USERNAME: Username của root user sẽ được tạo với mongodb
        - MONGO_INITDB_ROOT_PASSWORD: Password của root user sẽ được tạo với mongodb
    - Chạy lệnh `npm start`
## Các bước để chạy docker-compose up
- Chuẩn bị file `/DB-Server/config.js` cho `DB-Server`
Ví dụ:
    ```
    var config = {};

    // Mốc thời gian bắt đầu sync dữ liệu lịch sử
    config.beginAt = "2018-02-01 00:00:00";

    // Các cặp cần sync dữ liệu
    config.pairs = [
        {
            exchange: "binance",
            currency: "USDT",
            asset: "BTC",
        }
    ]

    // Hệ quản trị cơ sở dữ liệu, hệ thống hiện tại chỉ hỗ trợ mongodb
    config.adapterDatabase = "mongo";
    config.mongo = {
        connectionString: process.env.MONGO_URL,
        dbName:  "db_candles_of_cryptocurrency"
    }

    config.debug = true;
    config.silent = false;

    config.production = true;
    config.loggerAdapter = 'file';

    module.exports = config;
    ```
- Chuẩn bị file `/live-trading-manager/config.js` cho `live-trading-manager`
Ví dụ:
    ```
    let config = {};

    config.mongodb = {
        connectionString: process.env.MONGO_URL,
        dbName:  "db_live_trading_manager"
    }

    config.machine_learning_api = {
        base: process.env.ML_SERVER_BASE_API,
        live: "/live"
    }

    config.gekko_igniter_api = {
        base: `http://${process.env.DOCKER_HOST}:3006`,
        runGekko: '/run-gekko',
        stopGekko: '/stop-gekko',
        startGekko: '/start-gekko',
        backtest: '/backtest'
    }

    config.loggerAdapter = 'file';

    module.exports = config;
    ```
- Chuẩn bị file `/live-trading-manager/authentication/info.js` cho `live-trading-manager`
Ví dụ:
    ```
    module.exports = {
        users: [
            {
                username: "guest",
                password: "67583412239"
            }
        ],
        secret_key: "this_is_a_secret_key",
        // Thời gian mà token sẽ hết hạn sau khi được sinh ra
        live_time_of_token: 1000 * 60 * 60 * 24 //1 ngày
    }
    ```
- Sửa file `/ML-For-Trading-Bot/config.py` nếu như có thay đổi cấu hình với `ML-For-Trading-Bot`.
- Set các biến môi trường:
    - AUTHENTICATION_TOKEN: Token dùng chung cho tất cả các thành phần trong hệ thống.
    - MONGO_INITDB_ROOT_USERNAME: Username của root user sẽ được tạo với mongodb
    - MONGO_INITDB_ROOT_PASSWORD: Password của root user sẽ được tạo với mongodb
- Chạy lệnh `docker-compose up -d`

## Lưu ý:
- Lúc dừng toàn bộ hệ thống thì down từng node, không dùng `docker-compose down` dẫn đến network nội bộ bị xóa làm cho các gekko container không thể kết nối vào hệ thống mới sau khi start lại.
- User của mongodb chỉ được tạo 1 lần duy nhất lúc tạo volume, những lần sau đều sử dụng lại user ban đầu (lưu ý để truyền vào cho đúng cho live-trading-manager và db-server ở những lần chạy lại)