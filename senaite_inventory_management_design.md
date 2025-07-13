# SENAITE LIMS 智能库存管理模块 - 设计方案

## 1. 愿景与目标

**愿景**: 为 SENAITE LIMS 打造一个无缝集成、AI驱动的智能库存管理模块，将传统的实验室物料管理（试剂、耗材等）转变为主动、预测性和高度自动化的流程，从而显著降低成本、减少浪费、确保实验不间断，并为实验室供应链提供深度洞察。

**核心目标**: 

- **自动化**: 最小化手动库存盘点和采购请求。
- **智能化**: 利用AI预测物料消耗，优化库存水平，并提供智能决策支持。
- **集成化**: 与 SENAITE LIMS 的核心分析工作流深度融合，实现物料消耗的自动记录。
- **合规性**: 完整记录物料的生命周期（从入库到消耗），满足GLP/GMP等质量体系的追溯要求。
- **可扩展性**: 遵循 SENAITE 的模块化设计，使其易于维护和功能扩展。

---

## 2. 设计理念与技术选型

- **保持一致性**: 严格遵循 SENAITE LIMS 的架构和设计模式。后端采用 Python、Plone (Dexterity Content Types)，前端拥抱现代JavaScript框架（如 React/Vue）并通过 `senaite.jsonapi` 进行交互。
- **API优先**: 所有功能都将通过 `senaite.jsonapi` 暴露，确保前端和第三方系统可以轻松集成。
- **AI驱动**: 将AI作为核心能力而非附加功能来设计，贯穿于预测、预警和决策支持的各个环节。
- **用户中心**: 设计简洁直观的用户界面，降低学习成本，提高工作效率。

---

## 3. 架构设计

### 3.1. 数据层 (Model)

我们将基于 Plone 的 Dexterity Content Types 创建以下核心对象，存储在 Zope 对象数据库 (ZODB) 中。

- **`Supplier` (供应商)**
  - `title`: 供应商名称
  - `contact_person`: 联系人
  - `email`: 邮箱
  - `phone`: 电话
  - `address`: 地址
  - `rating`: 供应商评级 (用于AI推荐)

- **`Product` (产品目录)**: 代表一种特定规格的试剂或耗材
  - `title`: 产品名称 (如, "500mL 98% 硫酸")
  - `product_id`: 产品ID/货号
  - `supplier`: 关联到 `Supplier` (ReferenceField)
  - `category`: 类别 (如, "酸", "移液管")
  - `unit`: 单位 (如, "mL", "g", "个")
  - `safety_stock_level`: 安全库存阈值

- **`StockItem` (库存批次)**: 代表一个具体的库存实体
  - `title`: 批次号 (可自动生成)
  - `product`: 关联到 `Product` (ReferenceField)
  - `lot_number`: 厂商批号
  - `expiry_date`: 过期日期
  - `quantity_initial`: 初始数量
  - `quantity_current`: 当前数量
  - `storage_location`: 存储位置 (如, "冰箱A层", "危化品柜")
  - `status`: 状态 (如, "在库", "已用尽", "已过期")
  - `received_date`: 入库日期

- **`ConsumptionRecord` (消耗记录)**
  - `stock_item`: 关联到 `StockItem` (ReferenceField)
  - `analysis_request`: 关联到触发消耗的分析请求 (可选, ReferenceField)
  - `quantity_consumed`: 消耗数量
  - `consumed_by`: 消耗人
  - `consumed_date`: 消耗日期

### 3.2. 逻辑层 (Backend)

逻辑层将作为 `senaite.core` 的一个新插件 `senaite.inventory` 来实现。

- **核心服务 (Python)**
  - **库存服务**: 实现 `StockItem` 的 CRUD 操作，处理库存的增加（入库）和减少（消耗）。
  - **工作流集成**: 监听 `AnalysisRequest` 的状态变化事件。当一个分析被执行时，根据预设的物料清单 (BOM)，自动扣减相应 `StockItem` 的 `quantity_current` 并创建 `ConsumptionRecord`。
  - **通知服务**: 
    - 当 `quantity_current` 低于 `safety_stock_level` 时，触发低库存预警。
    - 当 `expiry_date` 临近时，触发临期预警。

- **API 端点 (`senaite.jsonapi`)**
  - `GET /inventory/products`: 获取所有产品信息。
  - `GET /inventory/stock_items`: 查询库存批次（可按产品、位置、状态过滤）。
  - `POST /inventory/stock_items`: 新建入库批次。
  - `POST /inventory/stock_items/{id}/consume`: 记录一次手动消耗。
  - `GET /inventory/suppliers`: 获取供应商列表。

- **AI 引擎 (Python)**
  - **消耗预测服务**: 
    - **数据源**: `ConsumptionRecord` 的历史数据。
    - **模型**: 初期可采用简单的时间序列模型 (如 ARIMA, Exponential Smoothing) 或基于历史平均消耗的回归模型。模型将定期（如每周）重新训练。
    - **API**: `GET /inventory/products/{id}/forecast?days=30` -> 返回未来30天的预测消耗量。
  - **智能补货建议服务**:
    - **逻辑**: 结合 `消耗预测`、`当前库存 (quantity_current)` 和 `安全库存 (safety_stock_level)`，生成补货建议列表。
    - **API**: `GET /inventory/reorder_suggestions` -> 返回 {product, suggested_quantity, supplier_recommendation} 列表。

### 3.3. 展示层 (Frontend)

前端将作为 SENAITE UI 的一个新模块，使用 React/Vue.js 开发。

- **仪表盘 (Dashboard)**
  - **核心指标**: 库存总览（种类、数量）、低库存警告、临期警告。
  - **图表**: 库存消耗趋势图、各类别物料占比图。
- **库存管理视图**
  - 以表格形式展示所有 `StockItem`，支持强大的搜索、排序和过滤功能。
  - 提供快速操作按钮，如 "登记消耗"、"盘点"、"标记用尽"。
- **产品目录视图**
  - 管理 `Product` 和 `Supplier` 信息。
- **入库表单**
  - 简洁的表单用于登记新的 `StockItem`，支持扫码枪快速输入批号。
- **智能补货页面**
  - 展示AI生成的补货建议列表，用户可一键生成采购订单或通知采购部门。

---

## 4. 开发路线图与时间预估

### 第一阶段: 核心功能 (MVP - 最小可行产品)

- **内容**: 
  - 完成数据层所有 Content Types 的定义。
  - 实现核心的库存 CRUD 和手动消耗逻辑。
  - 实现与分析工作流的初步集成（自动扣减库存）。
  - 开发基础的前端界面（库存列表、入库表单）。
  - 实现低库存和临期预警通知。
- **预估时间**: **15 - 20 开发日**

### 第二阶段: AI 赋能与体验优化

- **内容**: 
  - 开发并集成消耗预测模型。
  - 开发智能补货建议功能及相应界面。
  - 完善仪表盘，增加数据可视化图表。
  - 优化前端交互，支持扫码等高效操作。
- **预估时间**: **10 - 15 开发日**

### **总预估时间**: **25 - 35 开发日**

*此预估基于一名熟悉 SENAITE/Plone 架构的资深开发者。时间包括开发、单元测试和集成测试。*