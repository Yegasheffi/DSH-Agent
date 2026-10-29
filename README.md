# DSH Agent · 装在安卓手机里的 AI 智能体

一个能**直接操作你手机**的 AI 助手：你说一句话，它自己读文件、写文件、打开应用、发通知、跑命令，然后告诉你结果。

- **模型**：接入 DeepSeek（或任意 OpenAI 兼容接口），在手机上直接调 API，不经过任何中转服务器
- **执行**：19 个本机工具，全部在手机本地运行，不上传你的文件
- **可控**：写操作默认要你点「允许」，也可以切成只读模式
- **轻量**：APK 约 160 KB，无第三方依赖，不用 Root，不用电脑

> 适用：Android 8.0 及以上任意品牌手机。安装包自签名，需要允许「安装未知来源应用」。

| 对话与工具 | 写操作确认 | 会话列表 | 设置 |
| --- | --- | --- | --- |
| ![对话](docs/03-tools.png) | ![确认](docs/02-approval.png) | ![会话](docs/04-sessions.png) | ![设置](docs/05-settings.png) |

---

## 一、它能干什么

对着手机说一句话，它自己决定用哪个工具、分几步做完：

- **文件**：列目录、读文件、写/追加、建目录、移动重命名、递归删除、按文件名或内容搜索、看文件详情
- **应用**：列出已安装应用、按名字或包名直接打开
- **系统**：设备信息（型号/电量/存储/内存/网络/权限）、剪贴板读写、发通知、语音朗读
- **外部**：发 HTTP 请求（GET/POST）、用浏览器打开网址、调系统分享、用合适的应用打开文件（图片/PDF/APK 等）
- **shell**（默认关闭，需手动开启）：`ls`、`df`、`pm list packages` 等

不需要干活的时候它就只是聊天——问「你好」，它不会乱动文件。

---

## 二、三步开始用

### 1. 安装

把 `dist/DSH-Agent-v1.0.0.apk` 传到手机（微信/QQ 传输、USB、网盘都行），点开安装。

系统会提示「未知来源」，允许一次即可。Android 8.0（API 26）及以上都能装。

