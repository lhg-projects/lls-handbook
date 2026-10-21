# 变更单：中文默认 + 界面汉化 + 语音 AI（DeepSeek）

给执行的 AI 看：这份文档说的是「改什么、改到什么程度算完成」。按优先级从上往下做，每完成一项跑一次验收命令，不要跳步。

## 0. 前置规则（所有变更都受这几条约束）

1. **中文优先**：界面默认语言是简体中文，不是跟随系统语言。英文只在用户主动切到 EN 时出现。
2. **不动内部标识**：`.cuerecord` 目录、`cuerecord://` scheme、Keychain service `com.nolanlai.*`、UTI `com.nolanlai.cuerecord.document`、Swift 类型名 `CueRecord*` 全部保持原样。改名会让老用户的 Vault 和已存 API Key 失联。
3. **小步提交**：一次提交只做一件事，提交信息写清「改了什么 + 为什么」。
4. **先跑测试再交付**：`scripts/run-recording-core-tests.sh` 必须全 PASS；`scripts/build-local.sh` 必须 BUILD SUCCEEDED。没有这两个输出不算完成。
5. **不许留半成品**：注释掉的代码、临时 TODO、调试 print 提交前清掉。

## 1. 界面默认中文

| 项 | 位置 | 验收 |
|---|---|---|
| 默认语言 | `CueRecord/InterfaceLanguage.swift`，`InterfaceLanguageSettings.init` 的兜底值 | 全新安装（删掉 `com.liulaoshi.llsstudio` 偏好文件）后启动，界面是中文 |
| 已保存偏好优先 | 同上 | 手动切到 EN 后重启，仍是 EN |

## 2. 界面汉化补全

原始状态：`InterfaceLanguage.zhHans` 有 369 条词条，但界面上还有 68 处 `Text("…")` 直接写英文字面量，绕过了本地化层，导致中文模式下中英混杂。

改法：把这些字面量包进本地化层，字面量本身不动（词条表里大多已有对应中文）。

```swift
// 改前
Text("Next Page")
// 改后
Text(InterfaceLanguageSettings.shared.text("Next Page"))
```

覆盖范围：`NotchOverlayController.swift`、`SettingsView.swift`、`ExternalDisplayController.swift`、`CueRecordService.swift`、`UpdateChecker.swift`。

新增词条（词条表没有的）直接补进 `zhHans`，例如：

```swift
"Voice AI": "语音 AI",
"Voice commands": "语音口令",
"Let the AI model interpret unclear commands": "认不出的口令交给第三方模型解析",
```

验收：中文模式下把设置、提词器浮窗、更新提示、导入错误提示全点一遍，不出现英文句子。专有名词（API Key、DeepSeek、PPTX、TC）保留英文。

## 3. 第三方 AI 接口（DeepSeek）接入语音

### 3.1 已有的底座（不要重写）

- `CueRecord/AI/ScriptAI.swift`：`AIChatCompletionsClient`（OpenAI 兼容 chat/completions）、`AIProviderAPIKeyStore`（Keychain）
- `CueRecord/AI/AIModelCatalog.swift`：provider 目录，`defaultProviderID = "deepseek"`，模型选择持久化在 `AIScriptModelSelectionStorage`

### 3.2 新增：语音口令解析

文件：`CueRecord/AI/AIVoiceAssistant.swift`

两层设计，规则先跑，模型兜底：

1. **规则层**（离线、零延迟）：下一页 / 翻页 / 上一页 / 第一页 / 最后一页 / 第 N 页 / 暂停 / 继续 / 结束录制。命中即执行，不发网络请求。
2. **模型层**：规则认不出，且用户开了「模型解析」、配了 Key，才请求 DeepSeek。prompt 要求只返回 JSON：

```json
{"action":"next_page|previous_page|first_page|last_page|jump_page|pause|resume|finish|none","page":null,"confidence":0.8}
```

解析代码必须容忍：代码围栏、前后多余文字、confidence < 0.4 丢弃。

**已知坑（已修，别改回去）**：中文数字「一」也算数字，如果直接扫全句数字，「下一页」会被解析成第 1 页。页码解析必须先排除「下一页 / 上一页 / 下一张」，再按「第 / 跳到 / page」这类引导词切出数字区。

### 3.3 客户端扩展

在 `AIChatCompletionsClient` 增加通用方法，语音口令和 AI 断句共用同一套 provider、模型、密钥：

```swift
func complete(systemPrompt: String, userPrompt: String,
              model configuration: AIChatModelConfiguration,
              apiKey: String?, temperature: Double = 0,
              timeout: TimeInterval = 20) async throws -> String
```

### 3.4 设置界面

位置：设置 → 语音（`SettingsView.swift` 的 `guidanceTab`，插在麦克风选择之前）。

必须有的元素：

- 开关「语音口令」
- 开关「认不出的口令交给第三方模型解析」（口令关闭时置灰）
- 一行状态：`provider · 模型 · API Key 已保存/未填写`，用图标区分状态

### 3.5 提词器接线

位置：`CueRecord/NotchOverlayController.swift` 的 `NotchOverlayView`。

规则：

- 监听 `speechRecognizer.lastSpokenText`
- 只处理长度 ≤ 14 字的候选（超长视为讲稿正文，不拦截）
- 稳定 0.6 秒后仍未被新文本覆盖，才送去解析（避免半句触发）
- 动作映射：翻页 → `shouldAdvancePage`；结束 → `shouldDismiss`；跳页 → `content.jumpToPageIndex`；暂停/继续 → `isPaused`

验收：开启口令后说「下一页」翻页、「第 3 页」跳转、「结束录制」收浮窗；关掉开关后说同样的词不应有任何反应。

## 4. 测试

`Tests/RecordingCoreTests/main.swift` 增加 `AIVoiceCommandParsing`，覆盖：

- 中文口令：下一页 / 翻页吧 / 上一页 / 暂停一下 / 继续 / 结束录制
- 页码：第 3 页 / 跳到第 9 页（超出总页数应收敛）/ 第一页
- 反向用例：正常讲稿句子不得被识别成口令；长句不是候选
- 模型层：带代码围栏的 JSON、纯 next_page、非 JSON、低置信度

`scripts/run-recording-core-tests.sh` 的文件清单要补上 4 个 AI 源文件，否则测试编译不过。

## 5. 完成标准

```bash
./scripts/run-recording-core-tests.sh   # 全 PASS，含 AIVoiceCommandParsing
./scripts/build-local.sh                # BUILD SUCCEEDED
```

加一条人工验收：全新启动 → 界面中文 → 设置里能看到「语音 AI」区块 → 说「下一页」提词器翻页。

## 6. 本单之外的事（别自己动手）

- 组织改名 `lhg-projects` → `lls-projects`：需要 `admin:org` 权限，仓库内所有 README 的旧组织链接要同步替换，改完核对一遍。
- `lhg-skills` / `lhg-plugs` / `lhg-x` 三个组织是否改名：待定，不要擅自处理。
- 产品名与图标：已经定型（`刘老师 · 录课台` / `LLSStudio` / 石墨黑），不要再改。
