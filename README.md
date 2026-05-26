# Gemini 分流规则

这是一组面向 Google Gemini 的代理分流规则，覆盖 Gemini 网页版、Google AI Studio、Gemini Developer API 和 Vertex AI Gemini API。规则按“核心版”和“完整版”拆分，方便按使用场景选择。

## 文件说明

| 文件 | 适用客户端 | 说明 |
| --- | --- | --- |
| `rules/gemini-core.yaml` | Clash / Mihomo rule-provider | 核心规则，推荐默认使用 |
| `rules/gemini-full.yaml` | Clash / Mihomo rule-provider | 包含 Google 官方防火墙文档列出的 Gemini 辅助域 |
| `rules/gemini-core.list` | Surge / Shadowrocket / Stash | 核心规则 |
| `rules/gemini-full.list` | Surge / Shadowrocket / Stash | 完整规则 |
| `rules/gemini-sing-box-core.json` | sing-box rule_set | 核心规则 |
| `rules/gemini-sing-box-full.json` | sing-box rule_set | 完整规则 |
| `clients/*.example.*` | 各客户端示例 | 复制后替换策略组名和 GitHub raw URL |

## 规则选择

推荐先用 `core`：

- Gemini Web：`gemini.google.com`
- Google AI Studio：`aistudio.google.com`、`ai.google.dev`
- Gemini Developer API：`generativelanguage.googleapis.com`
- Vertex AI Gemini API：`aiplatform.googleapis.com` 及区域端点，如 `us-central1-aiplatform.googleapis.com`

如果 Gemini 网页端出现登录、图片、地图、YouTube 引用、静态资源或 Workspace 集成功能异常，再切换到 `full`。完整版包含 Google Workspace 官方 Gemini 防火墙页面列出的辅助域名，但会额外分流一部分 Google 资源。

## Clash / Mihomo 用法

把下面片段合并到配置中：

```yaml
rule-providers:
  gemini:
    type: http
    behavior: classical
    url: https://raw.githubusercontent.com/fancha0/Proxy-rules/main/rules/gemini-core.yaml
    path: ./ruleset/gemini.yaml
    interval: 86400

rules:
  - RULE-SET,gemini,Gemini
```

`Gemini` 是你的代理策略组名称，可以改成 `Proxy`、`AI` 或其他已有策略组。

## Surge / Shadowrocket / Stash 用法

Surge 示例：

```ini
RULE-SET,https://raw.githubusercontent.com/fancha0/Proxy-rules/main/rules/gemini-core.list,Gemini
```

Shadowrocket / Stash 可以直接导入 `.list` 文件，策略名同样按你的配置替换。

## sing-box 用法

sing-box 示例：

```json
{
  "route": {
    "rule_set": [
      {
        "tag": "gemini",
        "type": "remote",
        "format": "source",
        "url": "https://raw.githubusercontent.com/fancha0/Proxy-rules/main/rules/gemini-sing-box-core.json",
        "download_detour": "direct"
      }
    ],
    "rules": [
      {
        "rule_set": "gemini",
        "outbound": "proxy"
      }
    ]
  }
}
```

## 维护原则

- 不使用 `DOMAIN-SUFFIX,google.com` 或 `DOMAIN-SUFFIX,googleapis.com` 这类过宽规则，避免把整个 Google 生态都走代理。
- API 规则优先保守覆盖官方端点；网页端规则按实际依赖拆为核心和辅助。
- 如果客户端日志里发现新的 Gemini 专用域名，优先加到核心版；如果只是登录、静态资源或其它 Google 通用能力，放到完整版。

## 参考来源

- Google AI Studio 官方入口：<https://ai.google.dev/aistudio>
- Vertex AI GenAI REST 文档，服务端点为 `aiplatform.googleapis.com`：<https://cloud.google.com/vertex-ai/generative-ai/docs/reference/rest>
- Vertex AI Gemini global / regional endpoint 说明：<https://cloud.google.com/vertex-ai/generative-ai/docs/learn/locations>
- Gemini app 官方防火墙设置，列出 Gemini Web 相关辅助主机：<https://knowledge.workspace.google.com/admin/gemini/gemini-app-firewall-settings>

最后核对日期：2026-05-23。
