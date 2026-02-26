# 5分钟用 DeepSeek API 搭建你自己的 AI 助手（Python 版）

> 不用 ChatGPT Plus，用国产 DeepSeek API，几行代码就能搭一个自己的 AI 助手。

## 为什么选 DeepSeek？

- **便宜**：API 价格是 GPT-4 的 1/10
- **中文强**：中文理解和生成能力一流
- **无需翻墙**：国内直连，延迟低
- **兼容 OpenAI 格式**：现有代码几乎零改动

## 快速开始

### 1. 获取 API Key

去 [SiliconFlow](https://siliconflow.cn) 注册，免费送额度。获取 API Key。

### 2. 安装依赖

```bash
pip install openai
```

### 3. 最简代码（5行搞定）

```python
from openai import OpenAI

client = OpenAI(
    api_key="你的API Key",
    base_url="https://api.siliconflow.cn/v1"
)

response = client.chat.completions.create(
    model="deepseek-ai/DeepSeek-V3",
    messages=[{"role": "user", "content": "用Python写一个快速排序"}]
)

print(response.choices[0].message.content)
```

就这么简单。

## 进阶：做一个命令行聊天助手

```python
from openai import OpenAI

client = OpenAI(
    api_key="你的API Key",
    base_url="https://api.siliconflow.cn/v1"
)

def chat():
    messages = [
        {"role": "system", "content": "你是一个专业的编程助手，回答简洁有用。"}
    ]
    
    print("AI 助手已启动（输入 quit 退出）")
    print("-" * 40)
    
    while True:
        user_input = input("\n你: ").strip()
        if user_input.lower() in ("quit", "exit", "q"):
            print("再见！")
            break
        if not user_input:
            continue
        
        messages.append({"role": "user", "content": user_input})
        
        # 流式输出
        print("\nAI: ", end="", flush=True)
        stream = client.chat.completions.create(
            model="deepseek-ai/DeepSeek-V3",
            messages=messages,
            stream=True
        )
        
        full_response = ""
        for chunk in stream:
            content = chunk.choices[0].delta.content or ""
            print(content, end="", flush=True)
            full_response += content
        
        print()  # 换行
        messages.append({"role": "assistant", "content": full_response})

if __name__ == "__main__":
    chat()
```

运行效果：

```
AI 助手已启动（输入 quit 退出）
----------------------------------------

你: Python 列表去重有几种方法？

AI: 常用的有这几种：

1. **set 去重**（最快，不保序）
   list(set(arr))

2. **dict.fromkeys**（保序）
   list(dict.fromkeys(arr))

3. **列表推导**（保序，可自定义）
   seen = set()
   [x for x in arr if x not in seen and not seen.add(x)]

推荐用第2种，简洁且保持原始顺序。
```

## 实战：做一个代码审查助手

```python
import sys
from openai import OpenAI

client = OpenAI(
    api_key="你的API Key",
    base_url="https://api.siliconflow.cn/v1"
)

def review_code(filepath):
    """自动审查代码文件"""
    with open(filepath) as f:
        code = f.read()
    
    response = client.chat.completions.create(
        model="deepseek-ai/DeepSeek-V3",
        messages=[
            {"role": "system", "content": """你是一个资深代码审查员。请审查以下代码，指出：
1. Bug 和潜在问题
2. 性能优化建议
3. 代码风格改进
4. 安全隐患
用中文回答，给出具体的修改建议。"""},
            {"role": "user", "content": f"请审查这段代码：\n\n```\n{code}\n```"}
        ]
    )
    
    print(f"=== {filepath} 代码审查报告 ===\n")
    print(response.choices[0].message.content)

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("用法: python review.py <文件路径>")
    else:
        review_code(sys.argv[1])
```

## 费用估算

| 用途 | 每天调用 | 月费用 |
|------|---------|--------|
| 个人聊天 | 50次 | ≈ ¥3 |
| 代码审查 | 20次 | ≈ ¥5 |
| 批量处理 | 200次 | ≈ ¥15 |

基本上个人使用每月不超过 ¥20。

## 总结

DeepSeek + SiliconFlow 是目前国内最具性价比的 AI API 方案。5 行代码就能用上，中文效果不输 GPT-4。

下一篇教你用这个 API 做一个微信聊天机器人，自动回复消息。

---

*点赞收藏，关注获取更多 AI 实战教程 👍*
