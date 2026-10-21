# 改名决策与执行手册：CueRecord → 刘老师 · 录课台（LLSStudio）

给接手执行的人或 AI 看。这份文档回答三个问题：为什么要改名、已经改到什么程度、剩下的怎么改完。

---

## 1. 一句话说清

我们 fork 了一个开源的 macOS 录屏提词器（`nolangz/CueRecord`，Apache-2.0），修好了它的分轨录制 bug，然后把它改造成自己的产品：名字、图标、Bundle ID、文案、GitHub 仓库全部换成「刘老师 · 录课台」。**代码层面的改名已经完成并通过构建与测试**；GitHub 组织改名和跨仓库链接替换还没做，卡在一个权限上。这一步需要执行者接手。

---

## 2. 为什么改（前因后果）

### 2.1 起点是一次 bug

2026-10-21 从上游 `CueRecord v2.0.4` fork，起因是录制时画面被拆成两个文件（屏幕轨 + 摄像头轨），从未合成。根因有三处：编辑窗复用旧 take、全局布尔挡住后续 take、渲染请求被静默丢弃。三处已修，有回归测试。

### 2.2 修完 bug 暴露了身份问题

能用之后才发现，整个产品还是上游的名字：应用图标是上游作者的 `C` 字标，包名 `com.nolanlai.cuerecord` 带着上游作者的名字，更新检查地址指向上游仓库，界面文案写着 `CueRecord`。

对一个要对外发的产品，这意味着：

- 粉丝在 Dock 里看到的是别人的品牌；
- 更新按钮会去检查上游仓库，可能被上游的版本覆盖；
- 没法做商标——你注册的是别人的名字；
- 和已有的内容体系对不上。

### 2.3 身份体系要求产品落进同一个命名空间

用户的自媒体身份是「刘老师（Liu Lao Shi）」，缩写 **LLS**：

- 内容主线：AI 实操，连载《WorkBuddy 从零基础到入门》；
- 技能包命名：`lls-` 前缀（`lls-writing`、`lls-deep-research`、`lls-craft`…），粉丝在技能市场搜 `lls-` 一次找全；
- 仓库命名：`lls-` 前缀。

如果 App 还叫 CueRecord，粉丝看到的是两个互不相干的东西。命名必须收敛到一个入口。

### 2.4 商标要求有一个自造图形

「刘老师」是通用称谓，单独拿去做商标显著性不足。可注册的主体是**图形 + 文字组合**。所以这次不是简单换个名字，而是配套做了图形：石墨黑方底 + 白色课件卡（三行讲稿）+ 圆形出镜窗（人像）+ 录制红点，四个元素各对应一个真实功能。这个图形才是商标的显著性来源。

### 2.5 结论：中英双名

| 用途 | 名字 |
|---|---|
| 给人看（Dock / Finder / 菜单栏 / 关于页 / 宣传） | 刘老师 · 录课台 |
| 给机器看（`.app` 文件名、进程名、仓库名、Bundle ID、脚本） | `LLSStudio` / `lls-studio` / `com.liulaoshi.llsstudio` |

---

## 3. 已经做完的

### 3.1 代码与构建

| 项 | 值 | 位置 |
|---|---|---|
| 产物名 | `LLSStudio.app` | `PRODUCT_NAME = LLSStudio` |
| 显示名 | 刘老师 · 录课台 | `INFOPLIST_KEY_CFBundleDisplayName` |
| Bundle ID | `com.liulaoshi.llsstudio` | `PRODUCT_BUNDLE_IDENTIFIER` |
| 图标 | 石墨黑四元素，16–1024 全尺寸 | `CueRecord/Assets.xcassets/AppIcon.appiconset/` |
| 侧边栏标记 | 同款图形的小尺寸切图 | `CueRecord/Assets.xcassets/CueRecordLogo.imageset/` |
| 界面文案 | 40 处字符串里的产品名改为「录课台」 | 全 `CueRecord/**/*.swift` |
| 权限弹窗 | 四条 `NSxxxUsageDescription` 改成中文说明 | `Info.plist` |
| 文档类型名 | `录课台文稿`、右键「用录课台打开」 | `Info.plist` |
| 更新检查 | 指向 `lls-projects/lls-studio` | `CueRecord/UpdateChecker.swift` |
| 帮助链接 | 指向新仓库 | `CueRecord/CueRecordApp.swift` |
| DMG / 卷名 | `LLSStudio.dmg` / `LLSStudio` | `build.sh` |
| 品牌规范 | 名字、配色、图标四元素、文案规则 | `BRAND.md` |
| 上游归属 | 署名改为「刘老师 / lls-projects」，保留 Apache-2.0 声明 | `NOTICE` |

