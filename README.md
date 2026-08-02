# YouTube Shorts Processor

YouTube视频下载 → Whisper语音转文字 → 整理成Obsidian笔记 → Git同步到GitHub

## 功能概述

将YouTube视频的音频下载下来，使用Whisper进行语音转文字，最后整理成结构化的Obsidian笔记，并同步到GitHub仓库。

## 目录结构

按博主（频道）分目录存储，目录名就是博主名字：

```
~/Documents/youtube/
├── 王志安/
│   ├── 王志安-中菲仁爱礁冲突.mp3
│   ├── 王志安-中菲仁爱礁冲突.md
│   ├── 王志安-中菲仁爱礁冲突_summary.md
│   ├── 王志安-中菲仁爱礁冲突.epub
│   └── 王志安-中菲仁爱礁冲突.txt
├── 小翠時政財經/
│   └── ...
└── 老周横眉/
    └── ...
```

文件命名格式：`博主名-标题关键词`

---

## 工作流程

### 1. 下载音频

```bash
# 创建博主目录（如果不存在）
mkdir -p ~/Documents/youtube/博主名

# 下载音频到对应目录
yt-dlp -x --audio-format mp3 -o "~/Documents/youtube/%(uploader)s/%(uploader)s-%(title)s.%(ext)s" "视频URL"
```

下载后手动重命名为简短格式：`博主名-标题关键词.mp3`

### 2. 语音转文字（whisper-cli）

```bash
whisper-cli -m ~/models/ggml-small.bin -l zh -otxt -of "~/Documents/youtube/博主名/输出文件名" "音频文件路径"
```

常用模型：
- `ggml-small.bin` - 推荐，速度和精度平衡
- `ggml-base.bin` - 最快但精度较低

> **注意**：如果转文字报错包含"敏感"、"失败"、"error"等关键词，说明内容敏感或转写失败，此时只保存音频文件，不生成txt和md。

### 3. 整理成MD笔记

**输出文件：**
- `.md` - 完整笔记（90%~110%原文字数）
- `_summary.md` - 总结摘要（约20%原文字数）

**格式要求：**
- 顶部元信息：`来源 | 日期 | 链接`
- **链接必须填写完整的YouTube视频URL**，不能留空或写"待补充"
- 按主题分章节，小标题清晰
- 删除重复啰嗦的口水话
- 把大段口语精简成条理清晰的表述
- 保留核心观点和关键细节
- 附录（如图片、喊话等）放最后
- 底部注明：*整理自YouTube视频语音转文字*

**口语转书面语要求：**

| 口语 | 书面语 |
|------|--------|
| "其实"、"就是说" | 直接陈述 |
| "对对对"、"是吧/对吧" | 删除 |
| "我觉得" | 客观陈述 |
| "那会儿"、"这会儿" | "当时"、"现在" |
| "然后呢/那么" | "因此/于是" |
| "大家可以看到" | 删除 |
| 语气词"啊/呀/吧/呢" | 删除或转换 |

**字数要求：**
- MD笔记字数必须在转写字数的 **90%~110%** 之间
- 总结摘要字数在转写字数的 **18%~22%** 之间

**不要做的事：**
- ❌ 大段原文直接复制
- ❌ 保留所有重复的废话
- ❌ 做太多删减导致内容不完整
- ❌ 口语化表达直接保留

### 3.5 搜索视频中提到的X推文

如果视频中提到了X（原Twitter）上的推文，必须搜索这些推文的原文，并作为附录放到笔记末尾。

**搜索方法**：
1. 从转写文本中提取提到的用户名、推文关键词
2. 使用 `agent-reach` 搜索X推文
3. 如果cookie过期，改用Exa搜索
4. 推文URL格式：`https://x.com/用户名/status/推文ID`

**附录格式：**
```markdown
## 附录：视频中提到的X推文

### 推文1｜用户名 @xxx
> 推文正文原文...
>
> [转发/回复链接] · 日期
```

### 4. 转换为Epub电子书

```bash
pandoc "博主名/笔记文件.md" -o "博主名/笔记文件.epub" --split-level=2
```

### 5. Git同步到GitHub

**远程仓库**：git@github.com:totticarter/youtube-audio.git

```bash
cd ~/Documents/youtube
git add 博主名/音频文件.mp3 博主名/笔记文件.md 博主名/笔记文件_summary.md 博主名/笔记文件.epub
git commit -m "添加: 博主名-标题"
git push origin main
```

> **注意**：
> - 如果是首次使用，先clone仓库
> - 如果远程有更新，先`git pull origin main`再push
> - 敏感内容只推送.mp3，不生成.md

---

## 常用命令

| 用途 | 命令 |
|------|------|
| 克隆仓库 | `git clone git@github.com:totticarter/youtube-audio.git ~/Documents/youtube` |
| 创建博主目录 | `mkdir -p ~/Documents/youtube/博主名` |
| 下载音频 | `yt-dlp -x --audio-format mp3 -o "~/Documents/youtube/%(uploader)s/%(uploader)s-%(title)s.%(ext)s" "URL"` |
| 语音转文字 | `whisper-cli -m ~/models/ggml-small.bin -l zh -otxt -of "~/Documents/youtube/博主名/输出文件名" audio.mp3` |
| 转Epub | `pandoc "博主名/笔记文件.md" -o "博主名/笔记文件.epub" --split-level=2` |
| Git推送 | `git add 博主名/*.mp3 博主名/*.md 博主名/*.epub && git commit -m "commit message" && git push origin main` |
| 查看模型 | `ls ~/models/` |

---

## 注意事项

1. whisper-lsp不支持-f参数，使用whisper-cli
2. whisper-cli需要指定模型路径
3. 转写时指定 `-l zh` 表示中文
4. 整理笔记时先看一遍原文，理解内容后再重新组织
5. 下载后手动重命名文件为简短格式
6. 转写失败或敏感时只保留.mp3，不生成txt和md
7. 所有文件通过git同步到GitHub仓库
8. 音频和md文件存放在以博主命名的目录下
9. **视频中提到的X推文必须搜索原文并作为附录附在笔记末尾**
