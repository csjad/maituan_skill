---
name: meituan-coupon
description: "美团外卖自动领券工具，通过浏览器自动化（paw-browser）在 H5 页面帮助用户领取美团外卖商家优惠券。当用户提到美团领券、美团外卖优惠券、自动领券、美团优惠券领取、签到领券、或需要在美团平台批量领取优惠券时触发。支持短信验证码登录、登录态保存、自动检测和领取可用券。"
---

# Meituan Waimai Coupon Auto Collector（H5 版）

> ⚠️ **风险提醒**
> - 本 skill 仅供学习和个人使用，请遵守美团平台规则
> - 过于频繁的自动化操作可能触发平台风控，请适度使用
> - 脚本可能因美团 UI 更新而失效
> - 建议使用完毕后关闭浏览器会话

## 核心发现

美团**桌面网页版**不提供独立的领券中心，券功能主要集成在 **H5 移动版** 中。本 skill 使用 H5 入口 `i.waimai.meituan.com` 来领取商家优惠券。

## 核心流程

```
H5登录 → 保存登录态 → 进入H5首页 → 扫描商家券标签 → 点击商家进入详情 → 点击领券按钮 → 输出结果
```

## Step 1: 准备登录态

### 登录方式

美团 H5 登录页提供多种登录方式，按优先级推荐：
1. **短信验证码登录**（推荐）- 安全性高，无需密码
2. **账号密码登录** - 需要密码

### 登录流程

```
1. 导航到 H5 登录页
2. 截图展示当前登录方式给用户
3. 引导用户选择并完成登录
4. 保存登录态到本地 JSON 文件
```

```powershell
# 导航到 H5 登录页
paw browser-action '{"action":"navigate","url":"https://i.waimai.meituan.com/","waitUntil":"networkidle"}'
# 页面会重定向到 passport.meituan.com H5 登录页

# 截图展示当前页面给用户
paw browser-action '{"action":"screenshot"}'
```

### 切换到短信验证码登录

如果页面显示的是密码登录，点击切换：

```powershell
# 1. 先看快照确认当前登录方式
paw browser-action '{"action":"snapshot","interactive":true}'

# 2. 点击 "手机动态码登录" 或 "获取验证码" 之类的链接/按钮
paw browser-action '{"action":"click","selector":"@e<N>"}'

# 3. 输入手机号（用户需要提供）
paw browser-action '{"action":"fill","selector":"@e<N>","value":"手机号"}'

# 4. 点击获取验证码
paw browser-action '{"action":"click","selector":"@e<N>"}'

# 5. 提示用户输入收到的验证码

# 6. 输入验证码并提交
paw browser-action '{"action":"fill","selector":"@e<N>","value":"验证码"}'
paw browser-action '{"action":"click","selector":"@e<N>"}'

# 7. 等待登录完成
paw browser-action '{"action":"waitforloadstate","state":"networkidle"}'
```

**登录成功后**，保存登录态：

```powershell
paw browser-action '{"action":"state_save","path":"C:\\Users\\diege\\meituan-coupon-auth.json"}'
```

**后续使用时**，直接加载登录态后导航：

```powershell
paw browser-action '{"action":"state_load","path":"C:\\Users\\diege\\meituan-coupon-auth.json"}'
paw browser-action '{"action":"navigate","url":"https://i.waimai.meituan.com/waimai/mindex/home","waitUntil":"networkidle"}'
```

## Step 2: 进入商家列表（H5）

```powershell
# 导航到 H5 外卖首页
paw browser-action '{"action":"navigate","url":"https://i.waimai.meituan.com/waimai/mindex/home","waitUntil":"networkidle"}'

# 如果有弹窗遮挡（如定位授权弹窗），先关闭
paw browser-action '{"action":"evaluate","script":"var m = document.querySelector(\".modalShadow\"); if(m) { m.style.display = \"none\"; \"hidden\"; } else { \"no modal\"; }"}'
```

## Step 3: 扫描商家券标签

### 使用 evaluate 查找带券标签的商家

```powershell
# 查找页面上的优惠券标签元素
$json = '{"action":"evaluate","script":"var all = document.querySelectorAll(\"div, span\"); var coupons = []; for (var i = 0; i < all.length; i++) { var t = all[i].textContent.trim(); if (t.match(/^[领返新]\\d?[元客券]/) && t.length < 10) { coupons.push({index:i, text:t, tag:all[i].tagName, cls:all[i].className.substring(0,60)}); } } JSON.stringify(coupons.slice(0,15));"}'

paw browser-action $json
```

**H5 商家券标签 CSS 类名结构**（可能会随版本变化）：
- `.d_lb` - 标签内容元素
- `.d_sublabel` - 标签容器
- `.d_lb-wrap` - 标签包装
- `.poilist-item-info5` - 优惠券信息区域
- `.d_cmm-label-comp-wrap` - 通用标签组件

### 券标签文本模式

常见的券标签文本：
- `领2元券`、`领3元券` — 可领取的商家券
- `返2元券` — 下单返券
- `新客减1`、`新客减2`、`新客减3` — 新客立减

## Step 4: 领取优惠券

### 方式一：在商家列表页直接点击券标签领取