验证证据：

```bash
./scripts/run-recording-core-tests.sh   # 21 项全 PASS
./scripts/build-local.sh                # BUILD SUCCEEDED
/usr/libexec/PlistBuddy -c 'Print :CFBundleDisplayName' \
  ~/Applications/LLSStudio.app/Contents/Info.plist   # 刘老师 · 录课台
```

提交：

- `f95148c` 品牌化：图标、产品名、Bundle ID、BRAND.md
- `6405dbb` 中文默认、界面汉化补全、DeepSeek 语音口令

### 3.2 GitHub

| 项 | 结果 |
|---|---|
| 仓库改名 | `lhg-CueRecord` → `lls-studio`；`lhg-lanclip` → `lls-lanclip`；`lhg-jianclip` → `lls-jianclip` |
| 仓库描述 | 三个仓库全部重写 |
| topics | `macos` / `screen-recorder` / `teleprompter` / `lls` / `course-recording` |
| 组织主页 | `.github` 仓的 `profile/README.md` 整篇重写为 LLS 入口页 |
| 组织描述 | 改为「刘老师（LLS）的产品仓：AI 实操课程配套工具与开源项目」 |
| 约束手册 | 新建 `lls-handbook`（public），含总纲 + `changes/` 变更单 |
| 断链修复 | `srt-craft` 的 issue 链（原指向错误组织）、`quickshot-lan` 里的旧仓库名 |

---

## 4. 绝对不许动的标识

这些不是品牌，是**数据契约**。改了会导致老用户的 Vault、已存 API Key、已有成片失联，收益是零。

| 标识 | 位置 | 为什么不能改 |
|---|---|---|
| `.cuerecord` 元数据目录 | 工程目录内 | 老 Vault 靠它识别工程 |
| `.cuerecord` / `takeone` / `cueshot` 文档扩展名 | `Info.plist`、代码 | 老文件的双击打开 |
| UTI `com.nolanlai.cuerecord.document` | `Info.plist` | 已注册的类型标识 |
| URL scheme `cuerecord://` | `Info.plist` | 外部调用链 |
| Keychain service `com.nolanlai.cuerecord.ai-provider` / `.deepseek` | `ScriptAI.swift` | 用户已存的 API Key 在这里 |
| Swift 类型名 `CueRecord*` | 全项目 | 无用户价值，改它只增加 diff |
| DispatchQueue 标签 | 各文件 | 只在调试时可见 |

判断标准：**用户看得见的地方跟着品牌走，用户数据依赖的地方一个字节都不动。**

---

## 5. 还没做完的（执行者从这里接手）

### 5.1 组织改名：`lhg-projects` → `lls-projects`

状态：接口返回 `This API operation needs the "admin:org" scope`。当前 token 只有 `gist, read:org, repo`。

**第一步，授权**（这一步必须人来做，AI 拿不到交互式授权）：

```bash
gh auth refresh -h github.com -s admin:org
gh auth status      # 确认 Token scopes 里出现 admin:org
```

**第二步，改名**：

```bash
gh api -X PATCH orgs/lhg-projects -f login=lls-projects --jq '.login'
gh api orgs/lls-projects --jq '.login'    # 应返回 lls-projects
```

**第三步，改本地 remote**（如果本机有这个仓库的克隆）：

```bash
git remote set-url origin https://github.com/lls-projects/lls-studio.git
git remote -v
```

**影响面**（GitHub 会自动重定向旧地址，但要知道自己在动什么）：

