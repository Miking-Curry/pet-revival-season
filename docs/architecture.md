# 🐾 Pet Revival Season

> 一只住在你桌面上的小伙伴——轻量、有感情、不烦人。

---

## 🧱 技术栈

| 层 | 选型 | 理由 |
|---|---|---|
| 桌面框架 | **Tauri 2.x** | 比 Electron 轻一个数量级，Rust 后端适合跑状态机，资源占用控制在 50MB 内 |
| 渲染 | **Webview2 + Canvas/HTML** | 系统 webview 够用，Canvas 2D 播动画帧不需要 WebGL |
| 状态管理 | **Rust 行为树** | 待机、溜达、被打断、恢复，行为树天然支持优先级和中断恢复 |
| 存档 | **JSON → SQLite** | MVP 阶段 JSON 即可，加打工/离线结算后迁 SQLite |
| 资源 | **嵌入式 + 按需加载** | 动画帧 PNG 序列打包进 `assets` 目录，运行时只加载当前行为所需 |

> **MVP 简化原则：** 不熟行为树就用简单事件调度器 + 状态枚举，二期再重构。**先活再精。**

---

## 📁 目录结构

```
pet-revival-season/
├── src-tauri/               # Rust 后端
│   ├── src/
│   │   ├── main.rs
│   │   ├── state/           # 状态机 / 行为树
│   │   │   ├── mod.rs
│   │   │   ├── behavior.rs
│   │   │   └── scheduler.rs
│   │   ├── storage/         # 存档
│   │   └── ipc/             # 前后端通信
│   └── Cargo.toml
├── src/                     # 前端渲染层
│   ├── index.html
│   ├── styles/
│   ├── scripts/
│   │   ├── renderer.js
│   │   ├── behavior-bridge.js
│   │   └── input-handler.js
│   └── assets/
│       ├── sprites/
│       │   ├── idle/
│       │   ├── walk/
│       │   └── react/
│       └── sounds/
├── docs/
│   ├── mvp.md
│   └── architecture.md
└── README.md
```

---

## 🔀 架构：意图层 & 动画层解耦

```
[行为调度器 / 行为树]    ← Rust 后端，产出行为指令
         │
         ▼
  { action: "idle", variant: "look_around" }
         │
         ▼
[动画选择器]             ← 前端，选帧序列播
```

- **新加动作**：意图层注册行为 + 动画层放入帧资源，管道不动
- **换装系统**：直接换资源路径即可

---

## ⚠️ 窗口穿透关键验证

MVP 阶段必须验证以下三个 API：

```rust
window.set_always_on_bottom()         // 置底
window.set_ignore_cursor_events(true) // 点击穿透（拖拽时临时关）
window.set_skip_taskbar(true)         // 任务栏隐身
```

> **回退方案：** 若任一 API 走不通，降级为通知栏图标 + 弹出面板。体验及格，但不算真正的桌面挂件。

---

## ✅ MVP 行为优先级

完成前 4 项即可闭环，「活着」的感觉就到位了。

| 优先级 | 行为 | 描述 |
|:---:|---|---|
| 🥇 1 | **待机** | 站着偶尔眨眼 |
| 🥇 2 | **溜达** | 随机方向移动，到边缘折返 |
| 🥇 3 | **拖拽** | 被拖走，松手后生闷气 |
| 🥇 4 | **反应** | 点击反馈 |
| ⬜ 5 | 自发动作 | 打哈欠、看时间、趴边缘、发呆，低频随机触发 |
| ⬜ 6 | 投喂 | 拖食物，心情变化 |
| ⬜ 7 | 离线结算 | 下次启动更新状态 |
