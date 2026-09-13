# 歌词 / 封面内嵌工具

把单独的 `.lrc` 歌词文件写进 FLAC / MP3 的标签里，播放器就能直接显示歌词；
还能联网给**缺封面**的歌自动补上封面。
写入过程只增加/替换标签，**音频数据逐字节不变**。

两种用法，同一个内核（`core.mjs`）：

| | 桌面程序（推荐） | 命令行 |
| --- | --- | --- |
| 入口 | `歌词内嵌工具\歌词内嵌工具.exe` | `embed-lyrics.mjs` / `fetch-covers.mjs` |
| 适合 | 鼠标操作、拖文件夹、看进度 | 批量脚本、自动化 |

---

## 一、桌面程序

**双击 `歌词内嵌工具\歌词内嵌工具.exe`**（或根目录下带图标的 `歌词内嵌工具.lnk`）。

1. 选音乐文件夹（FLAC/MP3）和歌词文件夹（.lrc）——可以是同一个文件夹，也可以直接把文件夹拖进窗口。
2. 点「开始扫描」，自动按文件名配对：
   - 忽略 `_EM` / `_EG` / `_SQ` 等后缀
   - 忽略大小写、空格、全角半角括号、`-` 与 `–` 的差异
   - 歌词是 GBK/GB18030 编码会自动转成 UTF-8（QQ音乐导出的歌词正是这个编码）
3. 勾选歌曲，选「就地覆盖」或「输出到指定目录」，点「开始内嵌」。
4. 日志逐首显示结果，包含每首歌的**音频无损校验**结果。

便携版：整个 `歌词内嵌工具` 文件夹可以拷到别的 Windows 10/11 64 位电脑直接用，不写注册表、不需要管理员权限，删掉即卸载。
首次运行会在 `%APPDATA%\歌词内嵌工具` 生成几 MB 的界面缓存（这类程序的通行做法），可随时删除。

### 分享给别人

- **必须整个文件夹一起给**（先右键压缩成 zip 再发），只发 exe 是跑不起来的，它依赖同目录的 `resources` 和各 dll。
- **不要单独发 `歌词内嵌工具.lnk`**：快捷方式里存的是绝对路径（`D:\space\...`），换台电脑就失效。让对方自己右键 exe →「发送到 → 桌面快捷方式」。
- 对方首次运行若弹出「Windows 已保护你的电脑」，那是 SmartScreen 对未签名程序的提示，点「更多信息」→「仍要运行」。
- 仅支持 64 位 Windows 10/11（包里是 win32-x64 运行时）。

## 二、联网补封面与歌曲信息

**规则：已有的不动，缺的才补。**
封面和信息是**分开判断**的——已经有封面但标签为空的文件，照样会把标签补上。

### 桌面程序

第 4 个卡片「封面」：`扫描歌曲` →（可选：挑数据源）→ `联网查找封面` → 看一眼缩略图 → `写入选中的封面`。

三个开关：
- **写入封面**（默认开）
- **同时写入歌曲信息**（默认开）：歌名 / 歌手 / 专辑 / 年份 / 风格 / 音轨
- **连已有的歌曲信息也覆盖**（默认关）：默认只填空字段

高/中置信度默认勾选，低置信度默认不勾选；点缩略图可看大图；
点「换一个」可以手动改选其他候选。

### 命令行

```powershell
cd D:\space\lyric-tools

# 预演：列出每首歌的现状、命中的候选、将写入的字段，不动文件
& D:\node\node.exe fetch-covers.mjs -a "D:\Music" -v

# 确认后写入（封面 + 歌曲信息，只补空字段）
& D:\node\node.exe fetch-covers.mjs -a "D:\Music" --write

# 只补歌曲信息、不写封面
& D:\node\node.exe fetch-covers.mjs -a "D:\Music" --write --no-cover

# 强制覆盖已有的歌曲信息
& D:\node\node.exe fetch-covers.mjs -a "D:\Music" --write --overwrite-tags

# 指定数据源
& D:\node\node.exe fetch-covers.mjs -a "D:\Music" -s qqmusic --write
& D:\node\node.exe fetch-covers.mjs --list-sources
```

