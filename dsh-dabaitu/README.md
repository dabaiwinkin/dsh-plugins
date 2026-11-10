# dsh-dabaitu — 大白兔外观包

一个插件管两件外观改动，只装一次、只卸载一次：

1. **左上角品牌**：肉包吉祥物图标 + 「大白兔白又白」文字。填的是
   `@deepseek-ai/dsh-client-ui-sidebar` 声明的槽位 `sidebar.brand.mark` /
   `sidebar.brand.name`；`cordis.patch.yml` 同时让官方品牌填充（`ui-brand-official`）退场。
   折叠成 28×28 侧栏时同一个吉祥物按 `size` 缩放，两处共用一个填充。
2. **护眼暖黄纸质主题**：用 `@deepseek-ai/dsh-client-ui-theme` 的
   `ctx.theme.overrideTokens()` 挂一层可撤销的 token 覆盖，把两组中性色阶
   `--dsw-static-neutral-*` / `--dsw-static-neutral-bluish-*` 从冷灰蓝换成暖褐；
   界面上的底色、侧栏、卡片、代码块、边框、文字、阴影都是这两组色阶上的别名，会一起变。
   另有一条 `html::after { mix-blend-mode: multiply }` 把 `assets/paper-grain.png`
   这张无缝颗粒贴图乘到整个界面上。色阶在宿主的浅色/深色规则里是同一套单调色阶，
   深色模式取高段，所以浅色是纸、深色自动是深色纸。

品牌蓝与红/绿/琥珀语义色不变；只有当背景用的 `deepseek-50/100/200`（用户气泡等）
跟着纸走，比页面底色略深，气泡仍可分辨。选中文本改成琥珀色高亮。

全部改动只在运行时生效：卸载本包即恢复官方品牌与原配色。

## 插件列表里的图标与名字

插件自己带自定义图标与文字：`package.json` 顶层 `"icon": "./icon.svg"`（同一张肉包吉祥物，
包在 128×76 的 SVG 里），显示名与说明放在 `locale/zh.json` / `locale/en.json` 的 `meta`
（标题「大白兔白又白」）。这是宿主插件列表读取的两处元数据，改这两处就改插件卡片；
`icon.svg` 由 `assets/mark-128.png` 生成，重新抠图见 `../.tools/cut_mark.py`。

侧栏那两个槽位在宿主目录里是 `kind: 'single'`、`registerOptions` 为空，所以注册只给
`name`，不塞 `id`/`key`/`label`——与官方填充和 LiveShare 的写法一致。

## 只想要其中一件

- 只保留品牌：删掉 `apply()` 里第 2、3 段，以及 `inject` 里的 `'theme'` 和
  `package.json` 的 `@deepseek-ai/dsh-client-ui-theme`。
- 只保留主题：删掉 `apply()` 里第 1 段与两个品牌组件，以及 `cordis.patch.yml` 里
  让 `ui-brand-official` 退场的那三行（`inject` 里的 `'slots'` 也可去掉）。

## 微调

- 纸的深浅：改 `--dsw-static-neutral-bluish-00` 与相邻几档。
- 纸纹强弱：`client.js` 里叠加层的 `opacity`（当前 .45）。
- 想要原来的蓝色用户气泡：删掉 `PAPER_TOKENS` 里 `--dsw-static-deepseek-50/100/200` 三行。
- 文字/图标：`NAME_TEXT`、`assets/mark-128.png`（重新生成见 `../.tools/cut_mark.py`）。

调色与贴图由 `../.tools/make_paper_theme.py` 生成，`../.tools/make_look_pack.py` 组装本包；
`../.tools/check_tokens.py` 核对这里覆盖的每个 token 名确实存在于宿主样式表。
