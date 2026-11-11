# dsh-dabaitu — 大白兔外观包

一个插件管四件改动，只装一次、只卸载一次：

1. **左上角品牌**：肉包吉祥物图标 + 「大白兔白又白」文字。填的是
   `@deepseek-ai/dsh-client-ui-sidebar` 声明的槽位 `sidebar.brand.mark` /
   `sidebar.brand.name`；`cordis.patch.yml` 同时让官方品牌填充（`ui-brand-official`）退场。
   折叠成 28×28 侧栏时同一个吉祥物按 `size` 缩放，两处共用一个填充。
2. **护眼暖黄纸质主题**：用 `@deepseek-ai/dsh-client-ui-theme` 的
   `ctx.theme.overrideTokens()` 挂一层可撤销的 token 覆盖，把两组中性色阶
   `--dsw-static-neutral-*` / `--dsw-static-neutral-bluish-*` 从冷灰蓝换成暖褐；
   界面上的底色、侧栏、卡片、代码块、边框、文字、阴影都是这两条色阶上的别名，会一起变。
   另有一条 `html::after { mix-blend-mode: multiply }` 把 `assets/paper-grain.png`
   这张无缝颗粒贴图乘到整个界面上。色阶在宿主的浅色/深色规则里是同一套单调色阶，
   深色模式取高段，所以浅色是纸、深色自动是深色纸。
3. **外壳顶栏的「豆包」**：整个应用外壳的顶栏（Windows 上那条 40px 标题栏）**最靠右**的位置
   多一个「豆包」；不是会话自己的标题行，所以切换会话、滚动对话它都不动。点它调
   `ctx.sidebarRight.openTab('browser', { params: { url } })`——也就是在**应用内**
   右侧栏的 Browser tab 里打开 `https://www.doubao.com/chat/`，不是系统浏览器。
   挂的是 ui-layout 的 `AppFrame` 声明的 `shell.overlay`（`kind: 'list'`、root 作用域；
   全窗口浮层、本身 `pointer-events: none`，只有子元素接收指针，所以不挡应用）。
   这个槽位要求给 `id`，可选 `order`/`label`，本包用 `id: 'dabaitu-doubao'`、`order: 100`。
   位置：窗口最右上角那 138px 是 Windows 自己画的最小化/最大化/关闭按钮（Electron 的
   `titleBarOverlay`，高 40px），网页内容压在那下面就点不到，所以按钮从右边让开这一条，
   钉在窗口控制按钮的左边沿——这是网页能占的最右一格。让开的宽度由
   `measureCaptionControls` 填进 `--dabaitu-caption-controls`：实现了 Window Controls
   Overlay 的环境量真实几何，否则用 138px。
   `sidebarRight` 与 `sidebarRightTabs` 用 `ctx.get()` 可选取用（宿主 ui-chat 同样写法），
   不在 `inject` 里声明——声明了会在它们缺席时把整包挂起，纸纹与品牌跟着一起没。
   取不到时退成系统浏览器 `window.open`。
   （宿主没有「横跨整个框架的标题栏」槽位：`shell.leading` 只在抽屉或 macOS 收起侧栏时挂载，
   `root` 是单值槽位、占用会顶掉整个外壳，所以用的是 `shell.overlay` + 固定定位。）
4. **顶栏的「截图」**：紧挨在「豆包」左边的剪刀图标，**点一下直接开始截图**（没有选项菜单）：
   给当前界面拍一张快照 → 快照铺满屏幕当遮罩、拖动拉框选区（Esc 取消）→ 按选区裁图 →
   **复制到剪贴板**，并在底部冒一条带缩略图的小提示，可以点「保存」另存 PNG。
   - **为什么不用屏幕捕获**：宿主的 Electron 会话把媒体权限收得很死
     （`installMicrophonePermissions` 只放行麦克风），`display-capture` 会被直接拒掉，
     页面里既看不到选择框也不报错——这正是最初「点了没反应」的原因。
   - **快照怎么来**：把 `documentElement` 连同全部同源样式表克隆进
     `<foreignObject>`，再经 `<img>` 栅格化成画布。不依赖第三方库、不需要任何权限。
     SVG 命名空间的节点会先换成等尺寸占位（混进 `foreignObject` 会让整段解析失败）。
   - **截到的范围**：快照天然只含本应用界面，所以是一张纯净的应用截图。
   - **兜底**：若本次栅格化不可用（Chromium 拒画或画布被污染，会抽查中心像素判断），
     退回纯色底 + 正常拉框，选区尺寸与复制流程仍然可用（Console 会打一条 warn）。
   - 裁切按 `devicePixelRatio` 放大，高分屏下仍是清晰像素。

   平台限制：只截得到的范围是本应用界面——窗口背后的其它窗口本来就不在页面里。
   若要连同桌面一起截，需要宿主暴露 `desktopCapturer`/`capturePage` 之类的通道（当前没有）。

品牌蓝与红/绿/琥珀语义色不变；只有当背景用的 `deepseek-50/100/200`（用户气泡等）
跟着纸走，比页面底色略深，气泡仍可分辨。选中文本改成琥珀色高亮。

