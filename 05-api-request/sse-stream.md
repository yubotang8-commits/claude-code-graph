# claude-code-graph

Claude 流式链路可交互播放器 —— 单文件 HTML，零依赖，双击即开。

把 Claude Messages API 的完整流式链路做成**可播放、可暂停、可单步**的动画讲解页：

> 用户 → Agent 服务 → 推理服务 → 模型（GPU）
> 模型自回归采样 token → 推理服务打包成 SSE 事件 → Agent 逐事件拼装 Message → 工具执行 → 第二轮流

## 打开方式

- 下载 `index.html`，双击用任意现代浏览器打开（无任何外部依赖）；
- 或开启 GitHub Pages 直接在线播放。

## 里面有什么

**88 个子步骤，严格逐跳（hop-by-hop）**：每个事件拆成「① 产生/发出 → ② 线上传输 → ③ 到达写入」三段，**接收方内存只在数据包到达后才变化**——没有瞬移。

完整还原一轮流式响应的真实结构：

| Message 槽位 | 事件序列 |
|---|---|
| `content[0] · thinking` | `thinking_delta` 逐段流式 + 关块前 `signature_delta` 完整性校验 |
| `content[1] · text` | `text_delta` 逐字追加 |
| `content[2] · tool_use` | `input_json_delta` 部分 JSON 逐片拼接 → 关块时解析 |
| 顶层 | `message_delta` 盖最终值（**每条流仅 1 条**）→ `message_stop` 定型 |

## 功能

- **传输控制**：播放 / 暂停（空格）、单步 ← →、进度条逐段点击跳转、0.5×–4× 倍速、重置
- **聚光灯模式**（🔦 可开关）：当前节点泳道高亮，其余去饱和
- **⚡ 断流演示**（按钮或 B 键）：在工具参数传输中途切断连接，展示部分状态（thinking/text 完整、tool_use 的 JSON 拼一半、message_delta 未到达）与分代际恢复策略
- **机制注解**：prompt token 化 → prefill → decode 每步 1 token → 采样到 EOT（turn 结束标记）后推理服务收尾——以独立视觉类别的日志行呈现，与真实线上事件区分
- **线报文 console**：官方 tool-use 示例的真实事件逐行照录（可截断、未改写）

## 依据

事件序列、字段与报文内容依据 [Claude 官方流式文档](https://platform.claude.com/docs/en/build-with-claude/streaming) 整理；本页为讲解用途的可视化，非官方产物。