### 数据源（可手动选择）

界面上点「数据源」那一排标签即可切换：**全自动**，或单独勾选一个/多个。

| id | 名称 | 说明 | 本机实测 |
| --- | --- | --- | --- |
| `qqmusic` | QQ音乐 | 中文歌覆盖最好，封面 800×800 | 10 首中文歌全部精确命中 |
| `itunes` | iTunes | 欧美歌最好；会查台/港区以拿到中文歌手名 | 欧美歌准确 |
| `netease` | 网易云 | 中文歌备选 | 可用 |
| `deezer` | Deezer | 欧美歌备选 | 本机网络不可达，自动跳过 |
| `musicbrainz` | MusicBrainz | 开源兜底，索引较慢 | 可用 |

全部是公开 HTTPS 接口，**不需要安装 iTunes 或任何软件，不需要账号或密钥**。
全自动模式下按上表顺序依次尝试，**命中足够多高置信度候选就提前停止**，
所以中文歌通常只查一个源就结束。

### 一键换封面

每一行都有「换一个」按钮，展开后列出所有备选（带缩略图、来源、置信度、时长差），
点哪个就用哪个；选中的会在写入时生效并标注「（手动选择）」。

### 匹配规则

匹配综合三项：歌名相似度、歌手相似度、**音频时长**。
时长是很硬的证据（FLAC 的 STREAMINFO 里存的是精确值），能识破 Live 版、伴奏版、翻唱版。

| 置信度 | 条件 | 默认行为 |
| --- | --- | --- |
| 高 | 歌手对得上，或歌名 + 时长双双吻合（±2 秒） | 自动勾选 |
| 中 | 时长差 ≤6 秒，或歌手部分吻合 | 自动勾选 |
| 低 | 歌名对但歌手完全对不上、时长也不明（多半是翻唱） | **不勾选** |
| 否决 | 歌名不像，或时长差 >20 秒 | 直接丢弃 |

另外做了两个归一化：繁简字按单字比对（`海闊天空` / `海阔天空` 能得 0.75，
不需要完整繁简对照表），版本后缀（`(Live)`、`(伴奏)`、`feat.` 等）会被剔除再比较。

> 之前那个已知局限——中文歌手名与英文歌手名算不出相似度（`周传雄` vs `Steve Chou`）——
> 现在靠 QQ音乐/网易云的中文数据，以及 iTunes 台/港区，基本已经绕开了。

## 三、命令行（歌词）

```powershell
cd D:\space\lyric-tools

# 预演：只列出会做什么，不动任何文件
& D:\node\node.exe embed-lyrics.mjs -a "音频目录" -l "歌词目录"

# 确认后写入（就地覆盖）
& D:\node\node.exe embed-lyrics.mjs -a "音频目录" -l "歌词目录" --write

# 输出到别处，原文件不动
& D:\node\node.exe embed-lyrics.mjs -a "音频目录" -l "歌词目录" -o "D:\音乐_带歌词" --write

# 校验：有没有写入成功 + 音频是否无损（对比原始目录）
& D:\node\node.exe check-lyrics.mjs -a "D:\音乐_带歌词" --vs "音频目录"

# 打印全部标签，确认标题/歌手/封面没丢
& D:\node\node.exe check-lyrics.mjs -a "D:\音乐_带歌词" --tags
```

`embed-lyrics.mjs` 选项：`-a/--audio`、`-l/--lrc`（均可多次指定）、`-o/--out-dir`、`--write`、
`--fields LYRICS,UNSYNCEDLYRICS`、`--keep-tags`、`--lang chi`、`-r/--recursive`、`-v/--verbose`。

## 四、技术细节

- **FLAC**：重建 `VORBIS_COMMENT` 块写入 `LYRICS` 和 `TITLE`/`ARTIST`/`ALBUM`/`DATE`/`GENRE`/`TRACKNUMBER`；
  封面写进 `PICTURE` 块。原有标签全部保留，音频帧原样搬运。
