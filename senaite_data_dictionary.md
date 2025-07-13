# Senaite LIMS 数据字典

本文档旨在阐明 Senaite LIMS 的核心数据结构、对象及其关系，为定制开发和系统集成提供参考。

## 核心业务对象

Senaite LIMS 的业务逻辑围绕以下几个核心对象展开：

- **Analysis Request (分析请求)**: 系统的中心对象，代表一个样品及其所有相关的分析任务。它不仅是“样品”的电子记录，还包含了客户信息、采样详情、分析项目、结果、状态等所有信息。
- **Client (客户)**: 提交样品的组织或个人。
- **Analysis Profile (分析配置文件)**: 一组预定义的分析服务，可以快速应用于样品。
- **Analysis Service (分析服务)**: 单个的分析测试项目，是 LIMS 中最核心的业务对象之一。它不仅定义了要执行的测试，还包含了与测试相关的计算、方法、仪器、结果规格、修约规则等一系列复杂的配置。
- **Worksheet (工作单)**: 一组需要共同处理的分析请求，通常用于实验室内部的任务分配和管理。

---

## 1. Analysis Request (AR) - 分析请求

`AnalysisRequest` 是 Senaite 中最核心的对象，它代表了一个完整的样品分析流程。在代码层面，它由 `bika.lims.content.analysisrequest.AnalysisRequest` 类实现，并遵循 `bika.lims.interfaces.IAnalysisRequest` 接口。

**关键关系:**

- **属于 (Belongs to)** 一个 `Client` (客户)。
- **可以包含 (Can contain)** 多个 `Analysis` (分析) 对象，每个对象代表一个分析服务的结果。
- **可以关联 (Can be associated with)** 一个 `Batch` (批次)。
- **可以从 (Can be created from)** 一个 `Sample Template` (样品模板) 或 `Analysis Profile` (分析配置文件)。

### 1.1. 主要字段 (Fields)

| 字段名 (ID) | 字段类型 | 描述 | 关联对象 |
|---|---|---|---|
| `Client` | UIDReference | 样品所属的客户 | `Client` |
| `Contact` | UIDReference | 主要联系人 | `Contact` |
| `CCContact` | UIDReference (Multi) | 抄送联系人 | `Contact` |
| `CCEmails` | Emails | 其他需要通过邮件通知的地址 | - |
| `PrimaryAnalysisRequest` | UIDReference | 如果是次级样品，指向主样品 | `AnalysisRequest` |
| `Batch` | UIDReference | 样品所属的批次 | `Batch` |
| `Template` | UIDReference | 创建样品时使用的模板 | `SampleTemplate` |
| `Profiles` | UIDReference (Multi) | 应用于样品的分析配置文件 | `AnalysisProfile` |
| `DateSampled` | DateTime | 样品的采样日期和时间 | - |
| `Sampler` | String | 采样人员 | - |
| `DateReceived` | DateTime | 实验室接收样品的日期和时间 | - |
| `ClientSampleID` | String | 客户提供的样品ID | - |
| `SampleType` | UIDReference | 样品的类型 (如水、土壤等) | `SampleType` |
| `SamplePoint` | UIDReference | 采样点 | `SamplePoint` |
| `StorageLocation` | UIDReference | 样品的存储位置 | `StorageLocation` |
| `Analyses` | - (Computed) | 样品包含的所有分析服务及其结果 | `Analysis` |
| `Remarks` | Text | 备注信息 | - |

---

## 2. Client (客户) / Organization (组织)

代表一个客户公司或组织。在代码中由 `senaite.core.content.organization.Organization` 实现，遵循 `senaite.core.interfaces.IOrganization` 接口。

### 2.1. 主要字段

| 字段名 (ID) | 字段类型 | 描述 |
|---|---|---|
| `title` | String | 客户/组织名称 |
| `tax_number` | String | 税号 |
| `phone` | String | 电话 |
| `fax` | String | 传真 |
| `email` | String | 电子邮箱 |
| `address` | Text | 地址 |

---

## 3. Analysis Profile (分析配置文件)

用于将一组常用的分析服务打包，方便快速创建分析请求。由 `senaite.core.content.analysisprofile.AnalysisProfile` 实现。

### 3.1. 主要字段

| 字段名 (ID) | 字段类型 | 描述 | 关联对象 |
|---|---|---|---|
| `title` | String | 配置文件名称 |
| `description` | Text | 描述 |
| `profile_key` | String | 配置文件的唯一关键字 |
| `services` | UIDReference (Multi) | 该配置文件包含的分析服务列表 | `AnalysisService` |
| `sample_types`| UIDReference (Multi) | 适用于哪些样品类型 | `SampleType` |

