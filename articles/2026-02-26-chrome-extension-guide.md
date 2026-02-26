---
title: Chrome扩展开发入门：从零到上架Chrome商店（Manifest V3）
tags: Chrome扩展, JavaScript, 前端, 浏览器插件, 教程
category: 前端
description: 2026年最新Chrome扩展开发教程，基于Manifest V3，从零开始到上架商店全流程
---

# Chrome扩展开发入门：从零到上架Chrome商店（Manifest V3）

## 前言

Chrome扩展是前端开发者最容易变现的技能之一。一个好用的扩展，上架Chrome商店后可以持续产生收入。

本文基于最新的 Manifest V3 规范，手把手教你开发一个完整的Chrome扩展。

## 1. 项目结构

```
my-extension/
├── manifest.json      # 扩展配置文件（核心）
├── popup/
│   ├── popup.html     # 弹窗页面
│   ├── popup.css      # 弹窗样式
│   └── popup.js       # 弹窗逻辑
├── background.js      # 后台服务（Service Worker）
├── content.js         # 内容脚本（注入到网页）
└── icons/
    ├── icon16.png
    ├── icon48.png
    └── icon128.png
```

## 2. manifest.json — 扩展的身份证

```json
{
  "manifest_version": 3,
  "name": "我的第一个扩展",
  "version": "1.0.0",
  "description": "一个实用的Chrome扩展",
  "permissions": ["storage", "activeTab", "scripting"],
  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "css": ["content.css"]
    }
  ]
}
```

**关键字段说明：**
- `manifest_version: 3` — 必须是3，V2已废弃
- `permissions` — 扩展需要的权限
- `action` — 工具栏图标和弹窗
- `background` — V3用Service Worker替代了background page
- `content_scripts` — 注入到网页的脚本

## 3. 实战：做一个网页字数统计扩展

### popup.html

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body {
      width: 300px;
      padding: 16px;
      font-family: -apple-system, BlinkMacSystemFont, sans-serif;
    }
    h2 { margin: 0 0 12px; color: #333; }
    .stat {
      display: flex;
      justify-content: space-between;
      padding: 8px 0;
      border-bottom: 1px solid #eee;
    }
    .stat-value {
      font-weight: bold;
      color: #1a73e8;
    }
    button {
      width: 100%;
      margin-top: 12px;
      padding: 8px;
      background: #1a73e8;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }
    button:hover { background: #1557b0; }
  </style>
</head>
<body>
  <h2>📊 页面统计</h2>
  <div id="stats">
    <div class="stat">
      <span>字数</span>
      <span class="stat-value" id="wordCount">-</span>
    </div>
    <div class="stat">
      <span>字符数</span>
      <span class="stat-value" id="charCount">-</span>
    </div>
    <div class="stat">
      <span>段落数</span>
      <span class="stat-value" id="paraCount">-</span>
    </div>
    <div class="stat">
      <span>预计阅读时间</span>
      <span class="stat-value" id="readTime">-</span>
    </div>
  </div>
  <button id="copyBtn">复制统计结果</button>
  <script src="popup.js"></script>
</body>
</html>
```

### popup.js

```javascript
// 获取当前标签页的统计数据
async function getPageStats() {
  const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
  
  const results = await chrome.scripting.executeScript({
    target: { tabId: tab.id },
    func: () => {
      const text = document.body.innerText || '';
      const chars = text.length;
      // 中文按字计算，英文按空格分词
      const chineseChars = (text.match(/[\u4e00-\u9fff]/g) || []).length;
      const englishWords = text.replace(/[\u4e00-\u9fff]/g, '').split(/\s+/).filter(w => w).length;
      const words = chineseChars + englishWords;
      const paragraphs = text.split(/\n\s*\n/).filter(p => p.trim()).length;
      // 中文300字/分钟，英文200词/分钟
      const readMinutes = Math.ceil(chineseChars / 300 + englishWords / 200);
      
      return { words, chars, paragraphs, readMinutes };
    }
  });
  
  return results[0].result;
}

// 更新UI
async function updateStats() {
  try {
    const stats = await getPageStats();
    document.getElementById('wordCount').textContent = stats.words.toLocaleString();
    document.getElementById('charCount').textContent = stats.chars.toLocaleString();
    document.getElementById('paraCount').textContent = stats.paragraphs;
    document.getElementById('readTime').textContent = `${stats.readMinutes} 分钟`;
    
    // 保存到storage
    await chrome.storage.local.set({ lastStats: stats });
  } catch (e) {
    document.getElementById('wordCount').textContent = '无法统计';
  }
}

// 复制结果
document.getElementById('copyBtn').addEventListener('click', async () => {
  const stats = await getPageStats();
  const text = `字数: ${stats.words} | 字符: ${stats.chars} | 段落: ${stats.paragraphs} | 阅读时间: ${stats.readMinutes}分钟`;
  await navigator.clipboard.writeText(text);
  document.getElementById('copyBtn').textContent = '✅ 已复制';
  setTimeout(() => {
    document.getElementById('copyBtn').textContent = '复制统计结果';
  }, 2000);
});

updateStats();
```

### background.js

```javascript
// 监听扩展安装
chrome.runtime.onInstalled.addListener(() => {
  console.log('扩展已安装');
});

// 快捷键支持
chrome.commands?.onCommand?.addListener((command) => {
  if (command === 'count-words') {
    // 触发字数统计
    chrome.action.openPopup();
  }
});
```

## 4. 调试和加载

1. 打开 `chrome://extensions/`
2. 开启「开发者模式」
3. 点击「加载已解压的扩展程序」
4. 选择你的项目文件夹

修改代码后点击刷新按钮即可更新。

## 5. 常用API速查

```javascript
// 存储数据
await chrome.storage.local.set({ key: 'value' });
const data = await chrome.storage.local.get('key');

// 发送消息（popup ↔ background ↔ content）
chrome.runtime.sendMessage({ type: 'getData' });
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'getData') {
    sendResponse({ data: 'hello' });
  }
});

// 操作标签页
const tabs = await chrome.tabs.query({ active: true });
await chrome.tabs.create({ url: 'https://example.com' });

// 注入脚本
await chrome.scripting.executeScript({
  target: { tabId: tab.id },
  func: () => document.title
});

// 右键菜单
chrome.contextMenus.create({
  id: 'myMenu',
  title: '我的菜单项',
  contexts: ['selection']
});
```

## 6. 上架Chrome Web Store

### 准备材料
- 扩展zip包
- 128x128 图标
- 至少1张截图（1280x800）
- 详细描述（中英文）
- 隐私政策URL

### 流程
1. 注册开发者账号：https://chrome.google.com/webstore/devconsole （一次性 $5）
2. 上传zip包
3. 填写商店信息
4. 提交审核（通常1-3天）

### 定价策略
- 免费 + 高级功能付费（推荐）
- 直接收费（$1-5）
- 免费 + 广告

## 7. 变现技巧

1. **解决真实痛点** — 不要做"又一个TODO"
2. **SEO优化** — 标题和描述包含用户会搜的关键词
3. **好评引导** — 用户使用3天后弹窗请求评价
4. **持续更新** — 每月至少更新一次，提升排名

热门品类：
- 效率工具（广告拦截、标签管理）
- AI辅助（翻译、摘要、写作）
- 开发者工具（API测试、JSON格式化）
- 社交媒体工具（下载、批量操作）

## 总结

Chrome扩展开发门槛低、变现路径清晰。掌握 Manifest V3 + 几个核心API，就能开始做产品了。

---

> 这是Chrome扩展系列第一篇，后续会更新进阶技巧和实战案例。点赞关注不迷路。
