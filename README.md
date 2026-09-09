# 太平慧管家（WorkBuddy 迁移版）

太平保险代理人专属智能事业伙伴 —— 从 Kimi（xyrjweloa4nq4.ok.kimi.link）完整迁移的静态 SPA（React + Vite 构建产物）。

## 目录结构

```
index.html                          入口（已去除 Kimi sdk-seed 埋点，lang 改为 zh-CN，新增内联 SVG favicon）
assets/index---udaeVd.js            主 JS 包（含两处 bug 修复 + 水印功能补齐，见下）
assets/index-BbKmavjm.css           样式表
assets/__vite-browser-external-BIHI7g3E.js  动态分包（Node API 占位，浏览器端不执行）
images/warm-*.jpg                   4 张页面素材图（hero / meeting / stilllife / texture）
```

## 本地运行

任意静态服务器即可，例如：

```bash
cd 本目录
python -m http.server 8917 --bind 127.0.0.1
# 浏览器打开 http://127.0.0.1:8917/
```

## 迁移时发现并修复的问题

| # | 问题 | 修复 |
|---|------|------|
| 1 | `images/warm-texture.jpg` 未随迁移抓取，页面 404 | 已从源站补齐全部 4 张图 |
| 2 | Kimi 平台埋点脚本 `kimi.com/sdk-seed.js` 迁出后无效且拖慢加载 | 已从 index.html 移除 |
| 3 | favicon 404（原站即无 favicon 文件） | index.html 内联金色「太」字 SVG 图标 |
| 4 | 【原站真 bug】太平课堂 PPT 下载必失败：`y.write({type:"blob"})` 传参错误，JSZip `checkSupport` 抛 `toLowerCase is not a function` | 改为 `y.write("blob")`，实测可下载 ~200KB .pptx |
| 5 | 【原站真 bug】IP打造-水印生成按钮无任何 onClick、无文件选择器、无打水印逻辑（纯摆设） | 在 bundle 内注入完整实现：选照片 → canvas 按所选风格（简约/优雅/商务）叠加「中国太平 + 代理人姓名（+官方认证代理人）」底部渐变水印 → 下载 PNG。实测通过 |

## 回归测试结论（Playwright + Chrome 实测）

- 首页加载渲染：PASS
- 内容工坊 AI 生成（DeepSeek 接口，模型 deepseek-v4-flash-vision-exp）：PASS
- 太平课堂 PPT 生成并下载：PASS
- IP打造模块（Slogan / 水印 / 形象指南）：PASS
- 水印上传照片 → 生成 → 下载 PNG：PASS
- 控制台错误 / 页面异常 / 404：0 条

## ⚠️ 已知风险（迁移原样保留，建议后续处理）

1. **DeepSeek API Key 硬编码在前端 JS 中**（`assets/index---udaeVd.js` 内明文）。任何人打开浏览器开发者工具即可窃取该 Key 盗刷额度。2026-09-09 已换新 Key（hi-68fy…）与视觉模型 deepseek-v4-flash-vision-exp，旧 Key sk-9c24…已不再使用。正式上线前应改为后端代理转发，前端不落 Key。
2. AI 直连 `api.deepseek.com`，浏览器跨域依赖对方 CORS 策略，若其策略收紧会直接影响内容生成功能。
3. 「今日额度」（100 次/天）仅存于前端本地，清缓存即可绕过，无实际限制力。