- **MP3**：保留原有 ID3v2 帧，替换 `USLT` 歌词帧和 `TIT2`/`TPE1`/`TALB`/`TDRC`(`TYER`)/`TCON`/`TRCK`；
  封面写进 `APIC` 帧。中文文本按 v2.3 用 UTF-16、v2.4 用 UTF-8，避免乱码。
- **一次读写**：封面和歌曲信息在同一次读-改-写里完成，不会为了两件事复制两遍音频。
- **编码**：歌词自动识别 UTF-8 / UTF-8 BOM / UTF-16 / GB18030(GBK)，统一转 UTF-8 后写入。
- **读取健壮性**：`fs.read` 会短读（遇到几十 KB 的封面块时尤其明显），所有读取都循环读满，
  避免把零填充当成真数据、或误报「文件被截断」。
- **安全**：先写临时文件再改名，中途失败不会破坏原文件；CLI 默认预演，不加 `--write` 绝不写盘。
- **无损**：每次写入前后都对音频区域做 SHA-256 比对，桌面程序逐首显示结果。

## 五、目录结构

```
lyric-tools/
├─ 歌词内嵌工具/            便携版桌面程序（367 MB，含 Electron 运行时）
│  ├─ 歌词内嵌工具.exe
│  ├─ 使用说明.txt
│  └─ resources/app/       应用代码（core.mjs + cover-fetch.mjs + main.js + preload.js + ui/）
├─ 歌词内嵌工具.zip         打包好的单文件，发人用这个
├─ 歌词内嵌工具.lnk         带图标的快捷方式（只在你自己电脑上有效）
├─ core.mjs                 共享内核：配对、编码识别、歌词/封面读写、时长解析
├─ cover-fetch.mjs          封面抓取：文件名解析、归一化、多源查询、打分
├─ embed-lyrics.mjs         命令行：内嵌歌词
├─ fetch-covers.mjs         命令行：抓取封面
├─ check-lyrics.mjs         命令行：校验歌词与无损
└─ desktop/                 桌面版源码与构建脚本
   ├─ main.js               主进程（IPC、文件对话框、批量处理）
   ├─ preload.js            安全桥（contextBridge）
   ├─ ui/                   界面（原生 HTML/CSS/JS，无框架）
   ├─ assets/icon.png|ico   图标
   ├─ diagnose-cover.mjs    封面/标签体检工具
   ├─ get-electron.mjs      下载 Electron 运行时并校验 SHA-256
   ├─ unzip.mjs             自带的 ZIP/ZIP64 解包器
   ├─ make-icon.mjs         图标生成器
   ├─ build-portable.mjs    组装便携版
   └─ check-ui.mjs          界面静态一致性检查
```

重新构建便携版：

```powershell
& D:\node\node.exe desktop\build-portable.mjs
```

程序内置自检（不需要图形界面，验证内核、歌词写入、封面读写、联网抓取）：

```powershell
& ".\歌词内嵌工具\歌词内嵌工具.exe" --selftest <测试目录> <结果.json> --no-sandbox
```

## 六、注意

- `.mflac` / `.mgg` / `.qrc` 是 QQ音乐的**加密会员格式**，只有 QQ音乐能解码，任何第三方工具都读不了也写不了，
  程序会自动跳过并提示。需要先转成普通 FLAC/MP3。
  这类文件里连元数据都是加密的（实测 `fLaC`、`image/jpeg`、`QMQUALITY` 等明文标记一个都搜不到），
  所以转出来的 FLAC **天生没有封面**，只能用第 4 步联网补。
- 少数播放器只认歌曲同目录下的同名 `.lrc` 外挂文件，不读内嵌歌词。这种情况把歌词改名成和歌曲同名放一起即可。
- 若播放器显示乱码，用 `--fields LYRICS,UNSYNCEDLYRICS` 再写一次，或检查播放器的歌词编码设置。
- 联网抓封面需要网络。没网时会明确报错，不会假装「找不到封面」。
