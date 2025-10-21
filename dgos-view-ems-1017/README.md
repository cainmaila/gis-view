# DGOS-VIEWER Web Component 文檔

DGOS-VIEWER（`<dgos-viewer>`）是一個基於 3D 地理資訊系統 Web Component，提供高效能的地圖瀏覽、標記點管理和 3D 模型展示等功能。

## 概述

DGOS-VIEWER 透過 CSV 資料格式支援動態標記點和 3D 模型的渲染。元件使用 Web Components 標準，可輕鬆整合至任何前端框架或原生 HTML 應用中。

## 快速開始

### 基本使用

```html
<script type="module" src="./wc/dgos-1017.es.js"></script>

<dgos-viewer
	markerscsv="id,name,lon,lat,alt,heading,pitch,glbPath
marker-1,Point 1,121.5,25.1,100,0,45,./models/model1.glb"
	imagerytype="AERIAL"
></dgos-viewer>

<script>
	const viewer = document.querySelector('dgos-viewer')

	// 監聽載入完成事件
	viewer.addEventListener('ready', () => {
		console.log('Viewer 已準備就緒')
	})
</script>
```

## 使用 DGOS Editor 生成 CSV 資料

您可以使用 **DGOS Editor** 輕鬆生成和編輯標記點與模型資料，無需手動撰寫 CSV 格式：

🔗 **DGOS Editor 模型編輯器**：https://dgos-editor.vercel.app/models

**使用流程：**

