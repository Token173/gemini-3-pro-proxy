# Gemini 3 Pro 中转 API：国内直连谷歌 Gemini 3 Pro，长上下文与多模态一起用

Gemini 3 Pro 是 Google 这一代里质量最靠前的一档：长上下文、多模态理解、复杂推理都在线。问题是国内调不动——官方端点需要海外网络，Google Cloud 项目和结算又得绑境外卡。

Token173（https://token173.net）把 Gemini 3 Pro 放在国内节点上，OpenAI 兼容格式，改 `base_url` 就能跑。Gemini 3.8 Flash 也在同一个 Key 里，快慢两档随你切。

## 模型

| 模型 ID | 说明 |
|---|---|
| `gemini-3-pro-preview` | 质量优先，长文档、多图分析、复杂推理 |
| `gemini-3.8-flash` | 速度优先，实时对话、批量任务 |
| `gemini-4-carbon` | 需要长任务或代码审计时上探，[单独一篇](https://github.com/Token173/gemini-4-carbon) |

## 接入

```bash
curl https://token173.net/v1/chat/completions \
  -H "Authorization: Bearer sk-你的令牌" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-3-pro-preview",
    "messages": [{"role": "user", "content": "分析这份财报的三个风险点"}]
  }'
```

Python：

```python
from openai import OpenAI

client = OpenAI(api_key="sk-你的令牌", base_url="https://token173.net/v1")

r = client.chat.completions.create(
    model="gemini-3-pro-preview",
    messages=[{"role": "user", "content": "这两份合同的关键差异在哪？"}],
    max_tokens=4096,
)
print(r.choices[0].message.content)
```

## 传图做多模态

```python
import base64

with open("chart.png", "rb") as f:
    b64 = base64.b64encode(f.read()).decode()

r = client.chat.completions.create(
    model="gemini-3-pro-preview",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "这张图里的趋势说明了什么？"},
            {"type": "image_url", "image_url": {"url": f"data:image/png;base64,{b64}"}},
        ],
    }],
)
```

图片大的话先把边长压到 1024 左右，token 消耗能降一半，识别效果差别不大。

## 什么时候值得上 Pro

| 场景 | 是否用 Pro |
|---|---|
| 一份 50 页 PDF 要出结构化摘要 | 用，Pro 的长上下文和结构理解更稳 |
| 每秒几十条消息的客服机器人 | 不用，Flash 够了 |
| 表格截图转成 Excel | 用，多模态精度差距明显 |
| 日常代码问答 | 先 Flash，答不好再换 Pro |

Pro 的单价是 Flash 的好几倍，别默认全上 Pro。

## 客户端

| 工具 | 配置 |
|---|---|
| Cherry Studio | 供应商 Gemini，地址 `https://token173.net`，模型 `gemini-3-pro-preview` |
| Gemini CLI | `GOOGLE_GEMINI_BASE_URL=https://token173.net` |
| Cursor | 用 OpenAI 兼容模式，Base URL `https://token173.net/v1` |
| Dify | 模型供应商选 OpenAI 兼容，填域名和 Key |

## 计费

2 元 = 1 美元额度，Gemini 3 Pro 与 Flash 共用余额，注册送 0.2 美元。后台日志能看到每次请求的输入输出 token 与实际扣费，试算成本很方便。

## 常见问题

**Q：Gemini 3 Pro 和 3.8 Flash 的差别有多大？**
复杂推理和长文档上 Pro 明显更稳；短问答、分类抽取这类任务两者差距不大，Flash 快很多。

**Q：支持流式吗？**
支持，`stream: true` 按 SSE 返回。

**Q：一次能塞多长的内容？**
跟随模型上限，整份长文档可以一次进。但 token 会烧得很快，能切块就切块。

**Q：调用报 429？**
并发打满了。降并发或错峰跑，批量任务加个 sleep 更稳。

**Q：图片能不能直接传 URL？**
可以，`image_url` 填可公网访问的地址；内网图片先转 base64。

## 相关

- 文档站：https://docs.token173.net
- Gemini 全系：[gemini-api](https://github.com/Token173/gemini-api)
- Claude 专题：[claude-api](https://github.com/Token173/claude-api)
- 图像生成：[Nano-Banana-api](https://github.com/Token173/Nano-Banana-api)
