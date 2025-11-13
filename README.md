# NeteaseMiniPlayer Halo 插件

一个在 Halo 博客中嵌入网易云音乐迷你播放器的插件方案。

---

## 功能特性

- 支持单曲与歌单两种来源（`data-song-id` / `data-playlist-id`）
- 自动/浅色/深色主题，以及系统主题联动与宿主主题检测
- 歌词显示、最小化模式、空闲淡出动画、播放模式切换（列表/单曲/随机）
- 内置缓存与错误提示、封面渲染、歌词滚动、高度压缩的 UI
- 保持原始 JS/CSS 完整，不进行任何改动

---

## 目录结构

```
src/main/resources/
├── plugin.yaml
├── extensions/
│   ├── reverse-proxy.yaml
│   └── settings.yaml
└── static/
    ├── css/
    │   └── netease-mini-player-v2.css
    └── js/
        ├── netease-mini-player-v2.js
        ├── nmp-web-component.js
        └── nmp-shortcode-parser.js
```

- `netease-mini-player-v2.js` 与 `netease-mini-player-v2.css`：原始播放器资源
- `nmp-web-component.js`：注册并实现 `<nmp-player>` 自定义元素
- `nmp-shortcode-parser.js`：将 `[netease-player ...]` 短代码转换为播放器容器并初始化
- `plugin.yaml`：插件注册与元信息
- `extensions/reverse-proxy.yaml`：静态资源暴露到 `/plugins/netease-mini-player-v2/**`
- `extensions/settings.yaml`：插件设置表单（FormKit Schema）

---

## 安装与启用

目前仓库为资源型插件结构，适合以 Halo 开发模式加载：

1. 在 Halo 配置中启用开发模式并指定固定插件路径（Windows 示例）：

```yaml
halo:
  plugin:
    runtime-mode: development
    fixed-plugin-path:
      - e:\\ProjectFiles\\NeteaseMiniPlayer-Halo
```

2. 重启 Halo 服务，进入 Console 插件管理，启用 `Netease Mini Player v2`。

3. 在插件详情页配置设置项（详见“设置说明”）。

说明：如需打包为 JAR 并以部署模式运行，请基于 Halo 官方插件模板引入后端入口类与构建脚本；本仓库现阶段不包含 Java/Gradle 构建配置。

---

## 资源路径

通过 ReverseProxy 暴露静态资源：

- CSS：`/plugins/netease-mini-player-v2/css/netease-mini-player-v2.css`
- JS：`/plugins/netease-mini-player-v2/js/netease-mini-player-v2.js`
- Web 组件封装：`/plugins/netease-mini-player-v2/js/nmp-web-component.js`
- 短代码解析：`/plugins/netease-mini-player-v2/js/nmp-shortcode-parser.js`

在页面（主题或自定义 HTML）中引入相应脚本即可使用。

---

## 三种用法

### 1) Web 组件（推荐）

引入 `nmp-web-component.js` 后，可直接书写：

```html
<script src="/plugins/netease-mini-player-v2/js/nmp-web-component.js" defer></script>

<nmp-player
  song-id="1901371647"
  theme="auto"
  lyric="true"
  size="compact"
  autoplay="false"
  position="static">
</nmp-player>
```

- 属性映射：
  - `song-id` -> `data-song-id`
  - `playlist-id` -> `data-playlist-id`
  - `theme` -> `data-theme`（`auto|light|dark`）
  - `lyric` -> `data-lyric`（`true|false`）
  - `size` -> `data-size`（`compact|...`）
  - `autoplay` -> `data-autoplay`（`true|false`）
  - `position` -> `data-position`（`static|top-left|top-right|bottom-left|bottom-right`）
  - `embed` -> `data-embed`（`true|false`）
  - `default-minimized` -> `data-default-minimized`（`true|false`）

`nmp-web-component.js` 会自动注入必要的 CSS/JS 并调用 `NeteaseMiniPlayer.initPlayer(...)` 完成实例化。

### 2) Markdown 短代码

引入 `nmp-shortcode-parser.js` 后，Markdown 中的短代码将被自动转换并初始化：

