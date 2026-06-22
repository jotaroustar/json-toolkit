# JSON 工具箱（json-toolkit）

格式化 / Schema 校验生成 / JSONPath 查询 / 结构化 Diff 对比，四合一的纯前端 JSON 工具。核心引擎全部本地零 AI 成本，附带两个可选的 AI 辅助入口。

线上地址：`https://jotarou.com/tools/json-toolkit/`

## 功能

| Tab | 功能 | 是否需要 AI |
|---|---|---|
| 格式化 | 美化/压缩、树形展开折叠、语法高亮、错误定位（点击可跳转光标）、大文件自动降级 | 否，纯本地 |
| Schema | 校验 JSON 是否符合给定 Schema；从示例反向推断生成 Schema 草稿 | 否，纯本地推断；低置信度字段可选点击 💡 问 AI |
| 查询 | JSONPath 子集实时查询，结果联动高亮+自动定位 | 否，纯本地引擎；也可用自然语言描述需求，点击 ✨ 让 AI 生成表达式 |
| Diff | 两份 JSON 结构化对比，增删改高亮，"仅看差异"过滤 | 否，纯本地算法 |

## AI 辅助功能说明（可选，默认不触发）

- **Tab3 ✨ AI 生成表达式**：只把数据的*结构骨架*（字段名+类型，不含真实数据值）发给 AI，让它把自然语言需求翻译成 JSONPath 表达式。AI 返回结果会先经过本地 `tokenizePath()` 校验语法，校验失败不会执行。
- **Tab2 💡 问AI**：当某字段被本地规则判定为"类型不一致、置信度低"时，可以点击该按钮把这个字段的真实示例值（最多8条）发给 AI 协助判断该统一成什么类型。这是唯一会发送真实数据的地方，且范围仅限单个字段、用户主动触发。
- 两者都走固定网关 `https://jotarou.com/v1/chat/completions`，Key 和模型名存在 localStorage，通过 Header 右上角 🔑 按钮配置。

## 技术要点 / 已知限制

- **大文件渲染降级阈值**（`LARGE_SIZE_BYTES` / `LARGE_NODE_COUNT`，目前 2MB / 5000 节点）是开发期保守估计值，**部署前必须用真实大 JSON 样本实测调整**，不能直接信这两个数字。
- 错误定位的行列提示依赖 `position N` 格式的报错信息（Chrome/V8 格式），Firefox/Safari 会优雅降级为"无行列提示，只显示报错原文"。
- 大文件场景目前是"整体降级为纯文本"，不是真正的虚拟滚动（按 Tab/Diff 树的局部渲染）。如果实测发现降级太频繁、体验不够用，再考虑做虚拟滚动。
- Diff 算法是简化版结构化对比：对象按 key 比较，数组按下标对齐，**不做 LCS 重对齐**。如果数组重排场景常见，这里是后续升级点。
- JSONPath 只实现子集：`$` `.key` `[index]` `[*]` `..key`（递归下降）`['key']`（引号key）+ 基础 filter `[?(@.x<10)]`。不支持完整 JQ 语法、函数调用、复杂脚本表达式。
- AI 调用的 `model` 字段当前默认值是占位字符串 `deepseek-chat`，**部署前请在 Header 🔑 设置里换成你网关实际可用的模型标识**。
- AI 相关的 SSE 流解析、数据脱敏逻辑都是用构造数据在沙盒环境里模拟测试的（沙盒访问不了 `jotarou.com`），**上线后必须拿真实 Key 完整走一遍两条 AI 链路**，不能只信任开发期的 mock 测试结果。

## 文件结构

单文件 `index.html`，无构建步骤，无外部依赖（CSP 禁 CDN，所有逻辑内联）。

## 强制规范核对（与①-⑬一致）

- [x] `esc()` 转义 `& < > " '` 五字符
- [x] 隐私声明 footer（本工具核心引擎本地+AI入口需主动触发的准确版本）
- [x] AI 接口固定 `jotarou.com/v1/chat/completions`，OpenAI 兼容格式
- [x] 无外部 CDN
- [x] 右上角 ⊞ 返回按钮，footer `/tools/` 尾斜杠
- [x] 版权注释在 `<!DOCTYPE html>` 之前
- [x] CSS 设计 token + `[data-theme=dark]` + 🌙/☀️ 切换 + localStorage 持久化
- [x] AI 流式调用：`stream:true` + 手动 SSE 解析 + JSON 解析失败兜底 + toast 替代 alert
- [x] 主文件名固定 `index.html`
- [x] 涉及规则引擎/大文件：Web Worker 防卡死（解析层）+ 渲染层自动降级
- [x] 高亮联动：本工具用节点级直接 DOM 引用定位（非字符位置切片），架构上规避了"去标签还原=原文"的校验场景

© 2026 jotarou.com. All rights reserved. Unauthorized copying or commercial use is prohibited.