```powershell
# 点击 "领2元券" 元素
$json = '{"action":"evaluate","script":"var els = document.querySelectorAll(\".d_lb\"); for (var i = 0; i < els.length; i++) { if (els[i].textContent.trim() === \"领2元券\") { els[i].click(); \"clicked index \" + i; break; } }"}'

paw browser-action $json
```

### 方式二：进入商家详情页领取（推荐）

很多券需要进入商家详情页才能领取：

```powershell
# 1. 点击带券标签的商家卡片，跳转到商家详情页
$json = '{"action":"evaluate","script":"var all = document.querySelectorAll(\".d_lb\"); for (var i = 0; i < all.length; i++) { if (all[i].textContent.trim() === \"领2元券\") { var card = all[i].closest(\"[onclick]\"); if (card) { card.click(); \"clicked card\"; break; } } }"}'

paw browser-action $json

# 2. 等待商家详情页加载
paw browser-action '{"action":"waitforloadstate","state":"networkidle"}'

# 3. 在商家详情页查找领取按钮
#    商家详情页的券按钮 class 通常是 span.root_l0nL51
$json = '{"action":"evaluate","script":"var all = document.querySelectorAll(\"div, span, a\"); var results = []; for (var i = 0; i < all.length; i++) { var text = all[i].textContent.trim(); if (text.match(/^¥\\d领$/) && text.length < 10) { results.push({index:i, text:text, tag:all[i].tagName, cls: all[i].className.substring(0,80)}); } } JSON.stringify(results);"}'

paw browser-action $json

# 4. 点击领券按钮
$json = '{"action":"evaluate","script":"var el = document.querySelectorAll(\"span.root_l0nL51\")[1]; if(el) { el.click(); \"clicked: \" + el.textContent.trim(); } else { \"not found\"; }"}'

paw browser-action $json
```

### 商家详情页券按钮结构

商家详情页顶部通常有：
```
¥1领  ¥2领  ¥3领
38减16  8减2  98减3  128减5  168减10
```

- `span.root_l0nL51` — 领券按钮（¥1领、¥2领等）
- `.receive_DP9lDs` — "收藏门店并领取" 类按钮

## Step 5: 异常处理

### 弹窗/阴影遮挡

美团 H5 常有弹窗（如 `.modalShadow` 定位授权弹窗）遮挡交互元素：

```powershell
# 方式一：按 Escape
paw browser-action '{"action":"press","key":"Escape"}'

# 方式二：用 JS 隐藏
paw browser-action '{"action":"evaluate","script":"var m = document.querySelector(\".modalShadow\"); if(m) { m.style.display = \"none\"; }"}'

# 方式三：点击去开启关闭
paw browser-action '{"action":"evaluate","script":"var btns = document.querySelectorAll(\"div, span, a\"); for (var i = 0; i < btns.length; i++) { if (btns[i].textContent.trim() === \"去开启\") { btns[i].click(); break; } }"}'
```

### 登录态失效

如果 H5 页面又跳转到登录页，说明登录态失效：

```powershell
# 方式一：重新获取 URL 确认
paw browser-action '{"action":"url"}'
# 如果 URL 是 passport 开头，需要重新登录

# 方式二：直接重新登录
paw browser-action '{"action":"navigate","url":"https://i.waimai.meituan.com/","waitUntil":"networkidle"}'
```

### 验证码/风控

如遇到滑块验证或短信验证：

```powershell
# 截图展示给用户
paw browser-action '{"action":"screenshot"}'
# 提示用户手动完成
# 完成后继续...
```

## Step 6: 输出结果

领取完成后，汇总报告：

```
美团外卖 H5 领券完成 🎉

✅ 成功领取的券：
- 擂拌拌：领2元券 ✓
- 擂拌拌：领3元券 ✓

⚠️ 未能确认的：
- 部分券需要下单后才能确认是否到账

📍 领取入口：i.waimai.meituan.com H5 页面
💾 登录态已保存至: C:\Users\diege\meituan-coupon-auth.json
```

## H5 入口 URL 参考

| 用途 | URL | 备注 |
|------|-----|------|
| H5 首页（登录入口）| `https://i.waimai.meituan.com/` | 自动跳转登录 |
| H5 商家列表 | `https://i.waimai.meituan.com/waimai/mindex/home` | 含券标签 |
| H5 我的 | `https://i.waimai.meituan.com/waimai/mindex/my` | 已领券查看 |
| H5 红包 | `https://i.waimai.meituan.com/waimai/mindex/redpacket` | 个人红包 |
| 用户登录态 | `C:\Users\diege\meituan-coupon-auth.json` | state_save/state_load |

## Coupon 元素速查表

| 位置 | 元素类型 | 定位方式 | 说明 |
|------|----------|----------|------|
| 商家列表 | 券标签 | `.d_lb` (text 含 "领X元券") | 商家的券标签 |
| 商家列表 | 可点击卡片 | `[onclick]` 父容器 | 点击进入详情 |
| 商家详情 | 券按钮 | `span.root_l0nL51` (text "¥X领") | 页面顶部领券 |
| 商家详情 | 领取按钮 | `.receive_DP9lDs` | 收藏并领取 |
| 全站弹窗 | 遮挡层 | `.modalShadow` | 需先隐藏 |

## 安全建议

1. 登录态 JSON 文件包含敏感信息，不要分享给他人
2. 定期清理登录态文件（建议每周）
3. 不要设置过于频繁的自动执行
4. 发现账号异常立即停止使用
