# Kong + Konga 部署指南

本目录提供以 docker-compose 方式一键部署 [Kong API 网关](https://konghq.com/)
及其可视化管理面板 Konga 的生产级组合（含 Postgres 持久化存储）。

---

## 一、环境准备

- 推荐系统：Linux (也支持 macOS)
- 需安装：
  - [Docker](https://docs.docker.com/get-docker/)
  - [docker-compose](https://docs.docker.com/compose/)
- 推荐具备网络 8000/8001/8443/1337 端口未被占用

---

## 二、服务说明

- kong-database (Postgres): 持久化存储 Kong 与 Konga 数据
- kong-migration: 负责数据迁移和初始化 (首次部署自动运行)
- kong: Kong API 网关核心服务
- konga-prepare: Konga 管理库准备，仅首次自动执行
- konga: Kong 可视化管理面板（Web）

---

## 三、主要端口

| 宿主机端口 | 容器服务 | 用途            |
| ---------- | -------- | --------------- |
| 80         | kong     | Gateway HTTP    |
| 443        | kong     | Gateway HTTPS   |
| 8001       | kong     | Kong Admin API  |
| 1337       | konga    | Konga Web界面   |
| 55432      | postgres | PG 数据库管理口 |

---

## 四、启动与关闭

### 启动

```bash
docker compose up -d
```

- 首次会自动完成数据库表/迁移、admin 初始化。
- 部署完毕可访问：
  - Kong 管理界面（Konga）：http://localhost:1337
  - Kong 网关 HTTP: http://localhost/
  - Kong Admin API: http://localhost:8001
  - Kong 网关 HTTPS: https://localhost/

### 关闭

```bash
docker compose down
```

---

## 五、账号与初始设置

- Konga 首次访问需注册管理员账号
- Postgres 相关数据库用户名/密码详见 yaml 配置（kong/konga/kong）
- 配置项均可自定义，如需更改建议同步修改 yaml 和持久化的 volume 路径（防止数据丢失）

---

## 六、常见问题与配置说明

1. **端口占用**
   - 如宿主机上述端口被占用，可在 `ports` 下自行调整
2. **HTTPS 网关配置**
   - 切勿使用 `KONG_PROXY_LISTEN_SSL`，正确写法为：
     ```yaml
     KONG_PROXY_LISTEN: 0.0.0.0:8000, 0.0.0.0:8443 ssl
     ```
   - `443:8443` 映射关系，外部443即为 API Gateway 的 https 入口。
3. **Konga 无法连接数据库？**
   - 请确保数据库初始化成功后再启动 Konga。
   - konga-prepare 只需准备一次。
4. **konga-prepare 执行后报错？**
   - 可忽略，主要是管理库初始化，多次启动不会影响主业务。

---

## 七、数据持久化说明

- Postgres 持久化目录：参考 `volumes` 路径 `/root/docker/kong/data`（可按需修改为本机目录）

---

## 八、目录结构

```shell
docker/kong/
├── docker-compose.yml  # 部署主文件
├── README.md           # 本说明
└── ...（可能包含额外脚本/配置）
```

---

## 九、参考

- Kong 官方文档：https://docs.konghq.com/
- Konga 项目地址：https://github.com/pantsel/konga
- 如需高级参数调优详见 yaml 文件内注释
