# 珠三角滑板场地图

> 静态 HTML/CSS/JS + Leaflet + 高德卫星图。多城市滑板场地展示。

## 技术栈
- 高德 GCJ-02 坐标系，**不需要** WGS-84 转换
- API Key：`932c51e85a92fe6ac1a04008ee818d0a`（仅用于 geocode，不在前端暴露）
- 地图瓦片：`webst{s}.is.autonavi.com/appmaptile?style=6`（卫星）+ `style=8`（路网，opacity 0.55）
- 标记：`L.circleMarker`（radius 12, weight 4 white border），选中金色脉冲动画
- 计数器：counterapi.dev（服务端真实计数）

## 设计系统
- **美学方向**：工业粗野主义（brutalist），混凝土暖灰 + 安全橙强调色
- **字体**：标题 `ZCOOL QingKe HuangYou`（毛笔飞白），UI 系统字体
- **Header**：暗化照片叠加背景，保持文字可读
- **配色变量**：`--bg: #E8E4DD` / `--surface: #F2EFEA` / `--accent: #FF5A1F` / `--text: #1C1C1C`

## 数据规则
- **只加有实际场地可滑的**，规划中/建设中/不确定的一律不加
- 每个场地：`id, city, name, type, lat, lng, scale, operator, price, description, features[]`
- type 三选一：`venue`（滑板公园）、`training`（培训机构）、`street`（街头地形）
- ID 分段：深圳 1-49、广州 51-69、东莞 71-79、佛山 81-89、珠海 91-95、惠州 96-100、中山 101-103、肇庆 104-109
- 每城一个 `data/{citypinyin}.js`，变量名 `const {citypinyin}Spots = [...]`，`data/spots.js` 汇总

## 添加新城市（无需逐步确认）

1. **搜索**：WebSearch `"[城市名] 滑板场 滑板公园"` + `"[城市名] 滑板培训 室内"` + `"[城市名] 街头滑板 spot"`
2. **Geocode**：`curl "https://restapi.amap.com/v3/geocode/geo?key=932c51e85a92fe6ac1a04008ee818d0a&address=城市+场地名"`，搜不到用 POI 搜索：`curl "https://restapi.amap.com/v3/place/text?key=...&keywords=场地名&city=城市"`
3. **写文件**：创建 `data/{citypinyin}.js`，同类场地放一起，注释分组
4. **接入**：`index.html` 加 `<script>` 标签（`spots.js` 之前），`cityCenters` 加坐标，`spots.js` 加 `...spread`
5. **验证**：`for f in data/*.js; do node --check "$f"; done`，打开页面检查城市标签和地图定位
6. **提交**：`git add -A && git commit -m "add [城市名] 滑板场地数据" && git push`

## 注意事项
- VPN 可能拦截高德瓦片（`is.autonavi.com`）和 GitHub（push 失败重试即可）
- 私人自建场地（郊区厂房/仓库）WebSearch 很难搜到，需用户提供信息
- 免费场地不能写"约XX元/次"，预约制场地标注清楚
