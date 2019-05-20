# Trading_Bot_System

# PORT
- DB-Server: 3001
- ML-Server: 3002
- Gekko: 3003
- Manager: 3004
# Các bước để thêm 1 gekko mới
- clone source gekko sang thư mục khác
- xóa hết file trong thư mục `logs` và `save_info`
- sửa file `tin-config-paper-trading.js`
- chạy lệnh `docker build . -t <some_thing>`
- chạy lệnh `docker run -e LIVE_TRADE_MANAGER_BASE_API=http://trading_bot_system_live_trading_manager_1:3004 --name <some_thing> -v <Đường dẫn đến thư mục logs>:/usr/src/app/logs --network trading_bot_system_local <some_thing>`