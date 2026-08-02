---
name: youtube-shorts-processor
description: YouTube视频下载、语音转文字、整理成Obsidian笔记的完整工作流
license: MIT
compatibility: opencode
metadata:
  audience: general
  category: youtube
---

## 功能
YouTube视频下载 → whisper语音转文字 → 整理成结构化Obsidian笔记 → Git同步到GitHub

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
│   ├── 小翠時政財經-AI导致7000万人失业.mp3
│   ├── 小翠時政財經-AI导致7000万人失业.md
│   ├── 小翠時政財經-AI导致7000万人失业_summary.md
│   └── 小翠時政財經-AI导致7000万人失业.epub
└── 老周横眉/
    └── 老周横眉-王局究竟是不是大外宣.mp3
```

## 文件命名格式

`博主名-标题关键词`（博主名自动从目录名获取）

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

**注意**：如果转文字报错包含"敏感"、"失败"、"error"等关键词，说明内容敏感或转写失败，此时只保存音频文件，不生成txt和md。

### 3. 整理成MD笔记

**输出文件：**
- `.md` - 完整笔记（90%~110%原文字数）
- `_summary.md` - 总结摘要（约20%原文字数）

**格式要求（参考《中国初中生怎么学台湾.md》）：**
- 顶部元信息：`来源 | 日期 | 链接`
- **链接必须填写完整的YouTube视频URL**，不能留空或写"待补充"
- 按主题分章节，小标题清晰
- 删除重复啰嗦的口水话
- 把大段口语精简成条理清晰的表述
- 保留核心观点和关键细节
- 附录（如图片、喊话等）放最后
- 底部注明：*整理自YouTube视频语音转文字*

**口语转书面语要求：**
将口语化表达转为正式书面表达：
- "其实" → 可删除或保留核心含义
- "就是说" → 直接陈述
- "对对对" → 删除
- "是吧/对吧" → 删除
- "我觉得" → 客观陈述
- "那会儿" → "当时"
- "这会儿" → "现在"
- "什么什么的" → 具体陈述
- "然后呢/那么" → 承接词"因此/于是"
- "其实我觉得" → 直接陈述观点
- "大家可以看到" → 删除
- "你有没有发现" → 直接陈述
- 语气词"啊/呀/吧/呢" → 删除或转换

**字数要求：**
- 转写文本字数：统计 .txt 文件的字符数（含中文标点）
- MD笔记字数：统计 .md 文件的字符数
- MD字数必须在转写字数的 **90%~110%** 之间
- 总结摘要字数在转写字数的 **18%~22%** 之间

**工作记录模板：**
```
### 字数统计
- 转写文本：X 字
- MD笔记：Y 字（Z%）
- 总结摘要：W 字（20%）
```

**不要做的事：**
- ❌ 大段原文直接复制（那是语音转文字的原始输出，不是笔记）
- ❌ 保留所有重复的废话（如"这个呢"、"然后呢"、"那么"等）
- ❌ 做太多删减导致内容不完整
- ❌ 口语化表达直接保留
- ✅ 保留关键细节和数据
- ✅ 将口语转换为书面语

### 3.5 搜索视频中提到的X推文（重要）

如果视频中提到了X（原Twitter）上的推文或回复（如某人发了什么推文、谁回复了谁等），**必须**搜索这些推文的原文，并作为附录放到笔记末尾。

**搜索方法**：
1. 从转写文本中提取提到的用户名、推文关键词
2. 使用 `agent-reach` 搜索X推文（twitter-cli）
3. 如果cookie过期，改用Exa搜索：`mcporter call 'exa.web_search_exa(query: "site:x.com 用户名 关键词")'`
4. 推文URL优先使用 `https://x.com/用户名/status/推文ID` 格式

**附录格式要求**：
```
## 附录：视频中提到的X推文

### 推文1｜用户名 @xxx
> 推文正文原文...
>
> [转发/回复链接] · 日期

### 推文2｜用户名 @xxx
> ...
```

**推文格式规范**：
- 保留原文（包括标点、大小写）
- 注明发帖人用户名和ID
- 如果是回复/转发，注明原推文链接
- 标注发帖日期
- 如果原文超过140字可适当精简，但保留核心内容

**整合到笔记末尾**：
- 推文附录放在"整理自..."注释**之前**
- 摘要笔记只需列出关键推文的时间线表格
- 主笔记需要包含完整推文原文

### 4. 转换为Epub电子书

使用pandoc将MD笔记转换为Epub格式：
```bash
pandoc "博主名/笔记文件.md" -o "博主名/笔记文件.epub" --split-level=2
```

### 5. Git同步到GitHub

完成下载、转文字、整理后，将音频文件(.mp3)、笔记文件(.md)、总结文件(_summary.md)及Epub文件(.epub)推送到远程仓库。

**远程仓库**：git@github.com:totticarter/youtube-audio.git

**同步命令**：
```bash
cd ~/Documents/youtube
git add 博主名/音频文件.mp3 博主名/笔记文件.md 博主名/笔记文件_summary.md 博主名/笔记文件.epub
git commit -m "添加: 博主名-标题"
git push origin main
```

**注意**：
- 如果是首次使用，先clone仓库：`git clone git@github.com:totticarter/youtube-audio.git ~/Documents/youtube`
- 如果远程有更新，先`git pull origin main`再push
- 敏感内容只推送.mp3，不生成.md

---

## 常用命令

- 克隆仓库：`git clone git@github.com:totticarter/youtube-audio.git ~/Documents/youtube`
- 创建博主目录：`mkdir -p ~/Documents/youtube/博主名`
- 下载音频：`yt-dlp -x --audio-format mp3 -o "~/Documents/youtube/%(uploader)s/%(uploader)s-%(title)s.%(ext)s" "URL"`
- 语音转文字：`whisper-cli -m ~/models/ggml-small.bin -l zh -otxt -of "~/Documents/youtube/博主名/输出文件名" audio.mp3`
- 转Epub：`pandoc "博主名/笔记文件.md" -o "博主名/笔记文件.epub" --split-level=2`
- Git推送：`git add 博主名/*.mp3 博主名/*.md 博主名/*.epub && git commit -m "commit message" && git push origin main`
- 查看模型：`ls ~/models/`

---

## 注意事项
1. whisper-lsp不支持-f参数，使用whisper-cli
2. whisper-cli需要指定模型路径
3. 转写时指定-l zh表示中文
4. 整理笔记时先看一遍原文，理解内容后再重新组织
5. 下载后手动重命名文件为简短格式：`博主名-标题关键词`
6. 转写失败或敏感时只保留.mp3，不生成txt和md
7. 所有文件通过git同步到GitHub仓库
8. 音频和md文件存放在以博主命名的目录下，目录自动创建
9. **视频中提到的X推文必须搜索原文并作为附录附在笔记末尾**
