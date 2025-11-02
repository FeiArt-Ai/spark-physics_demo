# 互动酒馆演示（Interactive Tavern Demo）

### 在线体验地址：https://spark-physics.netlify.app

![Tavern Demo](tavern.gif)

## 项目简介
本仓库展示了如何把 [**Spark**](https://sparkjs.dev/) 的高斯点渲染（Gaussian Splats）、**Rapier** 物理引擎以及 **Three.js** 结合在一起，搭建一个具有第一人称交互、碰撞检测、空间音频和角色动画的三维酒馆场景。示例演示了如何在保留点云渲染效果的同时，引入传统网格碰撞体用于物理模拟，并通过 Web Audio API 构建沉浸式声音系统。

## 功能亮点
- **高斯点渲染环境**：使用 Spark 渲染器加载 `.spz` 点云文件，并与 Three.js 的相机保持同步，提供细腻的环境细节。
- **真实的物理模拟**：借助 Rapier 构建重力、刚体、碰撞体及连续碰撞检测，实现射弹、叠木块（Jenga）等交互对象的真实表现。
- **骨骼级别的角色碰撞**：为角色骨骼动态生成碰撞球体，使弹丸击中角色时能够准确反馈并触发语音。
- **空间音频系统**：通过 Web Audio API 实现背景音乐、碰撞音效和角色语音的距离衰减、音量/音调调制及静音切换。
- **可视化调试模式**：在高斯点渲染与碰撞网格线框之间切换，可视化骨骼碰撞器，便于调试物理效果。
- **第一人称控制与交互**：支持指针锁定、WASD 移动、R/F 垂直移动、空格调试切换（默认改为 `M`）、点击射击或抓取物体。

## 运行控制
- **点击画面**：进入指针锁定模式或发射弹丸。
- **W / A / S / D**：移动角色。
- **R / F**：向上 / 向下漂浮。
- **M**：切换调试模式（显示碰撞网格、骨骼碰撞器）。
- **空格**：在锁定状态下跳跃。
- **按钮🔊**：切换全局静音。

## 快速上手
### 环境要求
- Node.js 18+（推荐使用 LTS 版本）。
- npm（随 Node.js 安装提供）。

### 安装与启动
```bash
# 安装依赖
npm install

# 启动开发服务器（默认端口 5173）
npm run dev
```
开发服务器启动后将在浏览器自动打开页面，如未自动打开，可手动访问 `http://localhost:5173`。

### 构建与预览
```bash
# 生成生产构建
npm run build

# 在本地预览生产构建
npm run preview
```

## 仓库结构与代码解析
```
├─ index.html           # 页面入口，定义指针锁定 UI、音量按钮及 CDN Import Map
├─ src/
│  └─ main.js           # 项目核心逻辑（渲染、物理、音频、交互）
├─ public/              # 静态资源：模型（.glb/.fbx）、点云（.spz）、音频文件等
├─ vite.config.js       # Vite 配置，包含 COEP/COOP 头、依赖拆分、Rapier 排除等设置
├─ package.json         # 依赖、脚本与工具配置
└─ DEPLOYMENT.md        # Netlify 部署说明
```

### `index.html`
- 通过 `<script type="importmap">` 将 Three.js、Spark、Rapier 的 CDN 地址映射到模块导入路径，使开发服务器和静态托管场景下都能直接运行。
- 页面提供 `Click to play` 按钮、加载提示、中心准星和音量切换按钮；指针锁定控制逻辑由 `src/main.js` 接管。

### `src/main.js`
该文件集中实现所有交互逻辑，可按以下模块理解：
1. **全局配置 (`CONFIG`)**：定义重力、移动速度、音频音量、物理参数、资源路径以及叠叠乐塔（Jenga）的尺寸、层数和初始位置。
2. **通用工具函数**：
   - `setupMaterialsForLighting`：遍历模型材质，转换为标准材质并调整亮度，确保灯光生效。
   - `createBoneColliders`：为角色骨骼节点创建 Rapier 运动学刚体和球形碰撞器，用于命中检测。
   - `loadAudioFiles` 与 `playAudio`：负责音频资源的批量加载与播放。
3. **初始化流程 (`init`)**：
   - 异步初始化 Rapier（含超时保护），创建 Three.js 场景、相机、渲染器以及 SparkRenderer，并注入页面。
   - 配置环境光、方向光与点光源，建立 Rapier 物理世界。
   - 调用 `buildJengaTower` 生成可交互的叠叠乐塔，维护网格与刚体的映射关系以便后续同步。
   - 创建玩家胶囊体刚体，配合 PointerLockControls 管理第一人称相机。
   - 初始化音频上下文：加载背景音乐、弹跳音效、角色语音，并提供全局静音按钮。
   - 加载环境模型和高斯点云 (`SplatMesh`)，并在加载完成后支持碰撞网格与点云之间切换。
   - 通过 `GLTFLoader` 和 `FBXLoader` 加载角色模型，设置动画混合器，生成骨骼碰撞器。
4. **输入与交互**：
   - 监听键鼠事件实现移动、跳跃、调试模式、抓取/释放物体以及射击弹丸。
   - `updateHover` 使用 `THREE.Raycaster` 获取准星前方可抓取的物体，并临时修改材质自发光色实现高亮提示。
5. **物理与动画更新**：
   - `animate` 循环中使用固定时间步长（1/60s）驱动物理模拟，支持多步子迭代以保持稳定。
   - `updateMovement` 根据键盘输入和相机朝向计算目标速度，附带 `adjustVelocityForWalls` 以在墙面附近滑动。
   - 射弹系统为每个弹丸建立球形刚体与网格，处理碰撞音效、角色语音触发以及寿命清理。
   - 将 Rapier 刚体的位姿同步回 Three.js 网格，包括叠叠乐方块、弹丸以及被抓取的物体。
   - 跟随玩家胶囊体更新相机位置；更新 Spark 渲染器、动画混合器和骨骼碰撞器位置；在调试模式下刷新可视化球体。
6. **调试模式**：`toggleDebugMode` 在点云与网格间切换显示，同时记录原始材质、创建骨骼碰撞可视化球体，便于检视碰撞范围。
7. **窗口事件**：监听 `resize` 调整视口大小，使渲染器始终填满窗口。

### `public/` 静态资源
- `*.glb` / `*.fbx`：场景碰撞网格与角色模型。
- `*.spz`：Spark 使用的高斯点云数据。
- `*.mp3`：背景音乐、碰撞音效以及角色语音台词，`lines/` 目录内按角色分类。

### 构建工具 (`vite.config.js`)
- 开发服务器默认自动打开浏览器，并设置 COEP/COOP 响应头以启用 SharedArrayBuffer（Rapier 线程化依赖）。
- 生产构建阶段通过 `manualChunks` 将 Rapier 与 Three.js 拆分成独立代码块，优化缓存。
- `optimizeDeps.exclude` 避免 Rapier 在依赖预构建阶段出错。

## 部署建议
本示例可以直接部署到任意静态站点托管服务，例如 Netlify 或 Vercel。若需手动托管，只需将 `npm run build` 生成的 `dist/` 目录上传即可。部署时请确认服务器允许设置 COEP/COOP 响应头，以保证 Rapier 正常运行。
