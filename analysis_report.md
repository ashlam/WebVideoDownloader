# 项目 Skill 分析报告

## 项目概况

- 项目 `skills` 目录当前只有 1 个已安装 Skill：`video-downloader-skill`
- Skill 锁文件位于 `skills/.skills_store_lock.json`，记录来源为 SkillHub，当前版本为 `1.0.1`
- Skill 主目录位于 `skills/video-downloader-skill`
- 当前项目还包含一个工作区内的本地 SkillHub 运行环境：`.skillhub-home`，用于提供 `skillhub` CLI 和 `yt-dlp` 本地包装命令

## Skill 清单

| Skill 名称 | Slug | 版本 | 来源 | 目录 |
| --- | --- | --- | --- | --- |
| Video Downloader Skill | `video-downloader-skill` | `1.0.1` | SkillHub | `skills/video-downloader-skill` |

---

## 1. video-downloader-skill

### 1.1 基本信息

- `SKILL.md` 描述：通用视频下载工具，支持 YouTube、B 站、抖音等主流平台
- 入口脚本：`skills/video-downloader-skill/scripts/video_downloader.py`
- Skill 元数据：`skills/video-downloader-skill/_meta.json`
- 当前安装版本：`1.0.1`
- 当前项目中的本地增强：
  - 修复了格式选择逻辑，避免请求不可用的会员画质
  - 修复了重复下载到同一目录时返回旧文件的问题

### 1.2 工具调用方式

Skill 文档声明的调用方式：

```bash
/video-downloader <视频URL> [分辨率] [输出目录]
```

项目内实际可执行脚本调用方式：

```bash
python3 skills/video-downloader-skill/scripts/video_downloader.py <视频URL> [分辨率] [输出目录]
```

当前工作区内，为确保 `yt-dlp` 可用，实际推荐调用方式为：

```bash
PATH=/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/bin:$PATH \
python3 /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/skills/video-downloader-skill/scripts/video_downloader.py \
  "<视频URL>" "[分辨率]" "[输出目录]"
```

常见场景调用示例：

```bash
# 下载 YouTube 视频，默认优先 1080p
PATH=/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/bin:$PATH \
python3 /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/skills/video-downloader-skill/scripts/video_downloader.py \
  "https://www.youtube.com/watch?v=S8ESe_ZsDsQ" "" \
  "/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads"

# 下载 B 站视频，指定 1080p 目标画质
PATH=/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/bin:$PATH \
python3 /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/skills/video-downloader-skill/scripts/video_downloader.py \
  "https://www.bilibili.com/video/BV1HWRpBqEPp/?spm_id_from=333.40138.feed-card.all.click" "1080p" \
  "/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads"
```

空参数运行时，脚本会打印用法说明：

```text
用法: python video_downloader.py <视频URL> [分辨率] [输出目录]
示例: python video_downloader.py "https://youtu.be/xxx" "1080p" "./downloads"
```

说明：

- 该脚本没有实现 `--help`、`-h` 或子命令解析
- 参数全部采用位置参数
- 若分辨率参数格式不符合 `480p/720p/1080p` 这类形式，`int(resolution.rstrip('p'))` 可能触发异常

### 1.3 参数详解

| 参数名 | 类型 | 是否必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `视频URL` | `string` | 是 | 无 | 目标视频链接，传入任何 `yt-dlp` 支持站点的 URL |
| `分辨率` | `string` | 否 | `None` | 期望画质，典型值为 `1080p`、`720p`、`480p`；脚本会选择不高于目标值的最佳视频流 |
| `输出目录` | `path` | 否 | 临时目录 | 若不传则使用 `tempfile.mkdtemp(prefix='video_download_')` 创建临时目录 |

参数行为补充：

- 当不传 `分辨率` 时，脚本优先尝试 1080p；若不存在，则选择当前站点可用的最高分辨率
- 当前项目中的修复版本使用 `<选中video_format>+bestaudio/best` 的格式表达式，优先拼接最佳音频
- 当输出目录里已有旧文件时，当前项目中的修复版本会优先返回本次最新生成的文件

