# 大湾区滑板地图

> 静态 HTML/CSS/JS + Leaflet + 高德卫星图。

## 技术栈
- 高德 GCJ-02 坐标系，API Key `932c51e85a92fe6ac1a04008ee818d0a`（仅 geocode，不暴露前端）
- 地图瓦片：`webst{s}.is.autonavi.com/appmaptile?style=6`（卫星）+ `style=8`（路网，opacity 0.55）
- 标记：`L.circleMarker`，选中金色脉冲动画
- 计数器：counterapi.dev（只读查询需尾斜杠 `/visitors/`，不带尾斜杠返回 404）

## 数据规则
- **只加有实际场地可滑的**，规划中/建设中一律不加
- type 三选一：`venue` / `training` / `street`
- ID 分段：深圳 1-49、广州 51-69、东莞 71-79、佛山 81-89、珠海 91-95、惠州 96-100、中山 101-103、肇庆 104-109、香港 111-129、澳门 131-139
- 每城 `data/{citypinyin}.js`，`data/spots.js` 汇总
- 免费场地不能写"约XX元/次"，预约制标注清楚

## 添加场地流程

1. **搜索**：WebSearch 搜 `"[城市] 滑板场"`、`"[城市] 滑板培训 室内"`、`"[城市] 陆冲 泵道"`、`"[城市] 街头滑板 spot"`，也搜社交媒体（小红书/微信文章）
2. **Geocode**：直接用高德 POI 搜索 `curl "https://restapi.amap.com/v3/place/text?key=...&keywords=场地名&city=城市"`，不逐一确认
3. **写入**：追加到对应 `data/{city}.js`，同类场地放一起，注释分组
4. **接入**：新城市需在 `index.html` 加 `<script>`、`cityCenters` 加坐标、`spots.js` 加 spread
5. **验证**：`for f in data/*.js; do node --check "$f"; done`
6. **提交**：`git commit -m "..." && git push`（用 SSH，HTTPS 443 被阻断）

## 注意事项
- 私人自建场地（厂房/仓库）搜索引擎难找，需用户提供
- 陆冲场地包括泵道、浪壁、沿海碧道等，与滑板场不完全重叠
- 优先推荐零注册方案（如 FormSubmit），避免让用户注册第三方服务