> 没有 APK？见[第八节](#八从源码构建)自己构建，或向提供方索取。

### 2. 填 API Key

首次打开会自动弹出设置：

1. **API Key**：点设置里的「还没有 Key？去 platform.deepseek.com 申请」，登录后创建密钥并复制（`sk-` 开头，只完整显示一次）
2. **Base URL**：保持默认 `https://api.deepseek.com`（其它服务商见[第六节](#六换成别的模型服务)）
3. **模型**：默认 `deepseek-flash`，也可以选 `deepseek-v4-pro`
4. 点「测试连接」，看到「连接成功」后点右上角「保存」

### 3. 给它文件权限

设置 → 手机权限 → **所有文件访问** → 打开开关。

不开也能用，但只能访问应用自己的目录（`Android/data/com.dsh.agent/files/DSH`）。开了才能真正读写相册、下载、文档这些地方。

> 各品牌路径大同小异：设置 → 应用 → DSH Agent → 权限 → 所有文件访问 / 文件和媒体。小米、OPPO、vivo 等机型可能还叫「访问所有文件」。

完成后回到对话，直接说要它做什么。

---

## 三、能用的自然语言指令长什么样

它听得懂人话，不需要记命令：

| 你想做的事 | 直接说 |
| --- | --- |
| 看看手机里有什么 | 「看看我手机里都有什么文件夹，挑重点给我说说」 |
| 整理下载目录 | 「把 Download 里的安装包挑出来，列个清单告诉我哪些可以删」 |
| 写文件 | 「在 DSH 目录下写一份今天的待办清单，三条」 |
| 找文件 | 「帮我找一个星期内的截图，按时间排序」 |
| 打开应用 | 「打开微信」 |
| 读设备状态 | 「看看我手机还剩多少存储，电量多少」 |
| 跑命令 | 「用 shell 看看 /sdcard 的占用情况」 |
| 发通知 | 「十分钟后提醒我一下」 |

---

## 四、它怎么工作（以及为什么安全）

```
你说话 → 模型决定调用哪些工具 → 手机本地执行 → 结果回给模型 → 继续或给答案
```

- **数据往哪走**：只有对话内容和工具返回的结果会发给你配置的模型服务商。文件不会自动上传，除非你把文件内容发给它。
- **Key 存哪**：只存在手机本地的应用私有存储（`SharedPreferences`），不写日志、不上传第三方，设置页里默认显示为圆点。
- **三道闸**：
  - 写操作（写入/删除/移动/打开应用/通知/分享/shell）默认先弹确认卡片，你点「允许」才执行
  - 可以切成「只读模式」，整轮对话都不会改动手机内容
  - 删除有保护：拒绝删根目录和 `/sdcard` 根，禁止写 `/system`、`/proc` 等系统目录
- **没有后台**：没有常驻服务、没有开机自启、没有统计上报。关掉应用就彻底停了。

---

## 五、内置工具（19 个）

| 工具 | 作用 | 默认是否要确认 |
| --- | --- | --- |
| `list_dir` | 列目录 | 否 |
| `read_file` | 读文本（可按行分页） | 否 |
| `file_info` | 文件/目录详情 | 否 |
| `search_files` | 按文件名或内容搜索 | 否 |
| `list_apps` | 已安装应用列表 | 否 |
| `device_info` | 设备与权限状态 | 否 |
| `open_url` | 浏览器打开网址 | 否 |
| `clipboard` | 读剪贴板 | 否 |
| `http_request` | HTTP 请求（GET 免确认） | 视方法 |
| `write_file` | 写/追加文件 | 是 |
| `make_dir` | 建目录 | 是 |
| `move_path` | 移动/重命名 | 是 |
| `delete_path` | 删除（递归） | 是 |
| `launch_app` | 打开应用 | 是 |
| `open_file` | 用其它应用打开文件 | 是 |
| `share_text` | 系统分享 | 是 |
| `notify` | 发系统通知 | 是 |
| `speak` | 朗读文字 | 是 |
| `run_shell` | 执行 shell（需开启） | 是 |

「执行策略」在设置里可切三档：**每次询问**（默认）/ **全部放行** / **只读模式**。

---

## 六、换成别的模型服务

只要对方兼容 OpenAI 的 `POST {base}/chat/completions`（含 SSE 流式与 `tools` 字段），就能用：

| 服务 | Base URL | 模型示例 |
| --- | --- | --- |
| DeepSeek（默认） | `https://api.deepseek.com` | `deepseek-flash` / `deepseek-v4-pro` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` |
| 自建 vLLM / Ollama | `http://192.168.x.x:8000/v1` | 你部署的模型名 |
| one-api / 中转 | 你自己的地址 | 中转里配置的模型名 |

改 Base URL + 模型名即可，其它不用动。局域网地址也支持（明文 HTTP 已允许）。

---

## 七、常见问题

**填了 Key 还是报 401**
Key 复制不全、带空格，或已失效。重新复制一次；注意别把 `sk-` 后面的内容截断。

**报 404**
Base URL 多了或少了 `/v1`。DeepSeek 用 `https://api.deepseek.com`；OpenAI 用 `https://api.openai.com/v1`。

**报 429 / 402**
额度用尽或请求过快。去开放平台充值，或降低频率。

**它说没有权限读文件**
没开「所有文件访问」，或只开了「文件和媒体」。到设置里打开「所有文件访问」；实在找不到（部分定制系统），它也能在应用自己的目录里工作。

**发不出通知**
Android 13+ 需要单独授权通知：设置 → 手机权限 → 通知 → 去授权。

**打开应用 / 分享没反应**
部分系统限制后台启动界面，回到前台再试一次。

**shell 说没开启**
设置 → 执行策略 → 打开「允许 shell 命令」。只在应用权限内执行，不是 root。

**长任务切后台就断了**
这一版没有前台服务，切后台太久可能被系统冻结。建议留在前台，或先在设置里调大「最大工具轮次」。

**换手机了，Key 还在吗**
不在。Key 只存本机，换机要重新填。

---

## 八、从源码构建

只需要 **JDK 17+** 和 **Android SDK**（build-tools 34+、任意一个 `platforms/android-3x`）。不需要 Gradle，也不需要 Android Studio 工程。

```bash
cd DSH-Agent
export ANDROID_HOME=/path/to/android-sdk     # macOS 默认 ~/Library/Android/sdk，Linux 常见 ~/Android/Sdk
./android/build.sh
# 产物：dist/DSH-Agent-v1.0.0.apk
```

脚本会自动探测 SDK（`ANDROID_HOME`、`ANDROID_SDK_ROOT`、常见安装路径、仓库下的 `.android-sdk/`），并挑版本最高的 build-tools。

想改界面：`android/assets/web/`（HTML/CSS/JS），改完重跑脚本即可，无需重装工具链。
想改图标：`android/tools/make_icon.py`。
想改文案或工具：`android/src/com/dsh/agent/`。

发布到应用市场需要自己重签（替换 `android/keystore.jks`），并注意 `MANAGE_EXTERNAL_STORAGE`、`QUERY_ALL_PACKAGES` 属于敏感权限，Google Play 有单独的审核政策。

---

## 九、兼容性

| 项目 | 说明 |
| --- | --- |
| 系统 | Android 8.0 – 15（API 26 – 35），已在 Android 15 上实测 |
| 屏幕 | 任意分辨率与横竖屏，界面自适应 |
| 语言 | 界面中文；问答常见中文，模型支持的语言都能聊 |
| 网络 | Wi-Fi / 移动数据；支持局域网自建模型（明文 HTTP） |
| 权限 | 存储（可选，建议给）、通知（可选）、安装未知来源（必须） |

针对机型差异做了两处兜底：

- 没授权文件访问时，顶栏会出现提示条，点一下直达系统授权页；同时工作目录自动降级到应用私有目录，不会卡死
- 部分系统找不到「所有文件访问」入口时，应用仍可在自己的目录里正常读写

**已知限制**

- 不支持图片输入（纯文本对话）
- 没有后台常驻 / 定时任务
- shell 非 root，部分系统命令受限
- 删除不可恢复，确认弹窗是最后一道防线
- 模型不擅长时可能绕圈，靠「最大工具轮次」兜底（默认 32）

---

## 十、项目结构

```
DSH-Agent/
├── dist/DSH-Agent-v1.0.0.apk        # 可直接安装
├── android/
│   ├── build.sh                     # 无 Gradle 构建脚本（自动探测 SDK）
│   ├── AndroidManifest.xml
│   ├── art/icon-src.jpg             # 图标原图
│   ├── tools/make_icon.py           # 生成各密度图标
│   ├── res/                         # 图标、主题、网络安全配置
│   ├── assets/web/                  # 界面（HTML/CSS/JS）
│   └── src/com/dsh/agent/
│       ├── MainActivity.java        # WebView、JS 桥、权限、入口
│       ├── Agent.java               # 工具循环、审批闸门、会话持久化
│       ├── Llm.java                 # OpenAI 兼容 SSE 客户端
│       ├── Tools.java               # 19 个工具的 schema 与实现
│       ├── Store.java / Config.java # 会话与配置
│       ├── FileProviderX.java       # content:// 文件共享
│       └── SelfTest.java            # 设备自检（可选）
├── video/                           # 教学视频（成片 + 剪映工程 + 素材）
└── docs/                            # 截图
```

---

## 十一、教学视频

`video/` 里有从「申请 API Key」到「各功能演示」的完整上手视频（横屏 1920×1080，2 分 48 秒）：

- 成片：`video/out/DSH-Agent-教学视频-横屏-配乐版.mp4`
- 可编辑工程：`video/jianying_draft/DSH Agent 教学视频 横屏 配乐版/`
- 素材与文案：`video/scenes/`、`video/out/subtitles.srt`、`video/scenes.py`

详见 [video/README.md](video/README.md)。

---

## 十二、许可与免责

个人开源项目，随缘维护。APK 为自签名，仅供自用与内部分发。

AI 会犯错：涉及删除、转账、隐私数据的操作，请务必看清确认卡片再点「允许」。作者不对使用本软件造成的任何数据丢失或损失负责。
