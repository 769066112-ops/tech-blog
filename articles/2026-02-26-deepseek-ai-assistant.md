---
title: 手把手教你用DeepSeek API搭建私有AI助手（附完整代码）
tags: DeepSeek, AI, API, Python, 教程
category: AI
description: 使用DeepSeek API从零搭建一个私有AI助手，支持多轮对话、流式输出、上下文记忆
---

# 手把手教你用DeepSeek API搭建私有AI助手（附完整代码）

## 前言

DeepSeek 是目前性价比最高的国产大模型之一，API价格只有GPT-4的1/10，效果却不输太多。

本文教你用 DeepSeek API 搭建一个功能完整的私有AI助手。

## 准备工作

1. 注册 DeepSeek 账号：https://platform.deepseek.com
2. 获取 API Key
3. 安装依赖：`pip install openai rich`

DeepSeek API 兼容 OpenAI 格式，所以直接用 openai 库就行。

## 1. 最简单的调用

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://api.deepseek.com/v1"
)

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": "你是一个有用的AI助手"},
        {"role": "user", "content": "用Python写一个快速排序"}
    ]
)

print(response.choices[0].message.content)
```

## 2. 流式输出（打字机效果）

```python
def stream_chat(messages):
    stream = client.chat.completions.create(
        model="deepseek-chat",
        messages=messages,
        stream=True
    )
    
    full_response = ""
    for chunk in stream:
        if chunk.choices[0].delta.content:
            content = chunk.choices[0].delta.content
            print(content, end="", flush=True)
            full_response += content
    
    print()  # 换行
    return full_response
```

## 3. 多轮对话（带上下文记忆）

```python
class ChatBot:
    def __init__(self, system_prompt="你是一个有用的AI助手"):
        self.client = OpenAI(
            api_key="your-api-key",
            base_url="https://api.deepseek.com/v1"
        )
        self.messages = [
            {"role": "system", "content": system_prompt}
        ]
        self.max_history = 20  # 最多保留20轮对话
    
    def chat(self, user_input):
        self.messages.append({"role": "user", "content": user_input})
        
        # 控制上下文长度
        if len(self.messages) > self.max_history * 2 + 1:
            self.messages = [self.messages[0]] + self.messages[-(self.max_history * 2):]
        
        response = self.client.chat.completions.create(
            model="deepseek-chat",
            messages=self.messages,
            stream=True
        )
        
        full_response = ""
        for chunk in response:
            if chunk.choices[0].delta.content:
                content = chunk.choices[0].delta.content
                print(content, end="", flush=True)
                full_response += content
        
        print()
        self.messages.append({"role": "assistant", "content": full_response})
        return full_response
    
    def clear(self):
        self.messages = [self.messages[0]]
        print("对话已清空")

# 使用
bot = ChatBot("你是一个Python专家，回答简洁有代码示例")
bot.chat("怎么读取Excel文件？")
bot.chat("如果文件很大怎么优化？")  # 它记得上一轮的上下文
```

## 4. 完整的终端AI助手

```python
from openai import OpenAI
from rich.console import Console
from rich.markdown import Markdown
import json
from pathlib import Path

console = Console()

class AIAssistant:
    def __init__(self):
        self.client = OpenAI(
            api_key="your-api-key",
            base_url="https://api.deepseek.com/v1"
        )
        self.history_file = Path("chat_history.json")
        self.messages = self._load_history()
        
        if not self.messages:
            self.messages = [{
                "role": "system",
                "content": "你是一个全能AI助手。回答准确、简洁、有条理。代码要有注释。"
            }]
    
    def _load_history(self):
        if self.history_file.exists():
            return json.loads(self.history_file.read_text())
        return []
    
    def _save_history(self):
        self.history_file.write_text(
            json.dumps(self.messages, ensure_ascii=False, indent=2)
        )
    
    def chat(self, user_input):
        self.messages.append({"role": "user", "content": user_input})
        
        try:
            response = self.client.chat.completions.create(
                model="deepseek-chat",
                messages=self.messages,
                temperature=0.7,
                max_tokens=4096,
                stream=True
            )
            
            full_response = ""
            for chunk in response:
                if chunk.choices[0].delta.content:
                    full_response += chunk.choices[0].delta.content
            
            self.messages.append({"role": "assistant", "content": full_response})
            self._save_history()
            
            # 用Rich渲染Markdown
            console.print(Markdown(full_response))
            return full_response
            
        except Exception as e:
            console.print(f"[red]错误: {e}[/red]")
            self.messages.pop()  # 移除失败的用户消息
            return None
    
    def run(self):
        console.print("[bold green]AI助手已启动[/bold green] (输入 /quit 退出, /clear 清空)")
        
        while True:
            try:
                user_input = console.input("\n[bold blue]你:[/bold blue] ").strip()
                
                if not user_input:
                    continue
                if user_input == "/quit":
                    console.print("[yellow]再见！[/yellow]")
                    break
                if user_input == "/clear":
                    self.messages = [self.messages[0]]
                    self._save_history()
                    console.print("[yellow]对话已清空[/yellow]")
                    continue
                
                console.print("\n[bold green]AI:[/bold green]")
                self.chat(user_input)
                
            except KeyboardInterrupt:
                console.print("\n[yellow]再见！[/yellow]")
                break

if __name__ == "__main__":
    assistant = AIAssistant()
    assistant.run()
```

## 5. 进阶：Function Calling

```python
import json

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "城市名"}
                },
                "required": ["city"]
            }
        }
    }
]

def get_weather(city):
    # 实际项目中调用天气API
    return f"{city}今天晴，25°C"

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[{"role": "user", "content": "北京今天天气怎么样？"}],
    tools=tools
)

# 处理函数调用
if response.choices[0].message.tool_calls:
    call = response.choices[0].message.tool_calls[0]
    args = json.loads(call.function.arguments)
    result = get_weather(args["city"])
    print(result)
```

## 费用参考

| 模型 | 输入价格 | 输出价格 |
|------|----------|----------|
| deepseek-chat | ¥1/百万token | ¥2/百万token |
| GPT-4o | ¥75/百万token | ¥150/百万token |

DeepSeek 便宜了 **75倍**，日常使用一个月也就几块钱。

## 总结

用 DeepSeek API 搭建私有AI助手，成本低、效果好、完全可控。代码已经给你了，复制就能跑。

---

> 完整代码已上传 GitHub，点赞收藏不迷路。
