
# fresh-shop

<div align="center">
  <img src="https://img.shields.io/badge/golang-1.16+-blue"/>
  <img src="https://img.shields.io/badge/gin-1.7+-lightBlue"/>
  <img src="https://img.shields.io/badge/vue-3.2+-brightgreen"/>
  <img src="https://img.shields.io/badge/vite-3.0+-green"/>
  <img src="https://img.shields.io/badge/pinia-2.0+-yellow"/>
  <img src="https://img.shields.io/badge/element--plus-2.2+-blueviolet"/>
</div>

[English](./README-en.md) | 简体中文

## 简介

fresh-shop 是一个基于 Vue 3 和 Gin 开发的全栈前后端分离的生鲜电商项目。它实现了一整套电商系统所需的核心功能，旨在为开发者提供一个快速搭建、易于扩展的电商解决方案。

## 主要功能

*   **商品管理**: 支持商品的增删改查、分类、品牌、标签等管理功能。
*   **订单中心**: 包括订单创建、支付、发货、退货等完整的订单生命周期管理。
*   **购物车**: 提供稳定、高效的购物车功能。
*   **用户系统**: 包含用户注册、登录、地址管理、收藏夹等。
*   **系统管理**: 动态路由、菜单管理、权限控制等后台基础功能。
*   **微信集成**: 支持微信登录、支付等功能。

## 技术选型

- **前端**:
  - [Vue 3](https://vuejs.org/)
  - [Vite](https://vitejs.dev/)
  - [Pinia](https://pinia.vuejs.org/)
  - [Vue Router](https://router.vuejs.org/)
  - [Element Plus](https://element-plus.org/)
  - [Axios](https://axios-http.com/)

- **后端**:
  - [Go](https://golang.org/)
  - [Gin](https://gin-gonic.com/)
  - [GORM](https://gorm.io/)
  - [JWT](https://jwt.io/) for authentication
  - [Swagger](https://swagger.io/) for API documentation

## 快速开始

### 环境准备

- [Node.js](https://nodejs.org/) >= 16.0
- [Go](https://golang.org/) >= 1.16
- [MySQL](https://www.mysql.com/) >= 5.7

### 后端启动

```bash
# 克隆项目
git clone <your-repo-url>

# 进入后端目录
cd server

# 安装依赖
go mod tidy

# 编译
go build -o server main.go

# 运行
./server
```

### 前端启动

```bash
# 进入前端目录
cd web

# 安装依赖
npm install

# 启动
npm run serve
```

## 项目结构

```
.
├── server/         # 后端代码
│   ├── api/
│   ├── model/
│   ├── router/
│   ├── service/
│   └── main.go
├── web/            # 前端代码
│   ├── public/
│   ├── src/
│   │   ├── api/
│   │   ├── components/
│   │   ├── router/
│   │   ├── store/
│   │   └── views/
│   └── package.json
├── sql/            # 数据库脚本
└── deploy/         # 部署相关
```

## 贡献

欢迎提交 issue 和 pull request！

在提交代码前，请确保：
1.  代码风格与项目保持一致。
2.  提交信息清晰、规范。
3.  相关 issue 已被链接。

## 许可证

[Apache-2.0](./LICENSE)