### 1.4 核心功能与使用场景

核心功能：

- 列出视频格式并自动选择目标分辨率
- 调用 `yt-dlp` 下载视频
- 使用 `--merge-output-format mp4` 尝试输出 `mp4`
- 对输出文件名做安全清洗，避免特殊字符导致路径问题
- 在检测到音视频分离时，尝试通过 `ffmpeg` 二次合并

适用场景：

- 下载公开可访问的 YouTube、B 站视频
- 批量手工测试 `yt-dlp` 能力
- 作为后续“转发到飞书”或其它自动化流程的前置步骤

限制与风险：

- 未实现标准 CLI 帮助参数
- 未声明但实际依赖 `ffprobe` 进行流检测
- 对分辨率参数缺少校验
- 对会员画质、受限内容、Cookie 鉴权场景没有内建支持
- `_meta.json` 仍显示 `1.0.1`，但当前工作区脚本已经有本地修改，和原始上游内容不完全一致

### 1.5 环境依赖

#### 运行依赖

| 依赖项 | 当前状态 | 说明 |
| --- | --- | --- |
| `python3` | 已安装 | 当前系统默认 `python3` 为 `Python 3.8.18`，可运行脚本本体 |
| `yt-dlp` | 已通过工作区本地包装提供 | 本项目未在系统 PATH 中安装 `yt-dlp`，当前通过 `.skillhub-home/bin/yt-dlp` 调用 |
| `ffmpeg` | 已安装 | 当前存在 `/usr/local/bin/ffmpeg`，版本 `8.0.1-tessus` |
| `ffprobe` | 缺失 | 当前 `PATH` 下未找到 `ffprobe`，会影响音视频流检测的准确性 |
| `Python 3.14` | 已安装 | 当前本地 `yt-dlp` 包装使用 `/usr/local/bin/python3.14` 执行 `yt_dlp` 模块 |

#### Python 依赖

当前项目本地 `yt-dlp` 运行环境位于 `.skillhub-home/vendor`，已安装：

- `yt-dlp==2026.03.17`
- `requests`
- `urllib3`
- `certifi`
- `charset_normalizer`
- `idna`
- `pycryptodomex`
- `brotli`
- `websockets`
- `mutagen`

#### 安装步骤

官方 Skill 文档要求的基础安装方式：

```bash
brew install yt-dlp
brew install ffmpeg
```

当前项目实际采用的本地安装方式：

```bash
# 1. 安装 SkillHub CLI 到工作区本地 HOME
HOME=/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home \
bash /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-installer/cli/install.sh --cli-only

# 2. 安装 Skill 到当前 workspace 的 skills 目录
HOME=/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home \
/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/.local/bin/skillhub \
  --skip-self-upgrade install video-downloader-skill

# 3. 将 yt-dlp 安装到工作区本地 Python 包目录
/usr/local/bin/python3.14 -m pip install --upgrade \
  --target /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/vendor \
  yt-dlp
```

当前项目中的 `yt-dlp` 包装脚本内容如下：

```bash
#!/usr/bin/env bash
set -euo pipefail

export PYTHONPATH="/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/vendor${PYTHONPATH:+:$PYTHONPATH}"
exec /usr/local/bin/python3.14 -m yt_dlp "$@"
```

#### 环境要求判断

- 操作系统：当前验证环境为 macOS
- Python：脚本本体可由 `Python 3.8.18` 运行，但新版 `yt-dlp` 推荐使用 `Python 3.9+`
- 外部工具：建议同时具备 `yt-dlp`、`ffmpeg`、`ffprobe`
- 网络：需要能访问目标视频站点及相应下载资源
- 鉴权：下载会员画质、私有内容或受地区限制内容时，需要额外 Cookie 或授权信息

### 1.6 成功调用案例

#### 案例 A：下载 YouTube 视频

命令：

```bash
PATH=/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/bin:$PATH \
python3 /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/skills/video-downloader-skill/scripts/video_downloader.py \
  'https://www.youtube.com/watch?v=S8ESe_ZsDsQ' '' \
  /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads
```

