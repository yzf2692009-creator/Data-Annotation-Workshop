# DataLab Annotation Studio

> 中文名：数据标注工坊  
> 一个支持私有化部署的数据采集、数据标注、任务管理、审核质检与报表统计平台。

## 在线预览

访问地址：[https://annotation.fcpublic.com](https://annotation.fcpublic.com)

### 管理员权限获取

如需体验管理员权限，请添加下方微信二维码联系获取。

<img src="./docs/assets/admin-wechat-qr.jpg" alt="管理员权限获取微信二维码" width="260" />

## 基础版价格

| 版本 | 价格 | 交付方式 | 是否支持二开 | 适用场景 |
| --- | ---: | --- | --- | --- |
| 源码一次性购买版 | 21000 元 | 源码交付 | 支持 | 私有化部署、深度定制、长期扩展 |
| 镜像部署版 | 12000 元 | Docker 镜像交付 | 不支持 | 快速部署、标准化使用、低成本上线 |

数据标注工坊基础版提供两种交付方式，用户可根据自身部署、扩展和二次开发需求灵活选择。

源码一次性购买版适合希望长期自主管理、深度定制和持续扩展的平台用户。购买后可获得系统源码，支持本地私有化部署，并可根据业务流程、标注场景、权限体系、报表统计、模型接入等需求进行二次开发。

镜像部署版适合希望快速上线、低成本私有化部署的用户。平台以 Docker 镜像方式交付，部署更简单，环境依赖更清晰，适合标准化使用场景。该版本可正常使用平台已有功能，但不提供源码，因此不支持二次开发和深度定制。

## 项目简介

DataLab Annotation Studio 是一个面向 AI 数据生产流程的数据标注与数据采集平台，适用于企业内部数据标注团队、AI 实验室、算法团队、数据服务团队以及需要私有化部署的数据生产场景。

平台围绕“数据采集 - 数据集管理 - 标注任务 - 标注执行 - 审核质检 - 报表统计”的完整流程进行设计，帮助团队统一管理数据、人员、任务、进度和质量。

相比单一的数据标注工具，本项目不仅提供标注工作台，还包含任务中心、成员权限、数据采集、脚本采集、审核评分、操作记录、通知提醒和报表中心等能力，更适合构建企业内部的数据生产平台。

## 核心功能

### 数据集管理

- 支持数据集创建、管理和维护
- 支持图片、视频、音频、文本、文档、点云等多种数据类型
- 支持数据预览、文件管理和数据集关联
- 支持将采集结果导入统一数据集

### 任务中心

- 支持创建标注任务和数据采集任务
- 支持设置任务负责人、执行人员和审核人员
- 支持任务领取、任务执行、任务提交和审核流程
- 支持任务优先级、截止时间、任务说明和审核文档
- 支持任务进度、操作记录和任务详情查看

### 数据采集

- 支持脚本采集任务
- 支持平台脚本和自定义脚本方式
- 支持 Docker 沙箱运行采集脚本
- 支持运行参数填写、日志查看和采集结果预览
- 支持选择采集文件并提交任务结果
- 支持采集数据下载和后续导入数据集

### 标注工作台

平台内置多种标注页面，覆盖常见 AI 数据标注场景：

- 图像目标检测标注
- 图像多边形标注
- 图像语义分割标注
- 图像关键点标注
- 图像 / 视频描述生成标注
- 文本分类标注
- 文本 NER 标注
- 音频分段标注
- 视频标注
- 3D 点云标注
- 车道线标注
- 天气、路况等全图标签标注
- 通用企业数据标注页面

### 审核质检

- 支持标注结果审核
- 支持通过、驳回和反馈意见
- 支持标注结果预览
- 支持审核记录追踪
- 支持标注质量评分和质量统计

### 报表中心

- 支持标注任务报告
- 支持标注质量报告
- 支持标签频次统计
- 支持样本明细查看
- 支持标注员质量统计
- 支持任务完成率、通过率、返工率等数据展示

### 成员与权限

- 支持成员管理
- 支持角色管理和用户组管理
- 支持管理员、标注员、采集员、审核员等角色
- 支持批量导入、批量删除成员
- 支持权限菜单控制
- 支持单点登录控制，同一账号在多设备登录时可强制下线

### 操作审计与通知

- 支持任务创建、领取、执行、提交、审核、导出等操作记录
- 支持敏感操作追踪
- 支持任务分配、驳回、审核通过、脚本失败等通知提醒

## 技术栈

### 前端

- React
- Vite
- JavaScript
- 自定义企业级 UI 样式

### 后端

- Python
- FastAPI
- SQLAlchemy
- MySQL
- Redis

### 服务模块

- 前端 Web 服务
- 后端 API 服务
- 文件服务
- 模型服务
- Docker 脚本采集运行环境

## 项目结构

```text
data-annotation-project/
├── data-annotation-react/      # 前端项目
├── data-annotation-api/        # 后端 API 服务
├── data-annotation-file/       # 文件服务
├── data-annotation-model/      # 模型服务与模型接入示例
├── docs/                       # 项目文档
├── scripts/                    # 开发与启动脚本
├── docker-compose.yml          # Docker 部署配置
└── .env.docker.example         # Docker 环境变量示例
```

## Docker 部署

### 1. 克隆项目

```bash
git clone <your-repository-url>
cd data-annotation-project
```

### 2. 创建环境变量文件

```bash
cp .env.docker.example .env.docker
```

根据实际部署环境修改 `.env.docker`。

常用配置示例：

```env
WEB_PORT=8080
API_PORT=8000
FILE_PORT=8100
MODEL_PORT=8200

MYSQL_PORT=3306
REDIS_PORT=6379

HOST_DATA_ROOT=/data/annotation

INIT_SUPER_ADMIN_ACCOUNT=admin
INIT_SUPER_ADMIN_PASSWORD=Admin123456
INIT_SUPER_ADMIN_NAME=管理员
INIT_SUPER_ADMIN_EMAIL=admin@example.com
```

### 3. 启动服务

```bash
docker compose --env-file .env.docker up -d --build
```

### 4. 访问系统

本地访问：

```text
http://localhost:8080
```

服务器部署后访问：

```text
http://服务器IP:8080
```

### 5. 查看服务状态

```bash
docker compose ps
```

### 6. 查看日志

```bash
docker compose logs -f api
docker compose logs -f web
```

### 7. 停止服务

```bash
docker compose down
```

## 本地开发

### 后端 API

```bash
cd data-annotation-api
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python run.py
```

### 前端项目

```bash
cd data-annotation-react
npm install
npm run dev
```

### 本地依赖

本地开发需要准备：

- MySQL
- Redis
- Python 运行环境
- Node.js / npm

示例配置：

```env
DATABASE_URL=mysql+pymysql://annotation:annotation_pass@127.0.0.1:3306/data_annotation?charset=utf8mb4
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

## 邮件验证码配置

平台支持阿里云 DirectMail 邮件推送，用于注册验证码和找回密码验证码。

示例配置：

```env
ALIYUN_DM_ACCESS_KEY_ID=
ALIYUN_DM_ACCESS_KEY_SECRET=
ALIYUN_DM_REGION_ID=cn-hangzhou
ALIYUN_DM_ENDPOINT=dm.aliyuncs.com
ALIYUN_DM_FROM_EMAIL=
ALIYUN_DM_FROM_ALIAS=数据标注工坊
ALIYUN_DM_REGISTER_TEMPLATE_ID=
ALIYUN_DM_RESET_TEMPLATE_ID=
ALIYUN_DM_TEMPLATE_PLATFORM_KEY=platform_name
ALIYUN_DM_TEMPLATE_CODE_KEY=code
ALIYUN_DM_TEMPLATE_EXPIRE_KEY=expire_minutes
```

## 脚本采集运行环境

平台支持通过 Docker 沙箱执行数据采集脚本。

可配置内容包括：

- Python 基础镜像
- Python 依赖包
- CPU 和内存限制
- 网络模式
- 输出目录
- 环境变量
- 额外挂载目录
- 设备访问权限

采集脚本执行后，可以在平台中查看运行日志、下载采集结果，并选择需要提交的采集文件。

## 模型服务接入

平台支持 HTTP 方式接入外部模型服务，可用于模型预处理、辅助标注等场景。

模型接入示例位于：

```text
data-annotation-model/examples/
```

## 适用场景

- 企业内部数据标注平台
- AI 训练数据生产
- 计算机视觉数据标注
- NLP 文本标注
- 音视频数据标注
- 3D 点云数据标注
- 数据采集与数据清洗流程
- 标注团队任务管理
- 私有化部署的数据生产系统

## 推荐仓库描述

```text
支持私有化部署的数据采集与数据标注平台，覆盖任务管理、审核质检、报表统计和团队协作流程。
```

## License

本项目适用于私有化部署、企业数据标注流程和 AI 数据生产场景。
