# dsh-free-search · Soulize fork

**DeepSeek Harness 搜索增强插件，主力围绕 Exa + AnySearch，支持 Multi Search、全局引擎开关与优先级、自动回退，以及精简的动态系统提示词。**

> 基于 [DDDMUC/dsh-free-search](https://github.com/DDDMUC/dsh-free-search) 深度改造。这个 fork 已在搜索路由、设置 UI、多引擎并发、fallback 策略、提示词注入和 DSH 版本兼容等方面与上游产生较大差异；后续功能与行为以本仓库为准。

[中文](#中文) · [English](#english)

---

## 中文

<div align="center">
  <a href="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free1.png">
    <img src="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free1.png" alt="免费引擎设置 (Bing)" width="820" />
  </a>
  <br>
  <sub>▲ 免费引擎（以Bing为例）</sub>
</div>

### 为什么需要它

dsh 默认的搜索 provider 依赖 DeepSeek 官方 API key（`DEEPSEEK_API_KEY`）。如果你：
- 没有（或不想用）DeepSeek 官方 key，
- 用的是 opencode-go 这类网关（其 OpenAI 兼容端点不支持 `web_search` 工具），

……那么内置搜索必然失败，agent 会告诉你"无法联网"。

这个 fork 不以“堆最多引擎”为主要目标，而是把 **Exa + AnySearch** 作为主力免费搜索组合，其它引擎作为可选 fallback 或交叉验证来源；路由、并发合并、失败回退和提示词成本控制都由插件统一处理。

### 特性

- **Exa + AnySearch 优先** —— 这是本 fork 的主要使用组合：Exa 负责高质量语义检索，AnySearch 作为轻量、免 key 的补充与 fallback；其它引擎保留为可选来源。
- **Multi Search 默认模式** —— `Multi Search` 可以直接作为普通 `web_search` 的默认模式，并发调用路由/优先级前 3 个启用引擎，按 URL 合并去重，跨引擎重复命中的结果优先。
- **全局引擎开关** —— 设置页可以逐个禁用引擎；禁用项会从 web_search、Auto、advanced_search、multi_search、测试工具和引擎选择器中统一排除。
- **全局 fallback 优先级** —— 用 ↑/↓ 调整 `fallbackOrder`。单引擎模式始终先尝试当前首选，失败后严格按全局顺序回退；禁用引擎保留位置但不会执行。
- **Auto 智能路由** —— 根据查询语言和时间过滤选择候选引擎；只有在你实际自定义过全局顺序后，Auto 才在路由分组和后续 fallback 中应用该顺序。
- **统一失败回退** —— 缺 key、401、限流、空结果或网络错误都不会立即终止搜索，而是继续尝试下一个启用引擎。
- **精简动态系统提示词** —— 只注入当前启用引擎与实际有效的回退信息；已禁用引擎不会再把整段说明塞进上下文。Bing/DDG 被禁用后，与它们相关的设置说明也不会注入。
- **网页设置 UI + `/free-search-engine`** —— 支持默认模式/引擎切换、引擎开关、优先级调整、API key、缓存、Safe Search、Bing market、中英文界面和弹出式命令切换。
- **高级与多源工具** —— `advanced_search` 支持相对/绝对时间过滤；`multi_search` 可按指定引擎做并发交叉验证；`platform_search` 覆盖 GitHub / V2EX / Bilibili / Reddit / HN / Stack Overflow / Wikipedia / npm。
- **搜索结果安全处理** —— 插件自有工具的网页文本放进 `<untrusted-web-content>` 边界，统一清洗 snippet，并阻止网页内容伪造边界影响 agent 指令。
- **缓存与诊断** —— LRU 查询缓存、短 TTL fallback 缓存、`free_search_test` 引擎测试，以及设置页当前引擎直测。
- **DSH 新版兼容策略** —— 仅设置最低 DSH 版本要求，不锁最高版本；设置项使用 profile-owned volatile config，并在修改后动态刷新提示词。

如果这个 fork 对你有用，可以给 [Soulize/dsh-free-search](https://github.com/Soulize/dsh-free-search) 点个 ⭐。

> 上游来源：[DDDMUC/dsh-free-search](https://github.com/DDDMUC/dsh-free-search)。感谢原作者提供最初的插件结构和多引擎实现基础。

### 引擎列表

| id | 引擎 | 费用 | 说明 |
|---|---|---|---|
| `auto` | Auto 智能路由 | 动态 | **根据查询语言/时间条件自动选路**（含中文优先 Bing/Baidu/Aliyun/AnySearch，英文优先 Bing/Exa/Tavily；时间过滤优先支持引擎），末尾全量回退 |
| `multi` | Multi Search | 动态 | **并发搜索路由/优先级前 3 个启用引擎**，URL 去重合并，跨引擎重复命中优先 |
| `ddg` | DuckDuckGo HTML | 免费 | 偶发限流（反爬），解封自动恢复 |
| `ddg-lite` | DuckDuckGo Lite | 免费 | 轻量版，同上 |
| `bing` | Bing | 免费 | **默认引擎**，最稳定，中文优化（zh-CN） |
| `anysearch` | AnySearch AI | 免费 | AI 搜索，无 key（匿名额度） |
| `searxng` | SearXNG 元搜索 | 免费 | 多实例自动切换，支持自定义实例 |
| `exa` | Exa | 免费 | **无 key 也可用**（MCP 匿名），配 key 提升额度 |
| `tavily` | Tavily | 免费 | **无 key 也可用**（keyless 匿名），配 key 提升额度 |
| `keenable` | Keenable | 免费 | **无 key 也可用**（MCP 匿名），配 key 提升额度 |
| `firecrawl` | Firecrawl | 免费 | **无 key 也可用**（官方免 key 匿名额度），配 key 提升限额 |
| `parallel` | Parallel | 免费 | **无 key 也可用**（官方 MCP 匿名额度），配 key 提升额度并支持精确时间过滤 |
| `perplexity` | Perplexity | 付费 | 需 `PERPLEXITY_API_KEY` |
| `serpbase` | SerpBase | 付费 | 需 `SERPBASE_API_KEY`（serpbase.dev，注册送 100 次免费额度） |
| `deepseek-official` | DeepSeek 官方 | 付费 | 需 `DEEPSEEK_API_KEY` |
| `you` | You.com | 付费 | 需 `YOUCOM_API_KEY`（you.com/platform/api-keys，注册即送免费额度） |
| `baidu` | 百度千帆 AI 搜索 | 额度 | 需 `BAIDU_API_KEY`（千帆 AI 搜索，每日赠 50 次，超额按量后付费）；中文全网，支持时间过滤 |
| `kimi` | Kimi（Moonshot）联网搜索 | 付费 | 需 `MOONSHOT_API_KEY`（basic 约 ￥0.01/次），返回带正文 chunks 的中文结果 |
| `aliyun` | 阿里云百炼 EnhancedSearch | 付费 | 需 `DASHSCOPE_API_KEY`（MCP search_pro，约 ￥0.03/次，新用户 200 次免费包）；中文全网带来源 hostname |

- **代码默认仍为 `bing`**，用于无配置开箱即用；本 fork 更推荐把 **`exa` 和 `anysearch`** 放在启用列表与 `fallbackOrder` 前部，或只保留这两个引擎配合 `Multi Search`。
- **自动回退**：按当前首选模式、全局 `fallbackOrder` 和禁用列表决定实际候选；任一引擎失败会继续尝试下一个启用引擎。`Multi Search` 则并发调用候选并合并结果。
- **设置页有官网链接**：免费引擎显示"访问官网 →"，付费引擎显示"获取 API Key →"（新标签页打开）：
  - Exa：<https://dashboard.exa.ai/api-keys>
  - Tavily：<https://app.tavily.com/home>
  - Keenable：<https://keenable.ai/login>
  - Parallel：<https://platform.parallel.ai>
  - Perplexity：<https://www.perplexity.ai/settings/api>
  - SerpBase：<https://serpbase.dev>
  - DeepSeek：<https://platform.deepseek.com/api_keys>
  - You.com：<https://you.com/platform/api-keys>

#### 为什么免费引擎不需要 key？

- **AnySearch**：其 `v1/search` REST 接口提供匿名的公共搜索额度，无需注册或 API key。额度有限流（适合日常搜索），但作为免费引擎之一，与其他免费引擎互相回退，体验稳定。
- **Exa**：公开 MCP 端点（`mcp.exa.ai/mcp`）支持匿名调用，不配 key 也能用；配置 `EXA_API_KEY` 后可获得更高额度。
- **Tavily**：通过 `x-tavily-access-mode: keyless` 头走 keyless 匿名额度，不配 key 即可用；配置 `TAVILY_API_KEY` 后走账号档，额度更高、结果质量更稳定。
- **Keenable**：无 key 时走其公开 MCP 端点（`api.keenable.ai/mcp`）匿名调用；配置 `KEENABLE_API_KEY` 后走 REST API（`api.keenable.ai/v1/search`），额度更高、按组织限流。
- **Firecrawl**：其 `/v2/search` 端点**无需 key** 即可使用（官方文档明确说明，有匿名限流）；配置 `FIRECRAWL_API_KEY` 后可提高限额。支持 `tbs` 时间过滤（`qdr:h/d/w/m/y` 与自定义日期区间）。

### 安装

推荐直接安装这个 fork 的当前 master：

```sh
dsh plugin --profile web add "github:Soulize/dsh-free-search#master"
```

如果之前已经装过旧提交，显式带 `#master` 可以避免继续命中旧解析/锁定版本。

本地开发也可以：

```sh
git clone https://github.com/Soulize/dsh-free-search.git
dsh plugin --profile web add /path/to/dsh-free-search
```

然后重启：

```sh
dsh web
```

#### 接管行为与验证

插件加载后**自动接管搜索**：

- `web.searchProvider` 未设置，或仍是 DSH 出厂默认的官方搜索 `deepseek-official` 时，自动切换为本插件；
- 如果（你或别的插件）已显式选择其他 provider，本插件不抢占，只在启动日志输出 WARN 与切换用的 YAML。

正常安装（`dsh plugin add` / 插件管理器）时，插件自带的 **bundle patch**（`cordis.patch.yml`）还会在配置层显式写入 `searchProvider: ddg`；即使这层没生效（例如把插件作为普通依赖手工安装、或 profile patch 整体覆盖了 `web` 条目），上面的运行时兜底也会接管官方默认，不会再静默退回官方搜索。

需要显式声明、或从其他 provider 切过来时（`profiles/<profile>/cordis.patch.yml`）：

```yaml
# 让 harness 的 web_search 使用本插件。
# DSH 0.1.2+ 的 patch 是"整段覆盖 config"而不是深度合并：
# 已有 web 条目里的字段（如 fetchProvider）必须在这里一并重述，否则会被抹掉。
- id: web
  config:
    searchProvider: ddg
    fetchProvider: http
```

- `searchProvider: ddg` —— `ddg` 是**本插件注册的 provider id（固定值）**，不是"使用 DuckDuckGo 引擎"的意思；具体用哪个引擎由设置页的 `provider` 字段决定（可填 `bing`/`baidu`/`auto` 等）。
- `fetchProvider: http` —— 官方网页抓取（web-fetch-http），请保留；漏掉会导致网页抓取失效或重复注册。
- 想改回官方搜索：在插件管理器里停用本插件条目即可。

#### 上游作者的相关插件：dsh-preset-workbench（预设工作台）

上游作者维护的相关插件：在设置页里可视化创建/编辑 Agent 预设——分段提示词、15 项能力开关、内置「鲸鱼娘 / 梁神模式」模板，不用手写 YAML。两者搭配：**free-search 解决"AI 联网搜索"、preset-workbench 解决"AI 人设能力编排"**，都是纯免费、开箱即用。

- 仓库：<https://github.com/DDDMUC/dsh-preset-workbench>
- 安装：`dsh plugin --profile web add github:DDDMUC/dsh-preset-workbench`
- 用法：设置 → 预设工作台

如果你觉得 preset-workbench 也有用，同样欢迎给它的仓库点个 ⭐。🙏

#### 依赖说明

> 此 fork 仅要求 DSH `>=0.1.7-rc.1`，不设置最高版本上限。

插件对 `@deepseek-ai/dsh-settings` 和 `@deepseek-ai/dsh-tools` 使用 `peerDependencies`，这是刻意的：DSH 运行时必须使用安装树中的唯一实例。请通过 `dsh plugin --profile <profile> add ...` 安装插件，不要把 DSH 核心包复制进 profile 的本地 `node_modules`；重复副本会导致工具调度器失效。

### 使用

#### 网页设置（推荐）

安装后打开配置页（DSH 0.1.7-rc.1+）：

- 左侧 **插件** 页 → **已安装** 分组 → `free-search` → 点击组件行 `web-search-free`（行内"配置"入口）

配置页提供：

- **Search engine**：下拉框切换引擎或模式，支持 `Auto` / `Multi Search` / 单引擎；保存即生效
- **Global fallback priority**：用 ↑/↓ 调整全局回退顺序；禁用引擎仍保留排序位置，重新启用后继续沿用
- **API keys**：为 Exa / Tavily / Keenable / Firecrawl / Parallel / Perplexity / DeepSeek / SerpBase / You.com 填写 key（密码框，保存后只显示"已配置"；Exa / Tavily / Keenable / Firecrawl / Parallel 不填也可免 key 使用）
  - **推荐**：付费引擎 key 建议写入 harness 凭据中心 `~/.dsh/.credentials.yaml`（如 `DEEPSEEK_API_KEY: sk-...`，与官方 LLM provider 一致，一处管理所有 key）。插件读取优先级：凭据中心 > 设置页 > 环境变量，设置页填的 key 仅作为遗留兼容。
- **Test engine**：直测当前引擎可用性（不走回退链，付费引擎无 key 会明确报错）
- **Use Bing default**：把当前搜索引擎切回稳定的免费 Bing；`Discard` 只撤销尚未保存的编辑
- **Platform search**：勾选启用 GitHub / V2EX / Bilibili 平台搜索（`platform_search` 工具按此过滤）
- **EN / 中文**：切换界面语言（默认中文）

<table align="center" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="center" width="50%" style="border: none; padding: 6px;">
      <a href="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free.png">
        <img src="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free.png" alt="免费引擎设置" width="100%" />
      </a>
      <br>
      <sub>▲ <b>免费引擎</b>（显示绿色 FREE 徽章与官网链接）</sub>
    </td>
    <td align="center" width="50%" style="border: none; padding: 6px;">
      <a href="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-apikey.png">
        <img src="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-apikey.png" alt="付费引擎设置" width="100%" />
      </a>
      <br>
      <sub>▲ <b>付费/API Key 引擎</b>（显示橙色 API KEY 徽章与获取链接）</sub>
    </td>
  </tr>
</table>

#### 聊天框切换引擎（/free-search-engine）

不用进设置页也能切换引擎：在聊天框输入 `/free-search-engine`，**弹出引擎选择窗口**（和 `/model` 选模型一样的交互），点选即切换，当前引擎会标记出来。等效于设置页切换 + 保存，且界面语言跟随设置页（中文/英文）。

命令只改首选引擎配置，搜索仍走 `web_search` + 统一回退链：即使首选引擎挂了也会自动换其他引擎，永不直接失败。系统提示词同步刷新。

#### 配置文件

DSH 0.1.7-rc.1 起，配置跟随 profile 的插件条目保存：设置页与 `/free-search-engine` 都会写入当前 profile 的 `cordis.patch.yml` 中 `web-search-free`（`dsh-free-search`）条目的 `config`。旧版 `~/.dsh/settings.yaml` 的 `free-search:` 段只会在启动时自动导入一次，随后原文件被重命名为 `settings.yaml.imported`。

```yaml
# profiles/<profile>/cordis.patch.yml 中该条目的 config：
provider: bing              # ddg / ddg-lite / bing / searxng / anysearch / exa / tavily / keenable / firecrawl / parallel / perplexity / serpbase / deepseek-official / you
disabledEngines: []       # 此 fork：全局禁用的引擎 id 列表
fallbackOrder:              # 此 fork：全局回退优先级（从上到下）
  - exa
  - tavily
  - bing
  - anysearch
  - ddg
  - ddg-lite
  - searxng
lang: zh                    # 设置页界面语言（zh / en）
bingMarket: zh-CN           # Bing 市场
region: cn-zh               # DuckDuckGo 区域（可选）
searxngInstances:           # 自定义 SearXNG 实例（可选）
  - https://your-instance.example
exaApiKey: ...              # 或通过设置页填写
tavilyApiKey: ...           # 或通过设置页填写
keenableApiKey: ...         # 或通过设置页填写
firecrawlApiKey: ...        # 或通过设置页填写
parallelApiKey: ...         # 或通过设置页填写
perplexityApiKey: ...
serpbaseApiKey: ...         # 或通过设置页填写
deepseekApiKey: ...
```

#### 让 agent 测试所有引擎

对 agent 说"测试一下所有搜索引擎"，它会调用 `free_search_test` 工具，逐个测试并报告：

```
Search engine test:
- ddg: FAIL - DuckDuckGo is rate-limited right now (anti-bot challenge, usually temporary) - Bing works
- bing: OK (2 results, e.g. "DeepSeek Harness developer preview...")
- exa: FAIL - EXA_API_KEY not configured
```

#### 时间过滤（advanced_search）

让 agent 搜"最近一周的新闻"、"这个月的发布"、"最近 3 天的消息"、"7 月以来的更新"，它会调用 `advanced_search` 工具，带 `timeRange` 参数。该工具同样走统一回退链，且可显式指定 `engine`，返回结构同 `web_search`。

**timeRange 支持三种形式：**

| 形式 | 示例 | 含义 |
|---|---|---|
| 固定档 | `day` / `week` / `month` / `year` | 分别 = 1 / 7 / 30 / 365 天 |
| 自定义相对值 | `12h`、`3d`、`2mo`、`1y` | 最近 12 小时 / 3 天 / 2 个月 / 1 年 |
| 绝对日期 | `2026-07-01` | 该日期（含）之后发布的结果 |

**各引擎对 timeRange 的处理逻辑：**

| 引擎 | 参数 | 是否精确 | 说明 |
|---|---|---|---|
| Exa | `startPublishedDate` | ✅ 精确 | 自定义天数转成 ISO 日期（N 天前），绝对日期原样传入 |
| Keenable | `published_after` | ✅ 精确 | 相对值原样传（`12h/3d/2mo/1y`），绝对日期原样传 |
| Tavily | `time_range` | ⚠️ 近似 | 只认固定档，自定义天数自动映射到最近似档位 |
| Firecrawl | `tbs` | ⚠️ 近似 | 固定档映射到 `qdr:d/w/m/y`；绝对日期用 `cdr:1,cd_min:M/D/YYYY`（精确） |
| Parallel | `source_policy.after_date`（有 key 时精确）；无 key 走 MCP，无日期参数，改为把窗口写进 objective 作为新鲜度提示（软过滤） | ✅ 精确 / ⚠️ 软过滤 | 自定义天数转成 ISO 日期（N 天前），绝对日期原样传入 |
| SearXNG | `time_range` | ⚠️ 近似 | 同上 |
| DuckDuckGo / Lite | `df` | ⚠️ 近似 | 同上 |
| Bing / AnySearch | — | ❌ 忽略 | 无对应参数 |

**"最近似档位"映射规则**：`≤2 天 → day`，`≤14 天 → week`，`≤90 天 → month`，否则 `year`。例如 `3d` 在 Tavily 上按 `day` 处理，`2mo` 按 `month` 处理。

**引擎链优先级**：当带 timeRange 搜索时，支持时间过滤的引擎（tavily / exa / keenable / firecrawl / parallel / searxng / ddg / ddg-lite）会排到引擎链前面，确保过滤真正生效——即使首选引擎是 bing（不支持过滤），也会先尝试支持过滤的引擎。

示例对话：*"帮我搜最近 3 天关于 DSH 的新闻"* → agent 调用 `advanced_search`，`timeRange: "3d"`。

#### 多源并发合并搜索（multi_search）

当需要对重要问题做**多源交叉验证**、避免单一引擎偏差或单源死锁时，可以让 agent 调用 `multi_search` 工具：

- **并发请求**：默认基于当前查询类型并发请求前 3 个优选引擎（或显式传入 `engines` 列表），各引擎独立解析 API Key 与容错（缺 key 引擎自动跳过，不阻断其他引擎）。
- **去重与合并**：按规范化 URL 去除结尾斜杠并合并结果，多引擎共同命中的条目优先置顶排在最前，并在结果附带 `seenIn` 命中来源清单（如 `[seen in: bing, exa]`）。
- **不可信边界与清洗**：严格遵守 `<untrusted-web-content>` 数据边界，正文 snippet 统一清洗。
- ⚠️ 注：多源并发会消耗更多 API 配额，建议在需要多角度核验时按需使用。

#### 抓取网页内容（web_fetch）

搜索到 URL 后，可以让 agent **读取网页全文**（如"打开第一个链接看看内容"）。`web_fetch` 工具已启用（官方 `dsh-web-fetch-http` provider）：

- 自动跟随重定向、解码正文（HTML 转文本）
- 支持超时和大小限制
- ⚠️ 注意：`web_fetch` 无 SSRF 防护，agent 理论上可访问内网地址——按需使用

#### 平台搜索（platform_search）

让 agent 搜特定平台，如"在 GitHub 上搜 deepseek harness"、"看看 B站有什么相关视频"、"V2EX 上关于 dsh 的讨论"。`platform_search` 工具支持：

| 平台 | 用途 |
|---|---|
| `github` | GitHub 仓库搜索（API，免费无 key） |
| `v2ex` | V2EX 热门/相关主题 |
| `bilibili` | B站视频/内容搜索（公开接口） |
| `reddit` | Reddit 帖子/讨论搜索（公开 JSON API；部分网络环境可能被 Reddit 反爬拦截） |
| `hn` | Hacker News 技术社区讨论（Algolia 官方 API） |
| `stackoverflow` | Stack Overflow 技术问答（Stack Exchange 官方公开 API） |
| `wikipedia` | 维基百科词条（中文环境用 zh.wikipedia.org，`lang: en` 时切换 en.wikipedia.org） |
| `npm` | npm 包搜索（registry 官方 API） |

全部走公开 API，零外部依赖、无需任何 key，开箱即用。

### 本地引擎切换工具（tools/）

`tools/` 目录附带了一个本地切换小工具（零依赖）：

- **`启动搜索引擎切换器.cmd`**（Windows）——双击启动本地 Node 服务（`http://127.0.0.1:4789`）并自动打开浏览器选择页面
- **`switch-engine.html`** —— 选择页面：显示当前引擎，点选新引擎，一键写入配置
- **`server.mjs`** —— 本地服务，负责读写 `~/.dsh/profiles/web/cordis.patch.yml`
- **`switch-engine.ps1`** —— 无界面命令行版：`powershell -File tools/switch-engine.ps1 -Engine bing`

切换后重启 `dsh web` 生效。

> 配置卡片挂在左侧「插件」页的 `plugins.row.config` 行配置插槽（dsh 自带），配置读写走插件自建 bridge，**不依赖 dsh-web-ui**，插件可独立使用。

### 代理说明（国内用户）

DuckDuckGo 等引擎可能需要代理才能访问，而 Node.js 的 `fetch` 默认不走系统代理。需要给 dsh 进程设置（Node 24+）：

```sh
export NODE_USE_ENV_PROXY=1
export HTTPS_PROXY=http://127.0.0.1:7897   # 你的代理地址
export HTTP_PROXY=http://127.0.0.1:7897
```

Windows 用户：桌面快捷方式已内置此配置（`set NODE_USE_ENV_PROXY=1&& set HTTPS_PROXY=...`）。

### 工作原理

- `lib/index.js`：host 端。实现 `WebSearchProvider`（`id` / `available()` / `search()`），统一引擎路由 + 自动回退（付费引擎优先，免费兜底）；解析 `timeRange`（固定档/相对值/绝对日期）并透传给各引擎；在 `web-search-free` 条目上声明可编辑配置（`.volatile()`）并自带设置页（`plugins.row.config`）；提供 `/api/dsh-free-search-settings` 读写桥 + `raw-search` 调试接口；注册 `free_search_test`、`platform_search`、`advanced_search` 工具；动态注入引擎清单到系统提示词（设置变更时自动刷新）。
- `lib/client.js`：浏览器端。React 配置卡片（引擎选择 + key 输入 + 连通测试 + 中英切换），挂载到左侧「插件」页的 `plugins.row.config` 行配置插槽；注册 `/free-search-engine` 弹出式切换命令（`commandUi` popupSelect，与 `/model` 同机制）。
- `cordis.patch.yml`：插件 loader 配置。

---

## English

<div align="center">
  <a href="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free1.png">
    <img src="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free1.png" alt="Free Engine Settings (Bing)" width="820" />
  </a>
  <br>
  <sub>▲ Free engine (using Bing as an example)</sub>
</div>

### Why You Need It

dsh's default search provider relies on the official DeepSeek API key (`DEEPSEEK_API_KEY`). If you:
- Do not have (or prefer not to use) an official DeepSeek key,
- Use a gateway like opencode-go (whose OpenAI-compatible endpoint does not support the `web_search` tool),

...then the built-in search will inevitably fail, and the agent will tell you "I cannot access the internet."

This fork is not primarily about adding the largest possible engine list. Its main workflow is **Exa + AnySearch**, with the remaining engines kept as optional fallbacks or cross-checking sources. Routing, concurrent merge, failover, and prompt-budget control are handled centrally by the plugin.

### Features

- **Exa + AnySearch focused** — The primary workflow in this fork. Exa is the high-quality semantic search path; AnySearch is a lightweight keyless companion and fallback. Other engines remain available as optional sources.
- **Selectable Multi Search default mode** — Set `Multi Search` as the normal `web_search` mode. It queries the top 3 enabled routed/prioritized engines concurrently, merges duplicate URLs, and prioritizes results confirmed by multiple engines.
- **Global engine enable/disable** — Disable individual engines once and they are excluded consistently from web_search fallback, Auto routing, advanced_search, multi_search, engine tests, and the engine picker.
- **Global fallback priority** — Reorder `fallbackOrder` with ↑/↓ controls. Single-engine mode always tries the selected engine first, then follows the configured order. Disabled engines retain their position but are skipped.
- **Auto routing** — Language/time-aware engine routing. The original Auto behavior stays unchanged until you actually customize the global priority; then that order is applied within routing groups and the remaining fallback chain.
- **Unified failover** — Missing keys, 401s, rate limits, empty results, and network errors continue to the next enabled engine instead of terminating search immediately.
- **Compact dynamic system prompt** — Only enabled engines and effective fallback information are injected. Disabled-engine catalog noise is removed from context, and Bing/DDG-specific lines disappear when those engines are disabled.
- **Web settings UI + `/free-search-engine`** — Configure modes/engines, global enable state, fallback priority, API keys, cache, Safe Search, Bing market, UI language, and the popup engine switcher.
- **Advanced and multi-source tools** — `advanced_search` adds relative/absolute time filtering; `multi_search` performs explicit concurrent cross-engine validation; `platform_search` covers GitHub / V2EX / Bilibili / Reddit / HN / Stack Overflow / Wikipedia / npm.
- **Search-output safety** — Plugin-owned search output is wrapped in `<untrusted-web-content>`, snippets are normalized, and look-alike boundary tags from webpages are stripped.
- **Caching and diagnostics** — LRU query cache, shorter TTLs for fallback hits, `free_search_test`, and direct testing of the selected engine from Settings.
- **Forward-compatible DSH versioning** — Only a minimum DSH version is enforced; profile-owned volatile settings refresh runtime behavior and the injected prompt dynamically.

If this fork is useful to you, consider starring [Soulize/dsh-free-search](https://github.com/Soulize/dsh-free-search).

> Upstream: [DDDMUC/dsh-free-search](https://github.com/DDDMUC/dsh-free-search). Thanks to the original author for the initial plugin structure and multi-engine foundation.

### Supported Engines

| id | Engine | Cost | Description |
|---|---|---|---|
| `auto` | Auto Smart Routing | Dynamic | **Smartly routes engines based on query language/time filter** (Chinese queries prioritize Bing/Baidu/Aliyun/AnySearch, English queries prioritize Bing/Exa/Tavily; time filters prioritize time-capable engines), with full fallback |
| `ddg` | DuckDuckGo HTML | Free | Occasional rate limits (anti-bot challenges); recovers automatically |
| `ddg-lite` | DuckDuckGo Lite | Free | Lightweight version; same rate-limit behavior as above |
| `bing` | Bing | Free | **Default engine**, most stable, optimized for Chinese (`zh-CN`) |
| `anysearch` | AnySearch AI | Free | AI search, no key needed (anonymous quota) |
| `searxng` | SearXNG Meta Search | Free | Multi-instance automatic failover; supports custom instances |
| `exa` | Exa | Free | **Usable without a key** (anonymous MCP); configure a key for higher quota |
| `tavily` | Tavily | Free | **Usable without a key** (keyless anonymous); configure a key for higher quota |
| `keenable` | Keenable | Free | **Usable without a key** (anonymous MCP); configure a key for higher quota |
| `firecrawl` | Firecrawl | Free | **Usable without a key** (official keyless anonymous quota); configure a key for higher limits |
| `parallel` | Parallel | Free | **Works without a key** (official MCP anonymous quota); a key raises limits and enables precise time filtering |
| `perplexity` | Perplexity | Paid | Requires `PERPLEXITY_API_KEY` |
| `serpbase` | SerpBase | Paid | Requires `SERPBASE_API_KEY` (serpbase.dev, 100 free queries on signup) |
| `deepseek-official` | DeepSeek Official | Paid | Requires `DEEPSEEK_API_KEY` |
| `you` | You.com | Paid | Requires `YOUCOM_API_KEY` (you.com/platform/api-keys, free tier on signup) |
| `baidu` | Baidu Qianfan AI Search | Quota | Requires `BAIDU_API_KEY` (Qianfan AI Search, 50 free calls/day then pay-as-you-go); Chinese web-wide, supports time filtering |
| `kimi` | Kimi (Moonshot) Web Search | Paid | Requires `MOONSHOT_API_KEY` (basic ~¥0.01/call); returns Chinese results with body chunks |
| `aliyun` | Aliyun Bailian EnhancedSearch | Paid | Requires `DASHSCOPE_API_KEY` (MCP search_pro, ~¥0.03/call, 200 free calls for new users); Chinese web-wide with source hostnames |

- **The code default remains `bing`** for zero-config startup, but this fork is primarily tuned around **`exa` + `anysearch`**. Put them near the front of the enabled list / `fallbackOrder`, or keep only those two for a lean Multi Search setup.
- **Auto-failover**: the effective candidate order comes from the selected mode, global `fallbackOrder`, and the disabled-engine list. A failed engine continues to the next enabled candidate; Multi Search queries candidates concurrently and merges the successful results.
- **Official Links in Settings**: Free engines display "Visit Website →", while paid engines display "Get API Key →" (opens in a new tab):
  - Exa: <https://dashboard.exa.ai/api-keys>
  - Tavily: <https://app.tavily.com/home>
  - Keenable: <https://keenable.ai/login>
  - Parallel: <https://platform.parallel.ai>
  - Perplexity: <https://www.perplexity.ai/settings/api>
  - SerpBase: <https://serpbase.dev>
  - DeepSeek: <https://platform.deepseek.com/api_keys>
  - You.com: <https://you.com/platform/api-keys>

#### Why are some engines free?

- **AnySearch**: its `v1/search` REST endpoint provides anonymous public search quota without registration or an API key. Quota is rate-limited (fine for daily queries), but as one of the free engines with mutual fallback it stays reliable.
- **Exa**: its public MCP endpoint (`mcp.exa.ai/mcp`) supports anonymous requests, so it works without a key; configuring `EXA_API_KEY` grants a higher usage quota.
- **Tavily**: offers keyless anonymous quota via the `x-tavily-access-mode: keyless` header — it works without a key; configuring `TAVILY_API_KEY` switches to the account tier for higher quota and more stable results.
- **Keenable**: without a key it is called via its public MCP endpoint (`api.keenable.ai/mcp`); configuring `KEENABLE_API_KEY` switches to the REST API (`api.keenable.ai/v1/search`) for higher quota and organization-scoped rate limits.
- **Firecrawl**: its `/v2/search` endpoint works **without a key** out of the box (the official docs state "No API key needed to get started", with anonymous rate limits); configuring `FIRECRAWL_API_KEY` raises the limits. Supports `tbs` time filtering (`qdr:h/d/w/m/y` and custom date ranges).

### Installation

Recommended: install the current master of this fork directly:

```sh
dsh plugin --profile web add "github:Soulize/dsh-free-search#master"
```

If an older commit is already installed, the explicit `#master` ref helps avoid reusing the previous locked resolution.

For local development:

```sh
git clone https://github.com/Soulize/dsh-free-search.git
dsh plugin --profile web add /path/to/dsh-free-search
```

Then restart:

```sh
dsh web
```

#### Takeover behavior and verification

The plugin **takes over search automatically** once loaded:

- When `web.searchProvider` is unset, or still the shipped default `deepseek-official`, it switches to this plugin;
- If you (or another plugin) explicitly selected a different provider, it does not steal it — it only logs a WARN with a copy-pasteable YAML snippet.

With a normal install (`dsh plugin add` / plugin manager) the bundled **bundle patch** (`cordis.patch.yml`) additionally writes `searchProvider: ddg` into the config layer. Even if that layer does not apply (e.g. the plugin was added as a plain dependency, or a profile patch replaced the whole `web` entry), the runtime fallback above still takes over from the official default — search no longer silently falls back to the official provider.

To declare it explicitly, or to switch over from another provider (`profiles/<profile>/cordis.patch.yml`):

```yaml
# Route the harness web_search through this plugin.
# Since DSH 0.1.2 a patch REPLACES the whole entry config (no deep merge):
# restate any existing web.* fields (e.g. fetchProvider) or they are dropped.
- id: web
  config:
    searchProvider: ddg
    fetchProvider: http
```

- `searchProvider: ddg` — `ddg` is this plugin's **fixed provider id**, not "use the DuckDuckGo engine". The engine is chosen by the `provider` field in the settings page (`bing` / `baidu` / `auto` / …).
- `fetchProvider: http` — the official web-fetch provider; keep it, or page fetching breaks or double-registers.
- To go back to official search: disable this plugin's entry in the plugin manager.

#### Related project from the upstream author: dsh-preset-workbench

A related plugin maintained by the upstream author: a **visual workbench for creating/editing agent presets** right inside Settings — sectioned prompts, 15 capability toggles, and built-in "Whale Girl / Liangshen Mode" templates, no YAML needed. Pair them up: **free-search gives your AI web search, preset-workbench shapes its persona & capabilities** — both free and zero-config.

- Repo: <https://github.com/DDDMUC/dsh-preset-workbench>
- Install: `dsh plugin --profile web add github:DDDMUC/dsh-preset-workbench`
- Usage: Settings → Preset Workbench

If preset-workbench is useful to you too, a ⭐ on its repo is always welcome. 🙏

#### Dependency Note

> This fork requires only DSH `>=0.1.7-rc.1`; no upper DSH version bound is enforced.

This plugin intentionally specifies `@deepseek-ai/dsh-settings` and `@deepseek-ai/dsh-tools` as `peerDependencies`: the DSH runtime must use a single instance from the installation tree. Always install the plugin using `dsh plugin --profile <profile> add ...`. Do **not** copy DSH core packages into a profile-local `node_modules`, as duplicate copies can break the tool scheduler.

### Usage

#### Web Settings (Recommended)

After installation, open the config page (DSH 0.1.7-rc.1+):

- Sidebar **Plugins** page → **Installed** group → `free-search` → click the `web-search-free` component row (the row's "configure" entry)

The config page provides:

- **Search engine**: Select an engine from the dropdown; changes take effect immediately upon saving.
- **API keys**: Enter keys for Exa / Tavily / Keenable / Firecrawl / Parallel / Perplexity / DeepSeek (password fields; displayed as "configured" once saved; Exa / Tavily / Keenable / Firecrawl / Parallel work without a key too).
  - **Recommended**: store paid-engine keys in the harness credential center `~/.dsh/.credentials.yaml` (e.g. `DEEPSEEK_API_KEY: sk-...`, same as the official LLM providers — one place for all keys). Resolution order: credentials center > settings page > environment variable; the settings-page fields remain for backward compatibility.
- **Test engine**: Tests the selected engine directly (no fallback chain; paid engines without a key report an explicit error).
- **Use Bing default**: stage a switch back to the stable free Bing engine; `Discard` only cancels unsaved edits
- **Platform search**: check platforms (GitHub / V2EX / Bilibili / Reddit / HN / Stack Overflow / Wikipedia / npm) to enable them for the `platform_search` tool (disabled platforms are skipped).
- **EN / 中文**: toggle the interface language (default Chinese).

<table align="center" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="center" width="50%" style="border: none; padding: 6px;">
      <a href="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free.png">
        <img src="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-free.png" alt="Free Engine Settings" width="100%" />
      </a>
      <br>
      <sub>▲ <b>Free Engine</b> (shows green FREE badge and official website link)</sub>
    </td>
    <td align="center" width="50%" style="border: none; padding: 6px;">
      <a href="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-apikey.png">
        <img src="https://raw.githubusercontent.com/Soulize/dsh-free-search/master/assets/settings-apikey.png" alt="Paid/API Key Engine Settings" width="100%" />
      </a>
      <br>
      <sub>▲ <b>Paid / API Key Engine</b> (shows orange API KEY badge and link to get an API key)</sub>
    </td>
  </tr>
</table>

#### Switching Engines from the Chat (/free-search-engine)

You can also switch the engine right from the chat — no need to open the settings page. Type `/free-search-engine`: a **picker opens with all engines** (the same interaction as `/model` for selecting a model). Click one to switch; the current engine is marked. Equivalent to switching and saving in the settings page, and the language follows the settings page (Chinese/English).

The command only changes the preferred engine; search still goes through `web_search` + the unified fallback chain — even if the preferred engine fails, it automatically switches to others, never failing outright. The system prompt refreshes accordingly.

#### Configuration File

Since DSH 0.1.7-rc.1 the configuration is stored with the profile's plugin entry: the settings page and `/free-search-engine` both write the `config` of the `web-search-free` (`dsh-free-search`) entry in the active profile's `cordis.patch.yml`. The old `free-search:` section of `~/.dsh/settings.yaml` is imported once at startup; the file is then renamed to `settings.yaml.imported`.

```yaml
# config of that entry in profiles/<profile>/cordis.patch.yml:
provider: bing              # ddg / ddg-lite / bing / searxng / anysearch / exa / tavily / keenable / firecrawl / parallel / perplexity / serpbase / deepseek-official / you
lang: zh                    # settings UI language (zh / en)
bingMarket: zh-CN           # Bing market
region: cn-zh               # DuckDuckGo region (optional)
searxngInstances:           # Custom SearXNG instances (optional)
  - https://your-instance.example
exaApiKey: ...              # Or configure via the web settings UI
tavilyApiKey: ...           # Or configure via the web settings UI
keenableApiKey: ...         # Or configure via the web settings UI
firecrawlApiKey: ...        # Or configure via the web settings UI
parallelApiKey: ...         # Or configure via the web settings UI
perplexityApiKey: ...
serpbaseApiKey: ...         # Or configure via the web settings UI
deepseekApiKey: ...
```

#### Asking the Agent to Test All Engines

Tell the agent *"Test all search engines"*, and it will call the `free_search_test` tool to check each engine sequentially and report back:

```
Search engine test:
- ddg: FAIL - DuckDuckGo is rate-limited right now (anti-bot challenge, usually temporary) - Bing works
- bing: OK (2 results, e.g. "DeepSeek Harness developer preview...")
- exa: FAIL - EXA_API_KEY not configured
```

#### Time Filtering (`advanced_search`)

Ask the agent for *"news from the last week"*, *"releases this month"*, *"updates from the last 3 days"*, or *"posts since July"*, and it will call the `advanced_search` tool with a `timeRange` parameter. It uses the same unified fallback chain, can force a specific `engine`, and returns the same shape as `web_search`.

**The `timeRange` parameter accepts three forms:**

| Form | Example | Meaning |
|---|---|---|
| Fixed tier | `day` / `week` / `month` / `year` | = 1 / 7 / 30 / 365 days |
| Custom relative | `12h`, `3d`, `2mo`, `1y` | last 12 hours / 3 days / 2 months / 1 year |
| Absolute date | `2026-07-01` | results published on or after that date |

**How each engine handles `timeRange`:**

| Engine | Parameter | Precise? | Notes |
|---|---|---|---|
| Exa | `startPublishedDate` | ✅ precise | custom days become an ISO date (N days ago); absolute dates pass through |
| Keenable | `published_after` | ✅ precise | relative values (`12h/3d/2mo/1y`) and absolute dates pass through |
| Tavily | `time_range` | ⚠️ approximate | only fixed tiers; custom days map to the nearest tier |
| Firecrawl | `tbs` | ⚠️ approximate | fixed tiers map to `qdr:d/w/m/y`; absolute dates use `cdr:1,cd_min:M/D/YYYY` (precise) |
| Parallel | `source_policy.after_date` with a key (precise); without a key the MCP path has no date parameter, so the window is written into the objective as a freshness hint (soft filter) | ✅ precise / ⚠️ soft | custom days become an ISO date (N days ago); absolute dates pass through |
| SearXNG | `time_range` | ⚠️ approximate | same as above |
| DuckDuckGo / Lite | `df` | ⚠️ approximate | same as above |
| Bing / AnySearch | — | ❌ ignored | no corresponding parameter |

**Nearest-tier mapping rule**: `≤2 days → day`, `≤14 days → week`, `≤90 days → month`, otherwise `year`. For example, `3d` becomes `day` on Tavily, and `2mo` becomes `month`.

**Engine-chain priority**: when a `timeRange` is present, engines that support time filtering (tavily / exa / keenable / firecrawl / parallel / searxng / ddg / ddg-lite) are moved to the front of the fallback chain, so the filter actually takes effect — even if the preferred engine is bing (which does not support filtering), a filtering-capable engine is tried first.

Example: *"Find DSH news from the last 3 days"* → agent calls `advanced_search` with `timeRange: "3d"`.

#### Multi-Engine Concurrent Search (`multi_search`)

When you need **cross-source verification** to prevent biases or fail-safes from a single engine, the agent can call `multi_search`:

- **Concurrent Execution**: Defaults to querying the top 3 recommended engines for the query type (or an explicit list passed via `engines`). Each engine independently resolves API keys and handles errors (missing keys are skipped without failing the batch).
- **Deduplication & Merge**: Normalizes URLs and ranks items by the number of engines that found them (`seenIn` counts), with cross-hit results placed at the top.
- **Untrusted Boundary**: Enforces the `<untrusted-web-content>` safety wrapper and trims snippets.
- ⚠️ Note: Running multiple engines in parallel consumes more search quota; use on demand when high source diversity is needed.

#### Fetch Webpage Content (`web_fetch`)

After searching, the agent can **read full webpage content** (e.g., *"Open the first link and summarize it"*). The `web_fetch` tool is enabled by default (official `dsh-web-fetch-http` provider):

- Automatically follows redirects and decodes HTML to plain text.
- Supports timeout and response size limits.
- ⚠️ Note: `web_fetch` does not have SSRF protection; the agent could theoretically access internal network addresses. Use as needed.

#### Platform Search (`platform_search`)

Ask the agent to search specific platforms (e.g., *"Search GitHub for deepseek harness"*, *"Find related videos on Bilibili"*, or *"Discussions about dsh on V2EX"*). The `platform_search` tool supports:

| Platform | Purpose |
|---|---|
| `github` | GitHub repository search (public API, free, no key required) |
| `v2ex` | V2EX hot / relevant topics |
| `bilibili` | Bilibili video / content search (public API) |
| `reddit` | Reddit posts / discussions (public JSON API; may be blocked by Reddit anti-bot in some network environments) |
| `hn` | Hacker News tech community discussions (official Algolia API) |
| `stackoverflow` | Stack Overflow Q&A (official public Stack Exchange API) |
| `wikipedia` | Wikipedia articles (zh.wikipedia.org for Chinese; switches to en.wikipedia.org when `lang: en`) |
| `npm` | npm package search (registry official API) |

All platform searches rely on public endpoints with zero external dependencies and no API keys — they work out of the box.

### Local Engine Switcher (`tools/`)

The `tools/` directory includes a lightweight, zero-dependency switcher:

- **`启动搜索引擎切换器.cmd`** (Windows) — Double-click to launch a local Node server (`http://127.0.0.1:4789`) and automatically open the engine selector page in your browser.
- **`switch-engine.html`** — The selector UI: displays current engine status and allows one-click switching.
- **`server.mjs`** — The local backend service responsible for reading/writing `~/.dsh/profiles/web/cordis.patch.yml`.
- **`switch-engine.ps1`** — Headless PowerShell script: `powershell -File tools/switch-engine.ps1 -Engine bing`.

Restart `dsh web` after switching to apply changes.

> The settings card mounts into the official `plugins.row.config` component-row slot on the sidebar Plugins page (built into DSH), and configuration reads/writes go through the plugin's own bridge. **No `dsh-web-ui` dependency — the plugin can be used standalone.**

### Proxy Note (for Users in Mainland China)

Engines like DuckDuckGo may require a proxy. Since Node.js `fetch` does not use the system proxy by default, set the following environment variables for the dsh process (Node 24+):

```sh
export NODE_USE_ENV_PROXY=1
export HTTPS_PROXY=http://127.0.0.1:7897   # Your proxy address
export HTTP_PROXY=http://127.0.0.1:7897
```

Windows users: The desktop shortcut already includes this configuration (`set NODE_USE_ENV_PROXY=1&& set HTTPS_PROXY=...`).

### How It Works

- `lib/index.js`: Host side. Implements `WebSearchProvider` (`id` / `available()` / `search()`), unified engine routing + auto-fallback (paid engines first, free as fallback); parses `timeRange` (fixed tiers / relative values / absolute dates) and forwards it to each engine; declares its editable config as volatile fields on the `web-search-free` composition entry and ships its own settings page (`plugins.row.config`); provides the `/api/dsh-free-search-settings` read/write bridge + `raw-search` debug endpoint; registers the `free_search_test`, `platform_search`, and `advanced_search` tools; dynamically injects the engine list into system prompts (auto-refreshes on settings change).
- `lib/client.js`: Browser side. React configuration card (engine select, key inputs, connectivity test, and Chinese/English toggle), mounted into the official `plugins.row.config` component-row slot on the sidebar Plugins page; registers the `/free-search-engine` popup switch command (`commandUi` popupSelect, the same mechanism as `/model`).
- `cordis.patch.yml`: Plugin loader configuration.

### License

MIT

## safeSearch 安全搜索过滤

- 全新配置项 `safeSearch`：`off`（引擎默认，不加参数）/ `moderate` / `strict`
- 作用于 Bing（adlt）、DuckDuckGo HTML（adlt）、DuckDuckGo Lite（adlt）
- 默认 `off`：不额外过滤，保持引擎自身默认行为；需要时在「设置 > 插件 > Free Search」切换