---


## 4. Analysis Service (分析服务)

`AnalysisService` 是定义单个测试或测量项的核心对象。它由 `bika.lims.content.analysisservice.AnalysisService` 类实现。

### 4.1. 主要字段

| 字段名 (ID) | 字段类型 | 描述 | 关联对象 |
|---|---|---|---|
| `Title` | String | 分析服务的名称。 | - |
| `Description` | Text | 分析服务的详细描述。 | - |
| `Keyword` | String | 用于快速识别和搜索的关键字。 | - |
| `Price` | Currency | 单个分析的价格。 | - |
| `VAT` | String | 增值税率。 | - |
| `Methods` | UIDReference (Multi) | 可用于此分析服务的所有方法列表。 | `Method` |
| `Method` | UIDReference | 默认选择的方法。必须是 `Methods` 列表中的一员。 | `Method` |
| `Instruments` | UIDReference (Multi) | 可用于此分析服务的所有仪器列表。 | `Instrument` |
| `Instrument` | UIDReference | 默认选择的仪器。必须是 `Instruments` 列表中的一员。 | `Instrument` |
| `Calculation` | UIDReference | 关联的计算公式，用于从一个或多个其他分析结果计算得出此分析的结果。 | `Calculation` |
| `ResultOptions` | Lines | 结果的可选值列表。当结果是定性而非定量时使用。 | - |
| `ExponentialFormatPrecision` | Integer | 科学计数法格式的精度。 | - |
| `NumberOfDigits` | Integer | 结果允许的小数位数（修约规则）。 | - |
| `InterimFields` | Records | 中间计算字段/变量，用于复杂计算的中间步骤。 | - |
| `DefaultResult` | String | 输入结果时的默认值。 | - |
| `Conditions` | Records | 分析条件，允许在样品注册时为特定分析输入额外参数（如温度、压力等）。 | - |
| `Accredited` | Boolean | 是否通过认证。 | - |

---


## 5. Calculation (计算)

`Calculation` 对象用于定义分析结果的自动计算规则。它允许用户通过一个公式，基于一个或多个其他分析服务的结果或手动输入的中间变量，来计算得出最终结果。该对象由 `bika.lims.content.calculation.Calculation` 类实现。

### 5.1. 主要字段

| 字段名 (ID) | 字段类型 | 描述 | 关联对象 |
|---|---|---|---|
| `Title` | String | 计算的名称。 | - |
| `Formula` | Text | 核心的计算公式。使用标准的数学运算符，并用方括号 `[]` 引用其他分析服务的关键字或中间变量。例如：`[Ca] + [Mg]`。 | - |
| `InterimFields` | Records | 定义计算过程中需要的中间变量，如稀释倍数、样品重量等。这些字段可以在工作单中手动输入。 | - |
| `DependentServices` | UIDReference (Multi) | (隐藏字段) 该计算所依赖的其他分析服务列表。系统会自动根据公式中的关键字进行填充。 | `AnalysisService` |
| `PythonImports` | Records | 允许从外部 Python 库导入特定函数（如 `math.floor`），以在公式中使用。 | - |
| `TestParameters` | Records | 用于测试公式的一组参数。用户可以为公式中引用的每个变量输入一个测试值。 | - |
| `TestResult` | String | (只读) 基于 `TestParameters` 中的值计算出的测试结果。 | - |

---

## 6. Instrument (仪器)

仪器是实验室中用于执行分析测量的物理设备。在 Senaite LIMS 中，`Instrument` 对象不仅记录了设备的基本信息，还管理着其校准、认证、维护和数据接口等关键信息。

- **核心对象**: `bika.lims.content.instrument.Instrument`
- **接口**: `bika.lims.interfaces.IInstrument`

### 4.1. 主要字段

| 字段名 | 类型 | 描述 |
| --- | --- | --- |
| `InstrumentType` | UIDReference | 仪器的类型，关联到 `InstrumentType` 对象。 |
| `Manufacturer` | UIDReference | 仪器的制造商，关联到 `Manufacturer` 对象。 |
| `Supplier` | UIDReference | 仪器的供应商，关联到 `Supplier` 对象。 |
| `Model` | String | 仪器的型号。 |
| `SerialNo` | String | 仪器的唯一序列号。 |
| `Methods` | UIDReference (多值) | 该仪器支持的分析方法列表，关联到 `Method` 对象。 |
| `ImportDataInterface` | String (多值) | 用于从仪器导入结果的数据接口。 |
| `ResultFilesFolder` | Records | 配置用于自动导入结果的文件路径。 |
| `InstallationDate` | DateTime | 仪器的安装日期。 |
| `AssetNumber` | String | 仪器在资产登记册中的编号。 |
| `InstrumentLocation` | UIDReference | 仪器所在的物理位置，关联到 `InstrumentLocation` 对象。 |