全部改动只在运行时生效：卸载本包即恢复官方品牌与原配色、顶栏也不再有「截图」和「豆包」。

## 插件列表里的图标与名字

插件自己带自定义图标与文字：`package.json` 顶层 `"icon": "./icon.svg"`（同一张肉包吉祥物，
包在 128×76 的 SVG 里），显示名与说明放在 `locale/zh.json` / `locale/en.json` 的 `meta`
（标题「大白兔白又白」）。这是宿主插件列表读取的两处元数据，改这两处就改插件卡片；
`icon.svg` 由 `assets/mark-128.png` 生成，重新抠图见 `../.tools/cut_mark.py`。
插件卡片的名字与图标在应用重启或插件页刷新后才换新。

侧栏那两个槽位在宿主目录里是 `kind: 'single'`、`registerOptions` 为空，所以注册只给
`name`，不塞 `id`/`key`/`label`——与官方填充和 LiveShare 的写法一致。`shell.overlay`
那个列表槽位相反，`id` 是必填的；`plugins.row.config` 是 keyed 槽位，
key 必须是 `<包名>#<行 id>`（本包是 `@local/dsh-dabaitu#dabaitu-look`）。

## 在插件设置里换图标和文字

打开插件页里本包那一页，头部就有「**自定义品牌…**」按钮（`plugins.detail.actions`，
只在 subject 指向本包时出现），点开是一个自带的模态框；表单里可以**上传图标**
（PNG / SVG / JPG，上限 2 MB，读成 data URI）和**修改左上角文字**（最长 40 字），
点「保存」立刻生效，另有「恢复默认」。

同时也注册了行配置入口（`plugins.row.config`，key `@local/dsh-dabaitu#dabaitu-look`），
本包处于启用状态时那一行会多出配置按钮。两条路打开的是同一个表单——
头部按钮是主入口，因为它不依赖 keyed 槽位的账本时序。

- 存储：`localStorage`，键名 `@local/dsh-dabaitu:brand`（`BRAND_KEY`），值是
  `{ mark, name }`，`mark` 是 data URI。刷新、重启都还在；换浏览器 profile 不跟过去。
- 生效方式：`BunMark` / `BunName` 通过 `useBrand()` 订阅一个极小的 `brandListeners`
  订阅器，`writeBrand` 写完就通知，所以侧栏两个槽位立刻重渲染，不用重载页面。
- 只管**侧栏左上角**的品牌。插件列表卡片上的图标与名字仍是 `package.json` 的 `icon`
  和 `locale/*.json` 的 `meta`，运行时改不了（宿主在构建期读它们）。
- 想清掉自定义：表单里点「恢复默认」，或手动删掉那条 localStorage 记录。

## 只想要其中几件

- 只保留品牌：删掉 `apply()` 里第 2、3、4 段与 `actionStyle`，以及 `inject` 里的 `'theme'`
  和 `package.json` 的 `@deepseek-ai/dsh-client-ui-theme`。
- 只保留主题：删掉 `apply()` 里第 1 段与两个品牌组件，以及 `cordis.patch.yml` 里
  让 `ui-brand-official` 退场的那三行（`inject` 里的 `'slots'` 只在还想留豆包时保留）。
- 只去掉豆包：删掉 `apply()` 里第 4 段两段（注册 + 那条 `<style>`）、`makeDoubaoAction`、
  `actionStyle` 里的 `.dabaitu-header-action` 规则与 `DOUBAO_TEXT` / `DOUBAO_URL`。
- 只去掉截图：删掉 `apply()` 里第 5 段、`ShotAction`、`runScreenshot` 及其下面的
  `grabScreenFrame` / `pickRegion` / `cropShot` / `canvasBlob` / `copyCanvas` /
  `showShotToast` / `withWindowHidden` / `findBrowserFrame`，以及 `SCRATCH_STYLE`。

## 微调

- 纸的深浅：改 `--dsw-static-neutral-bluish-00` 与相邻几档。
- 纸纹强弱：`client.js` 里叠加层的 `opacity`（当前 .45）。
- 想要原来的蓝色用户气泡：删掉 `PAPER_TOKENS` 里 `--dsw-static-deepseek-50/100/200` 三行。
- 文字/图标：`NAME_TEXT`、`assets/mark-128.png`（重新生成见 `../.tools/cut_mark.py`）。
- 豆包的文字与地址：`DOUBAO_TEXT`、`DOUBAO_URL`；贴边的距离改 `CAPTION_EDGE_GAP`
  （`--dabaitu-caption-gap`，默认 8px），给窗口控制按钮让出的宽度改
  `CAPTION_CONTROLS_WIDTH`（`--dabaitu-caption-controls`，默认 138px）；
  `order` 只决定和其它浮层条目的先后，不影响屏幕位置。

调色与贴图由 `../.tools/make_paper_theme.py` 生成，`../.tools/make_look_pack.py` 组装本包；
`../.tools/check_tokens.py` 核对这里覆盖的每个 token 名确实存在于宿主样式表。
