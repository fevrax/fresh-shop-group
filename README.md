
# 启运冻品

[![Go](https://img.shields.io/badge/Go-1.18+-blue.svg)](https://golang.org/)
[![Vue](https://img.shields.io/badge/Vue.js-3.x-green.svg)](https://vuejs.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](/LICENSE)

**启运冻品** 是一款功能强大的现代化电商系统，基于 Go (Gin) 和 Vue 3 构建。它提供了一整套从后台管理到小程序商城的完整解决方案。

## ✨ 功能特性

*   **后台管理系统**:
    *   **仪表盘**: 核心数据可视化。
    *   **权限管理**: 基于 Casbin 的灵活角色权限控制。
    *   **商品管理**: 商品分类、规格、详情等。
    *   **订单管理**: 订单列表、详情、发货等。
    *   **用户管理**: 用户列表、信息、等级等。
    *   **系统设置**: 站点信息、支付方式、物流配置。
*   **微信小程序商城**:
    *   用户登录注册。
    *   商品浏览、搜索、收藏。
    *   购物车功能。
    *   在线下单与支付。
    *   订单查询与管理。

## 🚀 快速启动

### 环境准备

*   Go >= 1.18
*   Node.js >= 16.0
*   MySQL >= 5.7
*   Redis

### 后端 (`fresh-shop/server`)

1.  **克隆项目**
    ```bash
    git clone https://github.com/fevrax/fresh-shop-group.git
    cd fresh-shop-group
    ```

2.  **配置**
    *   进入 `fresh-shop/server` 目录，复制 `config.yaml.example` 为 `config.yaml`。
    *   修改 `config.yaml` 中的Mysql数据库、Redis 等配置。

3.  **初始化数据库**
    *   导入 `sql/fresh-shop.sql` 到你的 MySQL 数据库。

4.  **安装依赖并运行**
    ```bash
    cd fresh-shop/server
    go mod tidy
    go run main.go
    ```
    服务将启动在 `http://localhost:48888`。

### 前端后台管理 (`fresh-shop/web`)

> 本项目前端使用 `pnpm` 作为包管理器。请先安装 `pnpm`：
> ```bash
> npm install -g pnpm
> ```

1.  **安装依赖**
    ```bash
    cd fresh-shop/web
    pnpm install
    ```

2.  **运行**
    ```bash
    pnpm serve
    ```
    访问 `http://localhost:8080`。

### 微信小程序 (`fresh-shop-uniapp`)

1.  **导入项目**
    *   使用 HBuilder X 导入 `fresh-shop-uniapp` 项目。

2.  **配置**
    *   修改 `config/config.js` 中的 API 地址为你的后端服务地址。

3.  **运行到小程序模拟器**
    *   在 HBuilder X 中，点击“运行” -> “运行到小程序模拟器” -> “微信开发者工具”。


## 🤝 贡献

欢迎提交 PRs 和 Issues！

## 📄 开源许可

本项目基于 [MIT](/LICENSE) 许可。



![Star History](https://api.star-history.com/svg?repos=fevrax/fresh-shop-group&type=Date)