完整输出日志：

```text
获取视频格式信息...
选择格式: 399 (1920x1080)
开始下载...
检测到音视频分离，尝试合并...
下载完成: /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads/CAPTAIN_TSUBASA_2_-_WORLD_FIGHTERS_Story_Trail.mp4 (146.7 MB)

最终文件: /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads/CAPTAIN_TSUBASA_2_-_WORLD_FIGHTERS_Story_Trail.mp4
```

结果：

- 下载成功
- 输出文件：`downloads/CAPTAIN_TSUBASA_2_-_WORLD_FIGHTERS_Story_Trail.mp4`
- 文件大小：`153781865 bytes`

#### 案例 B：下载 B 站视频

命令：

```bash
PATH=/Users/wenguanggu/MyProjects/Python/WebVideoDownloader/.skillhub-home/bin:$PATH \
python3 /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/skills/video-downloader-skill/scripts/video_downloader.py \
  'https://www.bilibili.com/video/BV1HWRpBqEPp/?spm_id_from=333.40138.feed-card.all.click' \
  1080p \
  /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads
```

完整输出日志：

```text
获取视频格式信息...
选择格式: 30033 (854x478)
开始下载...
检测到音视频分离，尝试合并...
下载完成: /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads/5_100__ChatGPT-Image2_ChatGPT5.5.mp4 (3.6 MB)

最终文件: /Users/wenguanggu/MyProjects/Python/WebVideoDownloader/downloads/5_100__ChatGPT-Image2_ChatGPT5.5.mp4
```

结果说明：

- 下载成功
- 由于该视频的更高分辨率在未提供会员 Cookie 的情况下不可下载，脚本回退到了公开可用的较低分辨率流
- 输出文件：`downloads/5_100__ChatGPT-Image2_ChatGPT5.5.mp4`
- 文件大小：`3734100 bytes`

### 1.7 失败与排查记录

在分析过程中，出现过以下真实问题，并已确认原因：

1. 早期运行时，`yt-dlp` 不在系统 PATH 中
   - 现象：脚本无法找到 `yt-dlp`
   - 处理：在 `.skillhub-home/vendor` 中安装 `yt-dlp`，并增加 `.skillhub-home/bin/yt-dlp` 包装脚本

2. 使用系统默认 `python3` 驱动旧版本地 `yt-dlp` 时，B 站下载失败
   - 现象：提示 `Support for Python version 3.8 has been deprecated`
   - 原因：当前系统默认 `python3` 为 `3.8.18`，对新版 `yt-dlp` 的兼容性较差
   - 处理：改用 `/usr/local/bin/python3.14` 执行 `yt_dlp`

3. 原始 Skill 版本在 B 站场景下会错误请求不可用格式
   - 现象：`Requested format is not available`
   - 原因：原脚本格式表达式不稳定，容易落到不可下载流
   - 处理：改为使用精确视频格式加 `bestaudio/best`

4. 原始 Skill 版本在同一输出目录重复下载时会返回旧文件
   - 现象：本次成功下载 YouTube 视频后，脚本却报告上一次 B 站文件为“最终文件”
   - 原因：脚本直接选取输出目录中的第一个视频文件
   - 处理：改为按修改时间倒序，返回最新生成文件

5. 当前环境缺少 `ffprobe`
   - 影响：音视频流检测不准确，日志中可能出现“检测到音视频分离”但并未真正执行有效合并
   - 建议：补齐 `ffprobe` 并确保其在 PATH 中

### 1.8 结论

`video-downloader-skill` 当前已经可以在本项目中稳定完成公开 YouTube 与 B 站视频下载，且经本地修复后，比原始安装版本更适合当前环境直接使用。  
但它仍然存在以下后续优化空间：

- 增加标准 `--help` / `-h` 参数解析
- 增加分辨率参数合法性校验
- 明确声明 `ffprobe` 依赖
- 增加 Cookie 支持能力
- 增加对只下载音频、平台分目录输出等增强功能

