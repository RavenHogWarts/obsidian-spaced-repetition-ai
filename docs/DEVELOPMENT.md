# Obsidian Spaced Repetition AI 开发指南

欢迎来到 Obsidian Spaced Repetition AI (SRAI) 插件的开发文档！这份文档旨在帮助刚刚接触本项目（甚至刚接触 Obsidian 插件开发）的开发者快速上手。

## 1. 项目概述

**Spaced Repetition AI (SRAI)** 是一个 Obsidian 插件，旨在通过 AI 生成闪卡并利用 FSRS (Free Spaced Repetition Scheduler) 算法帮助用户进行间隔重复复习。

核心功能包括：
*   **AI 辅助生成**: 利用 OpenAI (GPT-3.5/4) 从笔记内容生成闪卡。
*   **间隔重复复习**: 基于 FSRS 算法安排复习计划，类似于 Anki。
*   **闪卡管理**: 在 Obsidian 内部查阅和管理闪卡。

## 2. 快速开始 (Quick Start)

### 环境准备

在开始之前，请确保你的电脑上安装了：
*   **Node.js**: 推荐 v16+ (你可以通过终端输入 `node -v` 检查)。
*   **npm** 或 **pnpm**: Node.js 包管理工具。
*   **Obsidian**: 用于测试插件。

### 安装依赖

打开终端（Terminal），进入项目根目录：

```bash
cd e:\WorkSpace\GitProgram\Contribute\obsidian-spaced-repetition-ai
npm install
```

### 启动开发模式

开发模式会监听文件变化并自动重新编译。

```bash
npm run dev
```

此命令会启动 `esbuild` 处于 watch 模式。编译后的 `main.js`, `main.css`, `manifest.json` 会生成在项目根目录。

**如何在 Obsidian 中加载插件？**
Obsidian 插件通常需要位于 `<你的Vault>/.obsidian/plugins/<插件ID>/` 目录下。
你可以手动将生成的 `main.js`, `manifest.json`, `styles.css` 复制到你的测试 Vault 中，或者使用符号链接 (Symbolic Link)。

> **提示**: 安装 [Hot Reload](https://github.com/pjeby/hot-reload) 插件可以让你在修改代码后自动刷新 Obsidian 里的插件，无需手动重启。

## 3. 项目结构说明

项目源代码主要集中在 `src` 文件夹中。以下是核心目录结构的解析：

```
src/
├── components/       # React UI 组件
│   ├── chat/         # 聊天界面相关组件 (AI 对话)
│   ├── review/       # 复习界面相关组件 (闪卡展示)
│   ├── onboarding/   # 引导页组件
│   ├── Chat.tsx      # AI 聊天主组件
│   ├── Review.tsx    # 复习主组件
│   └── ...
├── fsrs/             # 核心算法与数据模型
│   ├── algorithm.ts  # FSRS 调度算法实现
│   ├── Deck.ts       # 牌组 (Deck) 管理逻辑
│   ├── models.ts     # 核心数据接口定义 (Card, ReviewLog 等)
│   └── ...
├── LLM/              # 大语言模型交互
│   └── AIManager.ts  # 管理 OpenAI API 调用、Prompt 构建
├── memory/           # 数据持久化层
│   └── memoryManager.ts # 负责读写 JSON 数据文件 (存储复习进度)
├── views/            # Obsidian 视图容器
│   └── MainView.tsx  # 插件主界面容器 (挂载 React 应用)
├── utils/            # 工具函数
├── hooks/            # 自定义 React Hooks
├── constant.ts       # 全局常量 (API 模型列表, 配置默认值等)
├── main.ts           # 插件入口文件 (类似程序的 main 函数)
└── settings.ts       # 设置页面定义
```

## 4. 核心架构解析

### 4.1 插件入口 (`main.ts`)
这是 Obsidian 插件的起点。它继承自 `Plugin` 类。
*   `onload()`: 插件加载时执行。在这里初始化 `MemoryManager`, `DeckManager`, `AIManager`，注册视图 (`MainView`)，添加命令和设置页。
*   `saveSettings()`: 保存用户配置。

### 4.2 视图层 (React in Obsidian)
本项目使用 **React** 来构建 UI。
*   `MainView.tsx`: 是连接 Obsidian API 和 React 组件的桥梁。它创建一个 React Root 并渲染 `NavBar` 以及 `Chat` 或 `Review` 子视图。
*   **Chat 视图**: 用户与 AI 对话，生成闪卡。
*   **Review 视图**: 用户进行闪卡复习。

### 4.3 数据存储 (`memory/`)
插件不使用传统的数据库，而是将数据存储在 Vault（你的笔记库）中的 `SR` 文件夹下的 JSON 文件里。
*   每个笔记文件对应的闪卡数据可能会被单独存储或统一管理（由 `MemoryManager` 处理）。
*   避免直接手动修改这些 JSON 文件，虽然它们是文本格式，但格式错误可能导致数据丢失。

### 4.4 算法层 (`fsrs/`)
这是核心业务逻辑。
*   `Card`: 代表一张闪卡，包含难度、上次复习时间、下次复习时间等状态。
*   `FSRS`: 调度器，根据用户的评分（忘记、困难、良好、简单）计算下一次复习时间。

## 5. 开发常见任务指南

### 任务一：修改 UI 样式
本项目使用 **Tailwind CSS**。
*   你可以直接在 React 组件的 `className` 中修改样式类。
*   样式定义文件在 `src/tailwind.css`。
*   `postcss.config.js` 和 `tailwind.config.js` 控制 CSS 的构建。

### 任务二：增加新的 AI 模型支持
如果你想添加新的模型（例如 GPT-5）：
1.  打开 `src/constants.ts`。
2.  在 `ChatModels` 枚举中添加新模型 ID。
3.  在 `ChatModelDisplayNames` 枚举中添加显示名称。
4.  更新 `DISPLAY_NAME_TO_MODEL` 映射。

### 任务三：调试
*   **Console.log**: 最简单的调试方法。日志会输出到 Obsidian 的开发者工具控制台（可以通过 `Ctrl+Shift+I` 打开）。
*   **React DevTools**: 在 Obsidian 中调试 React 组件比较麻烦，建议多利用日志。

### 任务四：构建发布
当你准备好发布时，运行：
```bash
npm run build
```
这会生成压缩优化过的 `main.js` 和 `styles.css`。

## 6. 注意事项

1.  **TypeScript**: 项目全量使用 TypeScript，请务必定义好类型，尽量避免使用 `any`。这有助于代码维护。
2.  **异步操作**: 文件读写（Obsidian API）和网络请求（OpenAI API）都是异步的，记得使用 `async/await`。
3.  **安全性**: 永远不要将 API Key 硬编码在代码里！使用 `settings` 从用户配置中读取。

祝你开发愉快！如果有任何问题，欢迎在 GitHub Issues 中提出。