- 23 个仓库的全部 URL 换前缀，旧链接 301 跳到新地址；
- raw.githubusercontent.com 上的旧路径仍然可用（走重定向）；
- 任何硬编码 `lhg-projects` 的文档、脚本、徽章，字面上仍是旧名，需要顺手替换；
- 组织名一旦放开，原名字会被别人抢注，所以改名后不要回退。

**回滚**：`gh api -X PATCH orgs/lls-projects -f login=lhg-projects`（前提是新名字是你自己改回来的）。

### 5.2 跨仓库链接替换

改名后，以下 5 个仓库的 README 里还有旧组织地址，需要替换 `lhg-projects` → `lls-projects`：

| 仓库 | 待替换处数 |
|---|---|
| `lls-studio` | 1 |
| `quickshot-lan` | 1 |
| `srt-craft` | 4 |
| `pain-bank` | 1 |
| `wx-precheck` | 1 |

参考脚本（逐仓替换，先看 diff 再提交）：

```bash
for r in lls-studio quickshot-lan srt-craft pain-bank wx-precheck; do
  sha=$(gh api "repos/lls-projects/$r/contents/README.md" --jq .sha)
  gh api "repos/lls-projects/$r/contents/README.md" --jq .content | base64 -d \
    | sed 's|lhg-projects|lls-projects|g' > /tmp/readme-$r.md
  echo "--- $r: $(grep -c lls-projects /tmp/readme-$r.md) 处"
done
```

确认无误后逐个 PUT 回 GitHub，或者直接 clone、改、push。

另有 1 处不是地址而是一句话：`srt-craft-studio` 的 README 里写着「同为 lhg 的 MIT 代码」，按语境应改为「同为 LLS 的 MIT 代码」。

### 5.3 另外三个组织的去留（**建议不改，需人拍板**）

| 组织 | 仓库数 | 建议 | 理由 |
|---|---|---|---|
| `lhg-skills` | 19 | 不改 | 技能市场里成员名已经是 `lls-*`，组织名几乎不对外露出；改名会让已发布的 raw 安装地址多一跳 |
| `lhg-plugs` | 88 | 不改 | 88 个插件的安装链接散落在各处，改名风险不对等 |
| `lhg-x` | 2 | 可改可不改 | 取决于 X 账号简介怎么写 |

**执行者注意：这三个组织没有授权，不要擅自改名。**

---

## 6. 验收清单

- [ ] `gh auth status` 的 scopes 含 `admin:org`
- [ ] `gh api orgs/lls-projects --jq .login` 返回 `lls-projects`
- [ ] 23 个仓库都能通过新地址访问，`gh repo list lls-projects` 数量一致
- [ ] 本机 `git remote -v` 指向新地址，`git push` 成功
- [ ] 5 个 README 里的旧组织地址清零：`grep -r lhg-projects` 无结果
- [ ] `.github/profile/README.md` 里的仓库链接全部可点
- [ ] `lls-handbook` 能从组织主页点到
- [ ] 另外三个组织未被改动
- [ ] App 侧没有被顺手改动：`./scripts/run-recording-core-tests.sh` 仍全 PASS，`./scripts/build-local.sh` 仍 BUILD SUCCEEDED

---

## 7. 常见误判

| 现象 | 真实原因 | 怎么处理 |
|---|---|---|
| 改名后旧链接 404 | 重定向未生效或缓存 | 等几分钟，或用新地址直接访问一次 |
| `gh api orgs/lls-projects` 返回 404 | 多半是 token 缺 `admin:org`，不是组织不存在 | 回到 5.1 第一步重新授权 |
| clone 下来发现模型文件很小 | 没装 Git LFS / 没 pull LFS 对象 | `git lfs install && git lfs pull`，5 个资产应大于 100 MB |
| App 改了 Bundle ID 后录制没权限 | 新 Bundle ID 等于新 App，TCC 记录从零开始 | 重新授权屏幕录制 / 麦克风 / 摄像头 |
| 改名后粉丝说找不到仓库 | 他们用的是旧地址，重定向会兜住 | 不用处理，重定向长期有效 |
