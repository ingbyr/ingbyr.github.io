# 系统架构师考试11-未来信息综合技术


<!--more-->

## 信息物理系统 Cyber-Physical Systems, CPS

- CPS集成先进的感知、计算、通信、控制等信息技术和自动控制技术
- 构建了物理空间与信息空间中人、机、物、忽然块、信息等要素相互映射、适时交互、高效协同
- 实现系统内资源配置和运行的按需响应、快速迭代、动态优化
- 最终目的：实现资源优化配置
- 关键点：数据的自动流动
- CPS体系架构
    - 单元级
    - 系统级
    - SoS级
- CPS技术体系
    - 总体技术：系统架构、异构系统集成、安全技术、实验验证技术
    - 支撑技术：智能感知、嵌入式软件、数据库、人机交互、中间件、SDN、物联网、大数据
    - 核心技术：虚实融合控制、智能装备、MBD、数字孪生技术、现场总线、工业以太网
- 应用场景
    - 智能设计
        - 产品及工艺设计
        - 生产线/工厂设计
    - 智能生产
        - 设备管理应用场景
        - 生产管理应用场景
        - 柔性制造应用场景
    - 智能服务
        - 健康管理
        - 智能维护
        - 远程征兆性诊断
        - 协同优化
        - 共享服务
    - 智能应用
        - 无人装备
        - 产业链互动
        - 价值链共赢

## 人工智能技术

- 目的：了解智能的实质，产生一种新的能以人类智能类似的方式做出反应的智能机器
- 分类
    - 弱人工智能
    - 强人工智能
- 关键技术
    - 自然语言处理 NLP
    - 计算机视觉 CV
    - 知识图谱 KG
    - 人机交互 HCI
    - 虚拟现实或增强现实 VR/AR
    - 机器学习 ML
        - 学习模式
            - 监督学习
            - 无监督学习
            - 半监督学习
            - 强化学习
        - 学习方法
            - 传统机器学习
            - 深度学习

## 机器人技术

机器人4.0核心技术

- 云边端无缝协同计算
- 持续学习与协同学习
- 知识图谱
- 场景自适应
- 数据安全

机器人分类

- 按照控制方式
    - 操作机器人
    - 程序机器人
    - 示教再现机器人
    - 智能机器人
    - 综合机器人
- 按照应用行业
    - 工业机器人
    - 服务机器人
    - 特殊领域机器人

## 边缘计算

- 云计算擅长全局性、非实时、长周期的大数据处理与分析，
  能够在长周期维护、业务决策支撑等领域发挥优势
- 边缘计算更适用局部性、实时、短周期数
  据的处理与分析，能更好地支撑本地业务的实时智能化决策与执行
- 两者协同关系

### 分类

- 云边缘：逻辑上仍属于云服务，在云服务边缘侧延伸
- 边缘云：在边缘侧构建中小规模云服务能力
- 云化网关：在边缘侧提供协议/接口转换、边缘计算等能力

### 特点

- 联接性
- 数据第一入口
- 约束性
- 分布性

### 边云协同

- 资源协同
    - 边缘节点提供计算、存储等基础设施资源，具有本地资源调度功能
    - 云端下发组员调度管理策略
- 数据协同
    - 边缘节点负责现场/终端数据采集，初步处理与分析，上报云端
    - 云端提供海量数据存储、分析与价值挖掘
    - 实现高效低成本地对数据进行生命周期管理与价值挖掘
- 智能协同
    - 边缘节点使用AI模型执行推理，分布式智能
    - 云端集中式模型训练，下发边缘节点
- 应用管理协同
    - 边缘节点提供应用部署运行环境，应用生命周期管理调度
    - 云端提供应用开发、测试环境，生命周期管理能力
- 业务管理协同
    - 边缘节点提供模块化、为辅化的应用/数字孪生/网络等应用实力
    - 云端提供业务编排能力
- 服务协同
    - 边缘节点：ECSaaS
    - 云端提供SaaS，服务分布策略

### 边缘计算安全

- 运行维护
    - 应用监控
    - 应用审计
    - 访问控制
- 数据安全
    - 轻量级数据加密
    - 数据安全存储
    - 敏感数据处理
- 可信网络
    - 运营商安全保障
- 端到端防护
    - 威胁检测
    - 态势感知
    - 安全管理编排
    - 安全事件应急响应
    - 柔性防护

## 数字孪生

- 关键技术
    - 建模
    - 仿真
- 应用
    - 制造
    - 产业
    - 城市
    - 战场

## 云计算和大数据技术

- 云计算服务
    - 软件即服务 SaaS
    - 平台即服务 PaaS
    - 基础设施即服务 IaaS
- 云计算部署模式
    - 公有云
    - 社区云
    - 私有云
    - 混合云

大数据

- 特点
    - 大规模 Volume
    - 高速度 Velocity
    - 多样化 Variety
    - 价值 Value
- 步骤
    - 数据获取和记录
    - 数据抽取和清洗
    - 数据集成、聚集和表示
    - 查询处理、数据建模和分析
    - 解释
