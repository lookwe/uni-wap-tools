# AGENTS.md

## 项目概述

这是一个 **uni-app x** 工具类集合 APP（不是标准 uni-app 或 Vue）。目标平台为原生 Android 应用，APK 直接分发。

## 核心约定

- **文件扩展名**：组件使用 `.uvue`，脚本使用 `.uts` — 这些不是标准的 Vue/TypeScript
- **语言**：UTS (UniApp TypeScript) 在 Android 上编译为 Kotlin，而非 JS。许多 TS/JS 特性不受支持。
- **IDE**：HBuilderX — 没有 CLI 构建/测试/lint 命令。通过 HBuilderX 构建和运行。
- **路由**：页面在 `pages.json` 中声明，不使用 Vue Router。第一个条目为启动页。
- **条件编译**：`// #ifdef APP-ANDROID` ... `// #endif` 是平台条件编译代码块。
- **全局 SCSS**：`uni.scss` 中的变量在所有组件中自动可用，无需 `@import`。
- **缩进**：Tab（见 `.editorconfig`）。
- **入口文件**：`main.uts` → `App.uvue`

## 常见错误

- 不要使用浏览器 DOM API — UTS 有受限的子集和原生运行时。
- 不要添加 `package.json` 或 npm 依赖 — 此项目没有 Node.js 工具链。
- 不要使用 Vue Router、Pinia 或 Vuex — 使用 uni-app x 内置的状态管理和导航（`uni.navigateTo` 等）。
- 不要编辑 `unpackage/` 下的文件 — 这是构建输出（已在 gitignore 中）。
- `ref()` 和 `reactive()` 可用，但类型必须兼容 UTS（不支持 `any`，泛型有限）。
- 模板语法类似 Vue，但组件名必须匹配已注册的 uni-app x 组件或自定义 `.uvue` 文件。
- CSS 不兼容：不支持 `100vh`（用 `flex: 1` 替代）、不支持 `text-transform`。

## UTS 编译错误清单（实踩经验）

以下错误均来自实际编译失败后的修复经验，避免重复踩坑。

### 类型系统
| 错误 | 原因 | 修复 |
|---|---|---|
| `参数类型不匹配：实际类型为 'Unit'，预期类型为 'Function0<Unit>'` | UTS 不支持函数引用传递（如 `setInterval(tick, 1000)`） | 用匿名函数包装：`setInterval(function () { tick() }, 1000)` |
| `Assignment type mismatch: actual type is 'Number', but 'Int' was expected` | `setInterval` / `setTimeout` 返回 `Number` 而非 `Int` | 声明为上界 `Number`：`let timer : Number = -1` |
| `Assignment type mismatch: actual type is 'Number', but 'Double' was expected` | `Math.cos()` / `Math.sin()` 返回 `Number` 而非 `Double` | 加 `.toDouble()` 显式转换：`Math.cos(angle).toDouble()` |
| `找不到名称'kVal'` | UTS 不支持 Kotlin 的 `val` 关键字 | 统一使用 `const` / `let` / `var`（JS 风格） |
| `参数类型不匹配：实际类型为 'Unit'，预期类型为 'Float'` | 数字字面量默认 `Double`，Android API 需要 `Float` | 加 `.toFloat()`：`20.0.toFloat()` |
| `dp2px` 参数类型 | `dp2px` 函数签名接受 `Double` | 调用时传 `dp2px(16.0)`，不要传 `Int` 字面量 |

### 闭包与作用域
| 错误 | 原因 | 修复 |
|---|---|---|
| `找不到名称"expandDoneTimer"` | UTS 闭包无法捕获同作用域内 `const` 声明的变量 | 改用模块级 `let` 变量，避免局部 `const` 自引用 |

### Java 接口实现
| 错误/模式 | 说明 | 正确写法 |
|---|---|---|
| SAM 接口包装 | UTS 不支持自动 SAM 转换，需显式包装接口 | `View.OnTouchListener(function (v, event) : Boolean { ... })` |
| 匿名内部类 | 不建议在 UTS 中使用 `new Interface({...})` 实现多方法接口 | 用 `setInterval` 定时器模拟动画结束回调，避免 `AnimatorListener` |
| `catch` 类型注解 | 不支持 `catch (e: any)` | 去掉类型：`catch (e) { }` |

### 原生 API 调用
| 错误/模式 | 说明 |
|---|---|
| `FLAG_NOT_TOUCHABLE` | 设置后窗口不接收任何触摸事件，如需拖拽/点击必须移除 |
| `WindowManager.LayoutParams` 坐标 | 使用 `Gravity.TOP \| Gravity.START` 时，`x` 和 `y` 是屏幕绝对坐标偏移 |
| `DisplayMetrics` 获取屏幕尺寸 | `new DisplayMetrics()` + `windowManager.getDefaultDisplay().getMetrics(dm)`，然后 `dm.widthPixels` / `dm.heightPixels` |

### 模块系统
| 错误/模式 | 说明 |
|---|---|
| 插件导入路径 | 页面导入 UTS 插件必须用**相对路径**（如 `../../utssdk/floating-clock/index`），`@/` 别名无法解析 |
| `package.json` 禁用 | `utssdk/` 目录下**不能放 `package.json`**，否则编译卡死无报错 |
| 嵌套类访问 | 导入 `ValueAnimator from 'android.animation.ValueAnimator'` 后，可通过 `ValueAnimator.AnimatorUpdateListener` 引用内部接口 |

## 文件结构

| 路径 | 用途 |
|---|---|
| `main.uts` | 应用入口，创建 SSR 应用实例 |
| `App.uvue` | 根组件，生命周期钩子 |
| `pages.json` | 页面路由和全局样式配置 |
| `manifest.json` | 应用元数据，平台设置 |
| `platformConfig.json` | 目标平台（当前为 APP-ANDROID） |
| `uni.scss` | 全局 SCSS 变量（自动导入） |
| `pages/` | 页面组件，每个页面一个文件夹 |
| `static/` | 静态资源（图片等） |

## 动画规范：Apple Fluid（苹果流畅风格）

## 动画核心原则
保持安静、连续且自然的高级感，避免任何突兀或机械的动画。

## 精确参数
- **缓动曲线**：所有过渡必须使用 `cubic-bezier(0.22, 1, 0.36, 1)` 以实现自然的减速感。
- **时长规范**：
  - 悬停反馈 (Hover)：160ms
  - 弹窗/模态框 (Modal)：320ms
  - 大场景切换：560ms
- **入场动画**：标题从下方 28px 处平滑弹入，卡片之间的入场间隔严格设置为 45ms。
- **交互反馈**：按钮悬停时向上抬起 1px，按下时缩放至 0.96。

## 无障碍与禁区
- **无障碍降级**：必须包含 `prefers-reduced-motion` 媒体查询，当用户开启减少动画时，回退到无动画或仅保留透明度渐变。
- **绝对禁区**：禁止使用弹跳效果、禁止元素抖动、禁止劫持页面滚动。


