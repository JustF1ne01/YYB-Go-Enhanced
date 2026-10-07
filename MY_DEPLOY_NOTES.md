# 个人部署笔记（自用）

## 环境
- 设备：M5Plus（ARM 小机子）
- YYB 映射端口：8081（浏览器访问 http://<机器IP>:8081）
- 青龙容器名：qinglong，容器内部端口 5700，宿主机映射 15700

## 部署必做（否则起不来）
1. 建外部网络并接入青龙：
   docker network create qinglong
   docker network connect qinglong qinglong
   （compose 里 networks.qinglong 是 external:true，不会自动建，必须手动建+连）
2. 数据目录提前建好并放开写权限（否则 SQLite 报 "out of memory (14)"，
   实际是 SQLITE_CANTOPEN 权限问题，不是内存）：
   mkdir -p data/db data/avatars data/qr
   chmod -R 777 data

## QL_URL 填法（坑）
- 正确：QL_URL=http://qinglong:5700
  （同一 docker 网络内用「容器名+容器内部端口」，不是宿主机 IP/映射端口）
- 错误：http://10.0.0.22:15700 或 http://qinglong:15700
  （15700 是宿主机映射端口，容器间互访要用 5700）

## 青龙应用凭证
- client_id / client_secret 在青龙面板 → 系统设置 → 应用设置 → 添加应用 生成

## 已知报错对照
- connection refused :15938 → QL_URL 端口错，改成 5700
- 青龙 HTTP 400 [0].value 不允许为空 → 执行「当前账号独立推送」时，
  该账号在 YYB 里没登录/凭证为空，先登录再推
- init app: unable to open database file: out of memory (14) → data/db 目录无写权限，chmod 777

## 验证命令
docker network inspect qinglong        # 看 Containers 含 qinglong 和 yyb-go
docker compose ps                     # yyb-go 状态
docker compose logs yyb-go --tail 40  # 排错
docker inspect qinglong --format '{{json .NetworkSettings.Ports}}'  # 查容器端口