1. 開啟 [DGOS Editor 模型編輯器](https://dgos-editor.vercel.app/models)
2. 在編輯器中新增和配置標記點（位置、旋轉角度、3D 模型等）
3. 編輯完成後，直接複製生成的 CSV 資料
4. 將 CSV 資料貼入 `<dgos-viewer>` 的 `markerscsv` 屬性

這樣可以避免手動編寫 CSV 的繁瑣過程，提高工作效率。

## 屬性 (Props)

### `markerscsv` (字串)

CSV 格式的標記點與模型資料。格式如下：

```csv
id,name,lon,lat,alt,heading,pitch,glbPath,scale
marker-1,地點一,121.446151,25.172627,100,0,45,./models/model.glb,1.0
marker-2,地點二,121.450000,25.175000,150,90,30,./models/model2.glb,1.5
```

**CSV 欄位說明：**

- `id` (必填): 標記點唯一識別碼
- `name` (可選): 標記點名稱
- `lon` (必填): 經度（WGS84）
- `lat` (必填): 緯度（WGS84）
- `alt` (可選): 海拔高度（公尺，預設 0）
- `heading` (可選): 水平旋轉角度（0-360°，預設 0）
- `pitch` (可選): 傾斜角度（0-90°，預設 90 表示俯視，該參數與 Cesium 中數字呈反向）
- `glbPath` (可選): GLB 3D 模型檔案路徑
- `scale` (可選): 3D 模型縮放倍數（預設 1.0）

### `imagerytype` (字串)

底圖影像類型，預設值為 `aerial`（衛星影像）。

**支援的值：**

- `aerial`: 衛星影像
- `aerialWithLabels`: 衛星影像帶道路標籤
- `road`: 道路地圖
- 其他由 Cesium 提供的影像圖層類型

**使用範例：**

```html
<!-- 衛星影像 -->
<dgos-viewer imagerytype="aerial"></dgos-viewer>

<!-- 衛星影像帶標籤 -->
<dgos-viewer imagerytype="aerialWithLabels"></dgos-viewer>

<!-- 道路地圖 -->
<dgos-viewer imagerytype="road"></dgos-viewer>
```

## 方法 (Methods)

### `setCameraStatus(status: number[])`

設定相機的位置和視角。

**參數：**

- `status`: 陣列格式 `[longitude, latitude, height, heading, pitch]`
  - `longitude`: 經度（WGS84）
  - `latitude`: 緯度（WGS84）
  - `height`: 相機高度（公尺）
  - `heading`: 水平旋轉角度（0-360°）
  - `pitch`: 傾斜角度（0-90°）

**使用範例：**

```javascript
const viewer = document.querySelector('dgos-viewer')

// 監聽 ready 事件後再呼叫
viewer.addEventListener('ready', () => {
	// 設定相機到台北市，高度 1000m，俯視角 45 度
	viewer.setCameraStatus([121.5645, 25.033, 1000, 0, 45])
})
```

### `flyToId(id: string)`

飛行到指定 ID 的標記點位置。

**參數：**

- `id`: CSV 中定義的標記點 ID

**功能特性：**

- 飛行時間：3 秒
- 視角：向下俯視 45 度
- 距離標記點：約 500 公尺半徑

**使用範例：**

```javascript
const viewer = document.querySelector('dgos-viewer')

viewer.addEventListener('ready', () => {
	// 飛到 ID 為 'marker-1' 的位置
	viewer.flyToId('marker-1')
})
```

### `getMarkers(): Promise<I_MappingPoint[]>`

取得所有已解析的標記點資料。

**回傳值：** 包含所有標記點資訊的 Promise

**使用範例：**

```javascript
const viewer = document.querySelector('dgos-viewer')

viewer.addEventListener('ready', async () => {
	const markers = await viewer.getMarkers()
	console.log('所有標記點:', markers)
})
```

## 事件 (Events)

### `ready`

當元件完全載入並初始化完成時觸發。

**使用範例：**

```javascript
viewer.addEventListener('ready', () => {
	console.log('Viewer 已準備就緒，可以開始呼叫方法')
})
```

### `camerastatus`

當相機位置或視角改變時觸發。可用於記錄或同步相機狀態。

**回傳資料：** `event.detail` 為陣列 `[longitude, latitude, height, heading, pitch]`

**使用範例：**

```javascript
viewer.addEventListener('camerastatus', (event) => {
	const [lon, lat, height, heading, pitch] = event.detail
	console.log(`相機位置: ${lon}, ${lat}, 高度: ${height}m`)
	console.log(`相機方向: 水平 ${heading}°, 傾斜 ${pitch}°`)

	// 可用於保存相機狀態供下次載入
	localStorage.setItem('cameraStatus', JSON.stringify(event.detail))
})
```

### `selected`

當選中某個標記點時觸發。

**回傳資料：** `event.detail` 為標記點的 ID（字串）

**使用範例：**

```javascript
viewer.addEventListener('selected', (event) => {
	const markerId = event.detail
	console.log('已選中標記點:', markerId)
})
```

## 完整使用範例

### HTML 範例

```html
<!DOCTYPE html>
<html lang="zh-TW">
	<head>
		<meta charset="UTF-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />
		<title>FK-VIEWER 範例</title>
		<style>
			body {
				margin: 0;
				padding: 0;
				font-family: sans-serif;
			}
			#app {
				width: 100vw;
				height: 100vh;
			}
			.controls {
				position: absolute;
				top: 20px;
				right: 20px;
				background: white;
				padding: 15px;
				border-radius: 8px;
				box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
				z-index: 100;
			}
			button {
				display: block;
				margin-bottom: 10px;
				padding: 8px 16px;
				background: #007bff;
				color: white;
				border: none;
				border-radius: 4px;
				cursor: pointer;
			}
			button:hover {
				background: #0056b3;
			}
		</style>
	</head>
	<body>
		<div id="app">
			<dgos-viewer
				id="viewer"
				markerscsv="id,name,lon,lat,alt,heading,pitch,glbPath
marker-1,台北市,121.5645,25.0330,500,0,45
marker-2,台中市,120.6731,24.1372,300,90,30"
				imagerytype="AERIAL"
			></dgos-viewer>

			<div class="controls">
				<button onclick="flyToMarker1()">飛到台北</button>
				<button onclick="flyToMarker2()">飛到台中</button>
				<button onclick="getMarkerInfo()">顯示標記點</button>
			</div>
		</div>

		<script type="module" src="./wc/dgos-1017.es.js"></script>
		<script>
			const viewer = document.getElementById('viewer')

			viewer.addEventListener('ready', () => {
				console.log('Viewer 已載入完成')
			})

			viewer.addEventListener('camerastatus', (event) => {
				console.log('相機狀態:', event.detail)
			})

			viewer.addEventListener('selected', (event) => {
				console.log('選中標記點:', event.detail)
			})

			function flyToMarker1() {
				viewer.flyToId('marker-1')
			}

			function flyToMarker2() {
				viewer.flyToId('marker-2')
			}

			async function getMarkerInfo() {
				const markers = await viewer.getMarkers()
				console.table(markers)
			}
		</script>
	</body>
</html>
```

### 動態更新標記點

```javascript
const viewer = document.querySelector('dgos-viewer')

// 動態修改 markerscsv 屬性以更新標記點
viewer.setAttribute(
	'markerscsv',
	`id,name,lon,lat
marker-new-1,新地點一,121.6,25.0
marker-new-2,新地點二,121.7,25.1`
)
```

### 保存和還原相機狀態

```javascript
const viewer = document.querySelector('dgos-viewer')

// 監聽相機變化並保存
viewer.addEventListener('camerastatus', (event) => {
	sessionStorage.setItem('lastCameraStatus', JSON.stringify(event.detail))
})

// 載入後還原相機狀態
viewer.addEventListener('ready', () => {
	const savedStatus = sessionStorage.getItem('lastCameraStatus')
	if (savedStatus) {
		viewer.setCameraStatus(JSON.parse(savedStatus))
	}
})
```

## 技術需求

- **瀏覽器支援**：需支援 Web Components 和 WebGL 的現代瀏覽器
  - Chrome 60+
  - Firefox 63+
  - Safari 13+
  - Edge 79+
- **網路連線**：建議具備穩定的網路連線以載入資源和地圖圖層
- **硬體需求**：建議具備獨立 GPU 以獲得最佳的 3D 渲染效能

## 常見問題

### Q: 如何使用自訂的 3D 模型？

A: 在 CSV 的 `glbPath` 欄位中指定模型檔案的 URL 或相對路徑即可。支援的格式為 GLB/GLTF。

```csv
marker-1,建築,121.5,25.1,0,0,45,./models/building.glb,1.0
```

### Q: 可以在執行時動態新增或移除標記點嗎？

A: 可以。透過更新 `markerscsv` 屬性，元件會自動重新解析 CSV 資料並更新標記點。

```javascript
viewer.setAttribute('markerscsv', newCsvData)
```
