# SG Weather & PSI

新加坡天气与 PSI（空气质量）实时地图，单文件静态网页。基于 [OneMap](https://www.onemap.gov.sg/) 底图 + 新加坡国家环境局（NEA）开放数据，无需后端、无需构建。

## 功能

- **PSI 模式**：展示新加坡 5 个区域（West / North / Central / East / South）的实时 PSI 读数，圆标按 PSI 等级着色（优 / 中等 / 不健康 / 非常不健康 / 危险）。
- **降雨模式**：基于 NEA 2 小时天气预报，在约 47 个区域上标注天气图标（晴 / 多云 / 雷阵雨 等）。
- **底图切换**：Default / Night / Grey 三种 OneMap 样式。
- **中英文双语**：界面默认跟随浏览器语言，可在右上角切换，选择记忆到 `localStorage`。
- **移动端优化**：窄屏下头部与控制栏纵向堆叠、点击区 ≥ 42px、地图高度按视口（`vh`）自适应、适配 iOS 安全区；东西两侧气泡在窄屏自动内移以保证完整可见。

## 数据来源

| 数据 | 接口 |
|---|---|
| PSI 实时读数 | `https://api.data.gov.sg/v1/environment/psi` |
| 2 小时天气预报 | `https://api.data.gov.sg/v1/environment/2-hour-weather-forecast` |
| 地图底图瓦片 | `https://www.onemap.gov.sg/maps/tiles/{Default\|Night\|Grey}/{z}/{x}/{y}.png` |

地图底图版权归 OneMap / Singapore Land Authority 所有，已在页面署名。

## 技术栈

- 单文件 HTML（内联 CSS + 原生 JS），无构建步骤
- [Leaflet](https://leafletjs.com/) 1.9.4（通过 unpkg CDN 加载）渲染地图
- Google Fonts（Outfit / JetBrains Mono）

## 目录结构

```
sg-weather/
├── sg-weather-psi.html   # 主源码（开发在这里改）
├── deploy/
│   └── index.html        # 部署根，由 Cloudflare Pages 托管（与源码同步）
├── .workbuddy/           # 项目工作记忆（本地，不纳入版本控制）
└── README.md
```

## 本地运行

纯静态，直接双击 `sg-weather-psi.html` 即可；或起一个本地静态服务器：

```bash
python3 -m http.server 8080
# 浏览器打开 http://127.0.0.1:8080/sg-weather-psi.html
```

> 注意：NEA / OneMap 接口需要联网；部分接口在跨域（CORS）下可能受限，本地 `file://` 或 `http://localhost` 通常可用。

## 部署（Cloudflare Pages）

部署目录为 `deploy/`，托管后线上即是 `deploy/index.html`。

```bash
# 1. 保持源码与部署根同步
cp sg-weather-psi.html deploy/index.html

# 2. 用 wrangler 部署
npx wrangler pages deploy deploy --project-name sg-weather --branch main
```

主域名（`<project>.pages.dev`）在部署后有数秒传播延迟，验收时建议带 `?v=<timestamp>` 缓存旁路或等待片刻。

## 已知限制 / 备注

- 底图切换实现上**重建 Leaflet 图层**而非 `setUrl()`：Leaflet 1.9.4 的 `GridLayer.redraw()` 在 `zoomSnap:0.25` 下不会取整 zoom，会请求小数层级瓦片而 404。重建图层走 `onAdd` 路径可避免该问题。
- OneMap 对新加坡覆盖范围之外的瓦片返回 200 但 body 为 0 字节；已将地图容器底色设为各样式对应的「海面色」（Default `#6da8e4` / Night `#003753` / Grey `#d1d1d1`），使空白区融入海面。
- 数据刷新为手动（页面加载 / 点刷新按钮），未做自动轮询。