```html
<script src="/plugins/netease-mini-player-v2/js/nmp-shortcode-parser.js" defer></script>
```

示例：

```
[netease-player playlistId="14273792576" theme="auto" lyric="true"]

[netease-player songId="1901371647" theme="light" lyric="false"]
```

短代码将替换为：

```html
<div class="netease-mini-player"
     data-playlist-id="14273792576"
     data-theme="auto"
     data-lyric="true"></div>
```

随后调用 `window.NeteaseMiniPlayer.init()` 完成所有实例初始化。

### 3) 经典 HTML 容器

无需 Web 组件与短代码时，可直接在页面插入容器：

```html
<link rel="stylesheet" href="/plugins/netease-mini-player-v2/css/netease-mini-player-v2.css">
<script src="/plugins/netease-mini-player-v2/js/netease-mini-player-v2.js" defer></script>

<div class="netease-mini-player"
     data-song-id="1901371647"
     data-theme="auto"
     data-lyric="true"
     data-size="compact"
     data-autoplay="false"
     data-position="static"></div>
```

---

## 设置说明（Console）

`extensions/settings.yaml` 提供以下表单项：

- 基本设置：
  - `默认来源`：`song` / `playlist`
  - `默认ID`：字符串（歌曲或歌单）
  - `主题`：`auto` / `light` / `dark`
  - `显示歌词`：布尔
  - `默认最小化`：布尔
- 资源注入：
  - `注入CSS`：布尔
  - `注入JS`：布尔

说明：前端封装脚本默认会注入核心 CSS/JS 以保证即插即用；如需严格遵循上述开关，可在主题或页面脚本中读取插件设置（调用 Halo 插件 API）后按需加载。

---

## 初始化与 API

播放器在全局暴露为 `window.NeteaseMiniPlayer`：

- 自动初始化：`static init()` 会扫描 `.netease-mini-player` 容器并实例化（`src/main/resources/static/js/netease-mini-player-v2.js:1138-1159`）。
- 指定初始化：`static initPlayer(element)` 接受单个容器元素实例化（`src/main/resources/static/js/netease-mini-player-v2.js:1145-1150`）。
- 配置解析：从 `dataset` 读取配置（`parseConfig()`，`src/main/resources/static/js/netease-mini-player-v2.js:53-74`）。

示例（编程式）：

```js
import "/plugins/netease-mini-player-v2/js/netease-mini-player-v2.js";

const el = document.querySelector('.netease-mini-player');
const player = window.NeteaseMiniPlayer.initPlayer(el);
```

---

## 主题与样式

- 主题设置：`data-theme` 支持 `auto|light|dark`
- 自动检测：宿主类名与 `data-theme`、CSS 变量、系统主题联动（`detectTheme()` 系列方法）
- 样式来源：`src/main/resources/static/css/netease-mini-player-v2.css`

---

## 编辑器集成（规划）

- 富文本编辑器：基于扩展点 `default:editor:extension:create` 注册 Tiptap 扩展，提供工具箱入口、可视化表单与预览，一键插入播放器。
- Markdown 服务端渲染：在 Halo 的 Markdown 渲染链中（`@halo-dev/markdown-renderer`）注册插件，使短代码在服务端转换为最终 HTML，增强 SEO 与首屏性能。

上述前端 Console 集成代码将以 `src/main/resources/console/main.js` 或 `resources/ui` 形式提供（与 Halo 文档一致）。

---

## 常见问题

- 看不到播放器样式或交互？
  - 确认已正确引入 CSS/JS 或使用了 Web 组件/短代码脚本（它们会自动注入）。
  - 检查容器是否包含正确的 `data-*` 属性，如 `data-song-id` 或 `data-playlist-id`。
- 资源路径 404？
  - 确认插件已启用，检查 `/plugins/netease-mini-player-v2/**` 是否可访问。
  - 核对 `extensions/reverse-proxy.yaml` 配置是否存在。
- 主题不匹配？
  - 使用 `data-theme="auto"` 以自动适配；或明确指定 `light/dark`。

---

## 许可证

- Apache-2.0（详见 `plugin.yaml` 中的许可信息）
