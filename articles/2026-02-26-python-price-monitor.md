# 用 Python 写一个自动化赚钱工具：监控商品降价并通知

> 全自动监控电商价格，降价立刻通知你。代码不到100行，小白也能跑。

## 场景

你想买一个东西但嫌贵，想等降价。手动刷太累，写个脚本自动盯着，降了就微信/邮件通知你。

还能帮别人监控，收个服务费 —— 这就是自动化赚钱的思路。

## 技术栈

- Python 3.11
- requests + BeautifulSoup（爬取价格）
- schedule（定时任务）
- pushplus（微信推送，免费）

## 完整代码

```python
import requests
from bs4 import BeautifulSoup
import schedule
import time
import json

class PriceMonitor:
    def __init__(self, config_file="products.json"):
        self.config_file = config_file
        self.products = self.load_products()
        # PushPlus 微信推送 token（免费申请：pushplus.plus）
        self.pushplus_token = "YOUR_TOKEN_HERE"
    
    def load_products(self):
        """加载监控商品列表"""
        try:
            with open(self.config_file) as f:
                return json.load(f)
        except FileNotFoundError:
            return []
    
    def save_products(self):
        with open(self.config_file, "w") as f:
            json.dump(self.products, f, ensure_ascii=False, indent=2)
    
    def get_price(self, url):
        """获取商品价格（以京东为例）"""
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                          "AppleWebKit/537.36 Chrome/120.0.0.0 Safari/537.36"
        }
        try:
            resp = requests.get(url, headers=headers, timeout=10)
            soup = BeautifulSoup(resp.text, "html.parser")
            
            # 不同平台解析方式不同，这里是通用示例
            # 实际使用需要根据目标网站调整选择器
            price_elem = soup.select_one(".price, .p-price, .tb-rmb-num")
            if price_elem:
                price_text = price_elem.get_text(strip=True)
                price = float(''.join(c for c in price_text if c.isdigit() or c == '.'))
                return price
        except Exception as e:
            print(f"获取价格失败: {e}")
        return None
    
    def notify(self, title, content):
        """微信推送通知"""
        url = f"http://www.pushplus.plus/send"
        data = {
            "token": self.pushplus_token,
            "title": title,
            "content": content,
            "template": "html"
        }
        try:
            requests.post(url, json=data, timeout=10)
            print(f"通知已发送: {title}")
        except Exception as e:
            print(f"通知发送失败: {e}")
    
    def check_prices(self):
        """检查所有商品价格"""
        print(f"[{time.strftime('%H:%M:%S')}] 开始检查价格...")
        
        for product in self.products:
            current_price = self.get_price(product["url"])
            if current_price is None:
                continue
            
            name = product["name"]
            target = product["target_price"]
            last = product.get("last_price", current_price)
            
            # 降价通知
            if current_price <= target:
                self.notify(
                    f"🎉 {name} 降到目标价了！",
                    f"<b>{name}</b><br>"
                    f"当前价格: ¥{current_price}<br>"
                    f"目标价格: ¥{target}<br>"
                    f"<a href='{product['url']}'>立即购买</a>"
                )
            # 价格变动通知
            elif current_price < last:
                self.notify(
                    f"📉 {name} 降价了",
                    f"<b>{name}</b><br>"
                    f"原价: ¥{last} → 现价: ¥{current_price}<br>"
                    f"目标价: ¥{target}（还差 ¥{current_price - target:.2f}）"
                )
            
            product["last_price"] = current_price
        
        self.save_products()
        print("检查完成")
    
    def add_product(self, name, url, target_price):
        """添加监控商品"""
        self.products.append({
            "name": name,
            "url": url,
            "target_price": target_price
        })
        self.save_products()
        print(f"已添加: {name}（目标价 ¥{target_price}）")
    
    def run(self, interval_minutes=30):
        """启动监控"""
        print(f"价格监控已启动，每 {interval_minutes} 分钟检查一次")
        print(f"监控 {len(self.products)} 个商品")
        
        self.check_prices()  # 立即检查一次
        schedule.every(interval_minutes).minutes.do(self.check_prices)
        
        while True:
            schedule.run_pending()
            time.sleep(1)

if __name__ == "__main__":
    monitor = PriceMonitor()
    
    # 添加要监控的商品
    # monitor.add_product("AirPods Pro", "https://item.jd.com/xxx", 1499)
    
    monitor.run(interval_minutes=30)
```

## 配置文件 products.json

```json
[
  {
    "name": "AirPods Pro 2",
    "url": "https://item.jd.com/100038004540.html",
    "target_price": 1499
  },
  {
    "name": "机械键盘",
    "url": "https://item.jd.com/100012345.html",
    "target_price": 299
  }
]
```

## 部署到服务器

```bash
pip install requests beautifulsoup4 schedule
nohup python monitor.py > monitor.log 2>&1 &
```

## 赚钱思路

1. **代监控服务**：帮别人监控商品，收 5-10 元/月
2. **做成小程序**：用户自助添加监控，按次收费
3. **返利结合**：降价通知带返利链接，赚佣金

## 总结

不到 100 行核心代码，就能实现一个实用的价格监控工具。自动化的魅力就在于：写一次代码，持续产生价值。

---

*点赞收藏，下期教你做微信机器人自动接单 👍*
