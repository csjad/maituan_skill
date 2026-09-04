# 🎫 美团外卖自动领券 Skill

> 一个基于浏览器自动化的美团外卖优惠券领取工具，运行在 [CatPaw](https://catpaw.meituan.com/) 平台。

## ✨ 功能特点

- 🔐 **短信验证码登录** — 安全登录美团账号
- 💾 **登录态保存** — 一次登录，多次使用
- 🔍 **自动检测** — 智能识别商家优惠券
- 🧹 **批量领取** — 一键领取多张券
- 🛡️ **异常处理** — 处理弹窗、验证码等干扰
- 📊 **结果汇总** — 清晰展示领取结果

## 🚀 使用前提

- 安装了 [CatPaw](https://catpaw.meituan.com/) 客户端
- 美团账号（用于接收优惠券）

## 📦 安装

将本 skill 放入 CatPaw 的 skills 目录：

```bash
# 在 CatPaw skills 目录中
git clone https://github.com/你的用户名/meituan-coupon-skill.git
```

或直接下载 `SKILL.md` 文件到 `~/.catpaw/skills/meituan-coupon/` 目录。

## 💬 触发方式

在 CatPaw 中直接说：

- "帮我领美团外卖的券"
- "自动领券"
- "领取美团优惠券"
- "美团外卖券"

## 🔧 技术实现

| 项目 | 说明 |
|------|------|
| 平台 | CatPaw (Meituan) |
| 浏览器自动化 | paw-browser |
| 目标 | 美团外卖 H5 (`i.waimai.meituan.com`) |
| 登录 | 短信验证码 |
| 元素定位 | CSS Selector + JavaScript evaluate |

## ⚠️ 风险提示

- 本工具仅供学习和个人使用
- 请遵守美团平台规则
- 过于频繁可能触发风控
- UI 更新可能导致脚本失效

## 📄 License

[MIT](LICENSE)
