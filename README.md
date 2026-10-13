# AI Customer Service (银行AI客服)

本仓库旨在为银行客服相关的 AI 产品与工程团队提供一个开源资源汇总、落地模板与速查手册。内容面向业务产品经理（尤其是银行客服方向）、工程与数据团队，涵盖对话系统、知识检索（RAG）、语音到文本、呼叫中心集成、事件流/中台对接与观测监控等主题。

目录结构（建议）

- README.md — 本文件
- /conversational — 对话框架与示例（Rasa / Botpress）
- /retrieval — RAG / 向量检索（Haystack / Milvus / Weaviate）
- /asr — 语音识别 / 转写（Vosk / Whisper 示例）
- /telephony — 呼叫中心 / PBX（Asterisk / FreePBX）, IVR 流程示例
- /integrations — Kafka / Debezium / CRM / Activepieces 集成示例
- /analytics — Prometheus / Grafana / OpenSearch 仪表盘示例
- /examples — 参考架构、数据契约、API 设计与示例流
- /docs — 深入落地文档、合规与隐私指南

核心目标

1. 帮助 PM 与工程快速搭建可验证的 AI 客服 MVP（文本/语音）
2. 给出面向银行场景的技术选型与可落地的架构参考
3. 提供监控、合规、数据质量与线上治理的操作检查表
4. 汇总高质量开源项目与对接示例，方便工程快速集成

推荐开源项目（优先级/用途）

- 对话/NLU
  - Rasa (对话管理 + 自定义 action)：https://github.com/RasaHQ/rasa
  - Botpress (可视化、低代码)：https://github.com/botpress/botpress
  - Microsoft Bot Framework： https://github.com/microsoft/botframework-sdk

- RAG / 知识检索 / 向量 DB
  - Haystack (RAG pipeline)：https://github.com/deepset-ai/haystack
  - Weaviate (向量 DB + 知识图谱支持)：https://github.com/weaviate/weaviate
  - Milvus (大规模向量检索)：https://github.com/milvus-io/milvus
  - OpenSearch (全文/日志/聚合)：https://github.com/opensearch-project/OpenSearch

- ASR / 通话转录
  - Vosk (本地/流式 ASR)：https://github.com/alphacep/vosk-api
  - Whisper (离线/批量转录 + 社区前端)：https://github.com/openai/whisper

- 呼叫中心 / PBX / WebRTC
  - Asterisk (PBX / IVR)：https://github.com/asterisk/asterisk
  - FreePBX (Asterisk 管理面板)：https://github.com/FreePBX
  - Jitsi Meet (WebRTC 视频/语音接入)：https://github.com/jitsi/jitsi-meet

- 事件流 / 集成
  - Apache Kafka (事件总线)：https://github.com/apache/kafka
  - Debezium (数据库 CDC -> Kafka)：https://github.com/debezium/debezium
  - Activepieces (工作流/自动化)：https://github.com/activepieces/activepieces

- CRM / 工单
  - SuiteCRM： https://github.com/salesagility/SuiteCRM
  - OpenCRX： https://github.com/opencrx/opencrx

- 监控 / 可观测性
  - Prometheus： https://github.com/prometheus/prometheus
  - Grafana： https://github.com/grafana/grafana
  - OpenSearch（日志）：https://github.com/opensearch-project/OpenSearch

银行场景下的参考架构（文本+语音混合）

1. 前端渠道
  - Web chat / Mobile app / IVR / 电话录音接入（SIP）

2. 接入层
  - API Gateway -> 验证/鉴权 -> 路由到文本或语音处理流水线

3. 语音处理
  - 呼叫录音流入 -> ASR（Vosk/Whisper）-> 转写文本 -> 意图识别/实体抽取

4. 文本理解与对话
  - NLU（Rasa/自研）用于意图分类与槽位抽取
  - 规则层/对话管理（Rasa stories / Botpress flows）
  - 知识检索（Haystack + Milvus/Weaviate）用于回答依赖文档或政策的查询
  - LLM（如私有模型或调用外部 API）用于生成复杂回应，RAG 控制来源与可解释性

5. 业务与工单
  - 若需要人��接入，生成工单并写入 CRM（SuiteCRM）或把事件推入工单队列
  - 使用 Kafka 做异步事件与审计日志

6. 监控与治理
  - 指标采集（Prometheus）: latency, ASR WER, intent accuracy, conversation success rate, handoff rate
  - 日志/搜索（OpenSearch）: 呼叫录音索引、转写文本、QA 查询历史
  - 仪表盘（Grafana）: 业务与模型双视角

关键度量（KPI & ML 指标）

- 业务指标：首次接触解决率 (FCR)、转人工率、平均处理时长 (AHT)、客户满意度 (CSAT)、合规/审计通过率
- 模型/系统指标：ASR WER (word error rate)、NLU intent accuracy、检索命中率、RAG 的来源覆盖率与引用可信度
- 运营指标：系统延迟（端到端）、排队时长、告警频率、宕机恢复时间

合规、隐私与安全（银行必看）

- 最小化录音与个人信息保留策略（PII redaction）
- 在录音/文本中进行 PII 检测并可选脱敏存储
- 审计日志与操作追踪：谁查看了哪段录音或敏感对话
- 数据主权与模型托管：优先考虑本地/私有部署以满足监管要求

示例快速上手（最小可验证原型 MVP）

目标：构建一个“常见问题自动答复 + 工单触发”原型

步骤：
1. 准备 FAQ 文档（政策、流程、账户常见操作）并做简单分段
2. 使用 Haystack + Milvus 建立检索索引，能对自然语言查询返回文档段落
3. 用 Rasa 做意图识别：识别 FAQ 查询 vs 需要工单/敏感信息的场景
4. 接入 Whisper/Vosk 做语音 -> 文本（如果需要电话通道）
5. 通过一个小型 webhook 把“需要人工处理”的交互推入 SuiteCRM 或推 Kafka 供人工工单系统消费
6. 监控：设置 Grafana 仪表盘显示 FCR、转人工率、ASR WER

下一步（我可以为你执行）

- 把本 README 提交到仓库（已完成） ✅
- 在 /examples 下添加一份“FAQ -> RAG + Rasa” 的代码样例与 docker-compose 启动脚本
- 在 /docs 下添加银行合规与 PII 脱敏的详细操作清单
- 为你生成 PRD 模板与面向项目的里程碑模板

如果你愿意，我下一步可以直接继续补充 /examples 和 /docs 的模板内容。