### 4.2. 关联对象

- **`InstrumentCalibration`**: 记录仪器的校准活动。
- **`InstrumentCertification`**: 记录仪器的认证信息。
- **`InstrumentMaintenanceTask`**: 记录仪器的维护任务。
- **`Method`**: 仪器关联的分析方法，定义了分析的具体规程和参数。

## 7. AnalysisSpecification (分析规格)

`AnalysisSpecification` 用于定义单个分析服务在特定样品或产品中的可接受结果范围。它不是一个独立的内容类型，而是通过一个名为 `ResultsRangesField` 的字段嵌入到其他内容类型中，如 `AnalysisRequest` 和 `Product`。

**核心实现:**

- **字段类型**: `bika.lims.browser.fields.ResultsRangesField`
- **控件**: `bika.lims.browser.widgets.analysisspecificationwidget.AnalysisSpecificationWidget`
- **数据结构**: `bika.lims.content.analysisspec.ResultsRangeDict`

### 7.1. 主要字段

| 字段名 (ID) | 字段类型 | 描述 |
|---|---|---|
| `min` | Float | 结果的最小值。低于此值的结果将被标记。 |
| `max` | Float | 结果的最大值。高于此值的结果将被标记。 |
| `warn_min` | Float | 结果的警告下限。低于此值但高于 `min` 的结果会触发警告。 |
| `warn_max` | Float | 结果的警告上限。高于此值但低于 `max` 的结果会触发警告。 |
| `min_operator` | Selection | 最小值的比较操作符（如 > 或 ≥）。 |
| `max_operator` | Selection | 最大值的比较操作符（如 < 或 ≤）。 |
| `hidemin` | Boolean | 是否在报告中隐藏最小值。 |
| `hidemax` | Boolean | 是否在报告中隐藏最大值。 |
| `rangecomment` | String | 关于此规格范围的备注信息。 |
| `Methods` | UIDReference (Multi) | 适用于此规格的方法。 |
| `Unit` | String | 结果的单位。 |

### 7.2. 关联对象

- **`AnalysisRequest`**: 在分析请求级别定义特定样品的规格。
- **`Product`**: 在产品级别为所有使用该产品的样品定义统一的规格。
- **`AnalysisService`**: `AnalysisSpecification` 总是与一个特定的 `AnalysisService` 相关联，定义该服务的结果范围。

## Storage (存储)

`senaite.storage` 模块为 Senaite LIMS 提供了样品存储管理功能。它允许用户定义多层级的存储结构，从物理设施（如房间、冰箱）到具体的存储容器（如盒子、架子），再到容器内的精确位置，从而实现对样品的全生命周期追踪。

### 核心对象

| 对象 | 接口/类 | 描述 |
| --- | --- | --- |
| **Storage Facility** | `IStorageFacility` / `StorageFacility` | 物理存储设施，是存储层次结构的顶层容器，如实验室、储藏室或冰箱。它包含地址和联系信息。 |
| **Storage Container** | `IStorageContainer` / `StorageContainer` | 通用存储容器，可以容纳其他容器，形成嵌套结构。例如，一个冰箱（Facility）可以包含多个架子（Container），每个架子又可以包含多个盒子（Container）。它具有预设温度属性。 |
| **Storage Samples Container** | `IStorageSamplesContainer` / `StorageSamplesContainer` | 专门用于存放样品的容器，定义了容器的布局（行和列），并管理样品在其中的具体位置。 |
| **Storage Position** | `IStoragePosition` / `StoragePosition` | 容器内的具体存储位置，用于精确定位样品。 |

### Storage Facility 核心字段

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `Phone` | String | 设施的联系电话。 |
| `EmailAddress` | String | 设施的联系电子邮件地址。 |
| `Address` | Address | 设施的物理地址。 |

### Storage Container 核心字段

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| `Temperature` | Float | 容器的预期存储温度。 |
| `Rows` | Integer | （在样品容器中可见）容器布局的行数。 |
| `Columns` | Integer | （在样品容器中可见）容器布局的列数。 |

### 关联对象

- `AnalysisRequest`: 样品（分析请求）可以被存放在 `StorageSamplesContainer` 的特定位置。当样品被存储或取出时，其工作流状态会相应改变（如变为 `stored` 状态）。