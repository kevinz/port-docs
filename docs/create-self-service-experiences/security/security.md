---

title: 安全

---

# 安全

该页面包括允许 Port 在使用自助操作时与[backend](../setup-backend/setup-backend.md) 互动所需的其他配置。

## 用于允许列表的Port IP 地址

当 Port 向外拨打电话时(例如使用[Webhook](../setup-backend/webhook/webhook.md) 引用方法时) ，源 IP 地址是静态的。

Port外呼将从以下 IP 地址之一发起: 

```text showLineNumbers
44.221.30.248, 44.193.148.179, 34.197.132.205, 3.251.12.205
```