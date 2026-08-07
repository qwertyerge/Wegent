# QA 记录:移除 Wework 桌面端 native surface inset(窗口级截图验证)

验证方式:真实 Tauri 应用(`ai:verify` 隔离会话)+ macOS 窗口级捕获(`screencapture -l`),前后同一窗口位置 1280×720。

## 会话信息

| 会话 | 提交 | 代码来源 | App PID | 窗口 ID |
| --- | --- | --- | --- | --- |
| before | `e15996de2`(父提交,改动前) | worktree `/private/tmp/wegent-before` | 98392 | 75799 |
| after | `ef7cf6b8b`(改动后) | worktree `/private/tmp/wegent-contrib` | 9023 | 75904 |

before 会话: `test-results/ai-verify/2026-08-07T04-41-37-682Z-98100`
after 会话: `test-results/ai-verify/2026-08-07T04-45-56-515Z-8698`

## 焦点状态说明

macOS 窗口聚焦时头部(标题栏/标签栏)与主区容器底色存在色差,内层卡片边距/圆角明显;失焦时底色一致,前后看起来都协调。`screencapture -l` 按聚焦外观渲染窗口,因此本记录所有截图渲染状态一致,可直接对比。

## 截图清单(均为窗口级,含 macOS 原生标题栏/交通灯)

- `before/workbench-home-before.png` — before 会话工作台首页
- `before/plugins-before.png` — before 会话插件页
- `after/workbench-home-after.png` — after 会话工作台首页
- `after/plugins-after.png` — after 会话插件页
- `comparison/workbench-home-before-after.png` — 首页并排对比
- `comparison/plugins-before-after.png` — 插件页并排对比
- `comparison/workbench-home-left-edge-zoom.png` / `workbench-home-bottom-edge-zoom.png` — 首页左缘/底缘 4× 放大(Before 红框标注内边距带)
- `comparison/plugins-left-edge-zoom.png` / `plugins-bottom-edge-zoom.png` — 插件页左缘/底缘 4× 放大

## 像素采样(图片坐标,窗口框内 1280×720,偏移左 34 / 上 34)

| 探针 | before-home | after-home | before-plugins | after-plugins |
| --- | --- | --- | --- | --- |
| 左缘内 3px | rgb(228,228,227) 灰带 | rgb(244,244,244) 侧栏贴边 | rgb(228,228,227) 灰带 | rgb(247,247,247) 贴边 |
| 右缘内 3px | rgb(229,229,228) 灰带 | rgb(255,255,255) 内容贴边 | rgb(229,229,228) 灰带 | rgb(255,255,255) 内容贴边 |
| 底缘 -12px | rgb(227,227,226) 边框线 | rgb(248,248,248) 内容 | rgb(227,227,226) 边框线 | rgb(255,255,255) 内容 |
| 底缘 -24px | rgb(245,245,245) | rgb(255,255,255) | rgb(255,255,255) | rgb(255,255,255) |

边缘扫描(before,首页):左缘 x=36..41 为 rgb(226..229) 灰色边距带,卡片从 x=42 开始;右缘 x=1306..1313 为灰带;底缘 y=737..745 为灰带 + 边框线。after:对应位置为内容底色(侧栏 244 / 白色 255),无灰带与边框线。

结论:before 内容四周存在约 8px 内层边距 + 卡片边框线;after 全出血。四角残留圆角为 macOS 窗口自身形状。

## 验证命令

- `pnpm --filter wework ai:verify start` / `navigate --value /plugins` / `wait-for --selector [data-testid="desktop-workbench-surface"]`、`[data-testid="desktop-auxiliary-surface"]`
- `screencapture -x -l <windowID> <png>` 窗口级捕获
- 单元测试:307 files / 2992 tests 全部通过;`tsc -b`、eslint、prettier 通过(提交质量门禁)
