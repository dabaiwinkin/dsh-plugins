# @local/dsh-brand-dabaitu

把 DeepSeek Harness Web 客户端左上角（侧栏品牌行）的图标与名称换成本地品牌：

- 图标：肉包吉祥物（`assets/mark-128.png`，已抠出白底与「肉包」字样，内联进 `client.js`）
- 名称文字：`大白兔白又白`

实现方式是占据官方品牌包留出的两个 single slot：`sidebar.brand.mark` 与
`sidebar.brand.name`；同时在 `cordis.patch.yml` 里停用官方品牌填充行 `ui-brand-official`。
删除本组合包后，官方鱼形标志与名称会自动恢复。

改文字：编辑 `client.js` 里的 `NAME_TEXT`。改图标：替换 `assets/mark-128.png` 后重新生成
`client.js` 中的 `MARK_SRC`（或直接把图片放到别处并以宿主可访问的 URL 引用）。
