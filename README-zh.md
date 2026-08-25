# DSH Notifier – 后台确认提醒插件

[English](README-en.md) | [简体中文](README.zh.md)

当 DeepSeek Harness 中的 AI Agent 需要用户确认、授权或询问时，如果 Harness 应用处于后台（最小化、失焦或标签页未激活），本插件会发送系统桌面通知，避免用户错过重要交互请求。

## 功能特性

- **增量 DOM 检测**：仅处理 DOM 变化部分，资源占用极低。
- **后台提醒**：仅当 Harness 页面不可见时才发送通知。
- **可配置**：支持自定义关键词、CSS 选择器和通知冷却时间。
- **跨平台**：基于标准 Web API，适用于桌面端（Windows/macOS/Linux）和 Android WebView。
- **防抖扫描**：合并频繁变化，避免重复通知。
- **一键聚焦**：通知包含点击操作，尝试聚焦 Harness 窗口。

## 文件结构
text
dsh-notifier/
├── package.json # 插件包元数据
├── src/
│ ├── host.js # 主进程插件入口（提供客户端脚本路由）
│ └── client.js # 渲染进程监测脚本（注入到 Web UI）
├── config.js # 默认配置（可选，可合并到 client.js）
└── README.md # 本文件
text


## 安装方法

1. 将插件文件夹放置到 Harness 的插件目录（或通过 `dsh plugin add` 安装）。
2. 确保插件被加载（重启 Harness 或重新加载插件）。
3. 首次使用时，浏览器可能会询问通知权限，请允许。
4. （可选）在 Harness 页面打开浏览器开发者工具，执行 `window.__dshNotifierSetConfig({...})` 自定义设置。

## 使用方法

- 正常使用 Harness，当出现需要确认的操作且应用在后台时，系统会弹出通知。
- 点击通知即可返回 Harness 窗口。
- 如需调整检测规则，可通过开发者工具控制台或编辑插件配置（见下文）。

## 配置说明

默认配置如下表所示，您可以在 Harness 页面的开发者控制台中通过 `window.__dshNotifierSetConfig()` 进行修改。

| 配置项 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `keywords` | string[] | `['确认','继续','是','允许','同意','确定','接受','下一步','提交']` | 按钮文本包含任一关键词时触发通知 |
| `selectors` | string[] | `[]` | 额外的 CSS 选择器，匹配的元素会触发通知 |
| `cooldown` | number | `10000` | 两次通知的最小间隔（毫秒） |
| `requireInteraction` | boolean | `true` | 通知是否常驻，直到用户手动关闭 |

修改配置示例：
```javascript
window.__dshNotifierSetConfig({
  keywords: ['确认', '允许'],
  cooldown: 5000,
  requireInteraction: false
})

工作原理

    增量 DOM 监听：使用 MutationObserver 监听 childList 和 attributes 变化，仅处理新增节点或属性变化的元素。

    确认元素检测：检查元素是否为按钮类（button、a、input[type=button]、[role=button]、.btn），并判断其文本是否包含关键词或匹配选择器。

    可见性判断：通过 document.hidden 和窗口焦点事件确定 Harness 是否在前台。

    系统通知：调用 Web Notification API 发送系统级通知，并处理点击聚焦。

已知限制

    脚本注入方式：本插件假设 Harness 会自动加载 /dsh-notifier/client.js 路由。若您的 DSH 版本使用不同的客户端插件注入机制，请相应调整 host.js。

    通知点击聚焦：由于 DSH 不暴露 Electron 窗口控制，点击通知仅调用 window.focus()，是否真正将窗口置顶取决于宿主实现。

    DOM 变化遗漏：极少数情况下，确认元素可能通过修改已有元素的文本而非新增节点或属性变化来显示，可能无法被捕获。官方若能提供“等待确认”事件将提升检测准确性。

贡献

欢迎提交 Issue 或 Pull Request 改进本插件。请确保代码风格一致，并在提交前进行测试。
许可证

MIT License
