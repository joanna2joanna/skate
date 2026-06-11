---
name: add-skate-spot
description: 搜索、geocode 并添加新城市滑板场地数据到地图。完整流程：搜索 → geocode → 写数据文件 → 接入页面 → 验证 → 提交推送。
---

## 流程

### 1. 搜索
```bash
WebSearch "[城市名] 滑板场 滑板公园"
WebSearch "[城市名] 滑板培训 室内"
WebSearch "[城市名] 街头滑板 spot"
```

### 2. Geocode
Geocode（优先）：
```bash
curl -s "https://restapi.amap.com/v3/geocode/geo?key=932c51e85a92fe6ac1a04008ee818d0a&address=城市+场地名"
```
搜不到用 POI 搜索：
```bash
curl -s "https://restapi.amap.com/v3/place/text?key=932c51e85a92fe6ac1a04008ee818d0a&keywords=场地名&city=城市"
```

### 3. 写数据文件
创建 `data/{citypinyin}.js`：
```js
const {citypinyin}Spots = [
  // -- 滑板公园 --
  { id: 104, city: 'XX', name: '...', type: 'venue', lat: ..., lng: ..., scale: '', operator: '', price: '', description: '', features: [] },
  // -- 培训机构 --
  // -- 街头地形 --
];
```

规则：
- **只加有实际场地可滑的**，规划中/建设中/不确定的一律不加
- type 三选一：`venue`（滑板公园）、`training`（培训机构）、`street`（街头地形）
- ID 按城市分段，同类场地放一起，注释分组

### 4. 接入页面
- `index.html`：`<script src="data/{citypinyin}.js"></script>`（在 `spots.js` 之前）
- `index.html`：`cityCenters` 加城市坐标 `'城市名': [lat, lng, zoom]`
- `data/spots.js`：`[...shenzhenSpots, ...guangzhouSpots, ...{citypinyin}Spots]`

### 5. 验证
```bash
for f in data/*.js; do node --check "$f"; done
```
打开页面检查：城市标签出现、地图定位正确。

### 6. 提交
```bash
git add -A && git commit -m "add [城市名] 滑板场地数据" && git push
```

## 注意事项
- GCJ-02 坐标系，高德返回即用，**不需要** WGS-84 转换
- 私人自建场地（郊区厂房/仓库）WebSearch 难搜到，需用户提供
- 免费场地不写"约XX元/次"，预约制标注清楚
- VPN 可能拦截高德瓦片和 GitHub
- Git push 用 SSH：`git push git@github.com:joanna2joanna/skate.git main`
