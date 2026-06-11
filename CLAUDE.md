# 广东滑板场地图

> 静态 HTML/CSS/JS + Leaflet + 高德卫星图。多城市滑板场地展示，含运营场地/训练场/街头滑板点三种类型。

## 技术要点
- 坐标系：高德 GCJ-02，**不需要** WGS-84 转换
- 高德 API Key：`932c51e85a92fe6ac1a04008ee818d0a`（用于 geocode，不在前端暴露）
- 地图瓦片：`webst{s}.is.autonavi.com/appmaptile?style=6`（卫星底图）+ `style=8`（半透明路网标注，opacity 0.55）
- 标记：`L.circleMarker`（radius 12, weight 4 white border），选中时金色脉冲动画
- 数据文件：每城一个 `data/{citypinyin}.js`，`data/spots.js` 汇总

## 数据规则
- **只加有实际场地可滑的**，规划中/建设中/不确定的一律不加
- 每个场地必须有：`id, city, name, type, lat, lng, scale, operator, price, description, features[]`
- type 三选一：`venue`（运营场地）、`training`（训练场）、`street`（街头滑板点）
- ID 按城市分段：深圳 1-49、广州 51-69、东莞 71-79、佛山 81-89、珠海 91-95、惠州 96-100、中山 101-103、肇庆 104-109

## 添加新城市流程（无需逐步确认，信任执行）

### 1. 搜索场地
- WebSearch：`"[城市名] 滑板场 滑板公园 极限运动"` + `"[城市名] 滑板培训 室内滑板场"` + `"[城市名] 街头滑板 spot 地形"`
- 去重，分类为 venue/training/street，整理成列表

### 2. 批量获取坐标
- 用高德 API 逐个 geocode：
  ```
  curl "https://restapi.amap.com/v3/geocode/geo?key=932c51e85a92fe6ac1a04008ee818d0a&address=城市+场地名"
  ```
- 高德返回的就是 GCJ-02 坐标，直接用
- 搜不到的用高德 POI 搜索：
  ```
  curl "https://restapi.amap.com/v3/place/text?key=932c51e85a92fe6ac1a04008ee818d0a&keywords=场地名&city=城市"
  ```

### 3. 写入数据文件
- 创建 `data/{citypinyin}.js`，变量名 `const {citypinyin}Spots = [...]`
- ID 从下一个可用分段开始
- 同类场地放在一起，注释分组

### 4. 接入项目
- `index.html` `<head>` 中加入 `<script src="data/{citypinyin}.js"></script>`（在 `spots.js` 之前）
- `index.html` JS 的 `cityCenters` 对象中加入城市中心坐标和默认 zoom
- `data/spots.js` 数组中加 `...{citypinyin}Spots,`

### 5. 验证
- `for f in data/*.js; do node --check "$f" || echo "FAIL: $f"; done`
- 打开页面检查：城市标签栏出现新城市、点击能切换到正确地图位置

### 6. 提交
- `git add -A && git commit -m "add [城市名] 滑板场地数据" && git push`

## 注意事项
- 用户使用科学上网可能看不到高德地图瓦片，这是网络代理问题，非代码问题
- 私人自建场地（郊区厂房/仓库）WebSearch 很难搜到，用户可能自行提供信息
- 场馆如果"规划中/建设中"不加，"免费"的场地不能写"约XX元/次"
