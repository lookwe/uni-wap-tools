# 🧰 uni-wap-tools

<div align="center">

**📱 基于 uni-app x 的原生 Android 工具箱**

*悬浮时钟 · 全屏计时 · 效率工具集合*

</div>

---

## 📋 项目简介

`uni-wap-tools`（工具箱）是一个 **uni-app x** 原生 Android 工具应用，编译为纯原生 Kotlin 代码，以 APK 方式分发。目前已实现 **悬浮时钟** 功能，可在任意应用上层显示可拖拽、可调整字号的实时时间。

<p align="center">
  <img src="./static/logo.png" width="120" alt="Logo">
</p>

## ✨ 功能特性

### ⏰ 悬浮时钟

| 特性 | 说明 |
|---|---|
| 🔲 **悬浮窗显示** | 在所有应用上层显示半透明数字时钟 |
| 👆 **手势拖拽** | 手指拖动时钟到屏幕任意位置，自动限制在屏幕范围内 |
| 📐 **弧形菜单** | 点击时钟弹出弧形快捷菜单，Apple Fluid 动画曲线 |
| 🔤 **字号调节** | 支持 8sp ~ 24sp 范围、2sp 步进的字号调整 |
| 📍 **智能方向** | 菜单展开方向根据时钟位置自动判断左右 |
| ⏳ **自动收起** | 菜单 5 秒无操作自动收起 |

### 🎨 动画效果

- 全部动画采用 **Apple Fluid 风格**（`cubic-bezier(0.22, 1, 0.36, 1)`）
- 菜单展开/收起动画：**320ms**
- 流畅自然，无弹跳无抖动

## 🏗️ 技术架构

```
uni-wap-tools/
├── 📄 main.uts                          # 应用入口
├── 📄 App.uvue                          # 根组件 + 双击返回退出
├── 📄 pages.json                        # 路由配置
├── 📄 manifest.json                     # 应用配置 + 权限声明
├── 📄 uni.scss                          # 全局 SCSS 变量
├── 📄 pages/
│   ├── 📁 index/
│   │   └── 📄 index.uvue                # 🏠 首页工具卡片
│   └── 📁 floating-clock/
│       └── 📄 index.uvue                # ⏰ 悬浮时钟控制页
├── 📁 utssdk/
│   └── 📁 floating-clock/
│       └── 📄 index.uts                 # 🔧 原生 Android 悬浮时钟插件
├── 📁 static/
│   └── 🖼️ logo.png                      # 应用图标
└── 📄 AGENTS.md                         # 开发规范与踩坑记录
```

| 层级 | 技术 | 说明 |
|---|---|---|
| 🎯 **框架** | uni-app x | 特殊变体，编译为原生 Kotlin |
| 🧩 **组件** | `.uvue` 文件 | Vue 风格模板 + UTS 脚本 |
| ⚙️ **脚本** | `.uts` (UniApp TypeScript) | TS 子集，编译为 Kotlin 而非 JS |
| 📦 **原生** | Android SDK API | WindowManager、ValueAnimator、PathInterpolator |
| 🛠️ **构建** | HBuilderX IDE | 无 CLI，无 npm，无 Node.js 工具链 |

## 🚀 快速开始

### 环境要求

| 工具 | 说明 |
|---|---|
| 🔧 **HBuilderX** | 唯一构建工具，[下载地址](https://www.dcloud.io/hbuilderx.html) |
| 📱 **Android 设备** | Android 5.0+ |
| 🔑 **悬浮窗权限** | 应用需开启 `显示在其他应用的上层` 权限 |

### 构建步骤

1. 📂 用 **HBuilderX** 打开本项目目录
2. 🔗 连接 Android 设备或启动模拟器
3. ▶️ 点击 **运行 → 运行到手机或模拟器**
4. 📱 APK 自动安装并启动

### 使用悬浮时钟

1. 🏠 首页点击 **"悬浮时钟"** 卡片
2. 🔐 按提示前往系统设置开启 **悬浮窗权限**
3. ✅ 返回应用，点击 **"启动悬浮时钟"**
4. 👆 时钟出现后：
   - **拖拽** 移动位置
   - **点击** 弹出菜单（字号 ± / 关闭）

## 📄 路由

| 路径 | 标题 | 说明 |
|---|---|---|
| `pages/index/index` | 🏠 工具箱 | 启动页，工具卡片入口 |
| `pages/floating-clock/index` | ⏰ 悬浮时钟 | 控制面板，权限检查与开关 |

## ⚠️ 开发注意事项

### 🔑 核心规则

- ❌ **不要添加** `package.json`、npm 依赖
- ❌ **不要编辑** `unpackage/` 目录（构建输出）
- ❌ **不要使用** Vue Router、Pinia、Vuex
- ✅ **使用** `uni.navigateTo` 导航、`ref()` / `reactive()` 管理状态
- ✅ **使用** `// #ifdef APP-ANDROID` 进行平台条件编译
- ✅ **使用** 相对路径导入 UTS 插件（`@/` 别名无效）

### 🐛 常见编译错误速查

<details>
<summary>📋 类型系统</summary>

| 错误 | 🔧 修复 |
|---|---|
| `找不到名称'kVal'` | ❌ 不支持 Kotlin `val`，用 `const` / `let` |
| `Math.cos/sin` 返回 `Number` | ➕ 加 `.toDouble()` 转换 |
| `setInterval` 返回 `Number` | 变量声明 `: Number` 上界 |
| 数字传参不匹配 `Float` | ➕ 加 `.toFloat()` |
| 函数引用不匹配 | 用匿名函数包装：`setInterval(function(){ tick() }, 1000)` |

</details>

<details>
<summary>📋 闭包与作用域</summary>

| 错误 | 🔧 修复 |
|---|---|
| 闭包找不到局部 `const` | 改用**模块级 `let`** 变量 |

</details>

<details>
<summary>📋 Java 接口</summary>

| 错误 | 🔧 修复 |
|---|---|
| SAM 接口需显式包装 | `View.OnTouchListener(function(v, e): Boolean {...})` |
| 避免匿名内部类 | 用 `setInterval` 定时器代替 `AnimatorListener` |
| `catch (e: any)` | 去掉类型：`catch (e) { }` |

</details>

<details>
<summary>📋 模块与路径</summary>

| 错误 | 🔧 修复 |
|---|---|
| `@/` 别名无法解析 | 用相对路径：`../../utssdk/floating-clock/index` |
| 编译卡死无报错 | `utssdk/` 下不要放 `package.json` |

</details>

> 📖 更多细节见 [AGENTS.md](./AGENTS.md)

## 🎨 设计规范

本项目遵循 **Apple Fluid** 动画规范：

| 参数 | 值 |
|---|---|
| 🎯 **缓动曲线** | `cubic-bezier(0.22, 1, 0.36, 1)` |
| ⏱️ **弹窗动画** | 320ms |
| ↕️ **入场偏移** | 28px 底弹 |
| 📐 **卡片间隔** | 45ms 入场 |

### 🚫 动画禁区

- ⛔ 禁止弹跳效果
- ⛔ 禁止元素抖动
- ⛔ 禁止劫持页面滚动

## 📦 权限

| 权限 | 用途 |
|---|---|
| `android.permission.SYSTEM_ALERT_WINDOW` | 🪟 悬浮窗显示 |

---

<div align="center">

**Built with ❤️ using [uni-app x](https://uniapp.dcloud.net.cn/) and HBuilderX**

</div>
