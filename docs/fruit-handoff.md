# 水果拟人视频：全流程复现与 Agent 交接

整理日期：2026-10-13。对象是本聊天已交付的六水果、第二版人物、30 秒女声配音成片。

本文包含制作决策、参考图规范、完整生图和视频提示词、Grok 项目操作、剪辑参数、配音文案、可复制脚本与验收记录。它是本次实际过程的复盘及操作手册，不代表后续平台入口和额度一直不变。

## 1. 接手时先读：最终交付到底是什么

- 主题：**你讨厌的水果变成人，还会讨厌吗？**
- 水果顺序：榴莲、香瓜、油柑、芭乐（番石榴）、黄皮、诺丽果。
- 视觉：原创虚构成年东亚人物，半写实电影感植物奇幻服饰，暖金温室；用果皮、果肉、籽、叶片构建服装轮廓和材质。
- 画面：9:16，720×1280，24fps，720 帧，精确 30 秒；无标题、字幕、名称、编号或水印文字层。
- 结构：开场 2 秒 + 六个水果各 4 秒 + 六宫格选择结尾 4 秒。
- 最新声音要求：**温柔、有一点神秘感的女声旁白**。早期用户要求无声，后来明确授权添加配音，故有声版是最新交付。没有另外添加 BGM 或音效。
- 视频制作路线：8 张已批准静帧 → Grok 项目内逐段生成约 6 秒原片 → FFmpeg 选段剪辑 → 单独合成旁白 → 无损复制视频流封装有声版。
- “最终成片已导出”不等于“全面人工视听验收通过”。技术和采样检查已完成；连续运动审核及独立试听存在未验证项，见第 10 节。

### 本机关键位置

```text
工作区：C:/Users/pc/Documents/ChatGPT/互动视频
批准静帧：output/fruit-personification-v2/
视频制作目录：output/fruit-personification-grok/
最新有声成片：output/fruit-personification-grok/fruit-personification-30s-voiced.mp4
保留无声成片：output/fruit-personification-grok/fruit-personification-30s-silent.mp4
纯旁白：output/fruit-personification-grok/voiceover/narration-30s.mp3
旁白 WAV：output/fruit-personification-grok/voiceover/narration-30s.wav
预览索引：output/fruit-personification-grok/review.html
```

Grok 项目：[讨厌的水果变成人｜第二版无声短片](https://grok.com/project/6b7840e6-f85b-4175-b4a9-0ba8a0cc3fed?tab=conversations)。项目名沿用无声阶段，后期配音在本地完成。

实际生成会话：[8 段生成所在会话](https://grok.com/imagine/agent/6b7840e6-f85b-4175-b4a9-0ba8a0cc3fed?conversation=12f68b89-4968-4937-808a-f3c0ffa2c5b3)。网页标题最初是“生成水果开场竖屏视频第01段”，后续七段也在同一会话。

### 两种复现模式

**A. 复用原片，重建同一成片。** 同一工作区中直接使用 `originals/*.mp4`、`assemble.ps1`、`voiceover/*.mp3` 与混音脚本。无需重新生图、重新调用 Grok 或重新合成语音。相同媒体输入可重建相同视听内容；不同 FFmpeg 版本可能使容器哈希不同。

**B. 从头制作同类作品。** 按下文完成选题、静帧审批和逐段生成。生成模型具有随机性，即使用同一 Prompt，也不保证复现相同脸、动作或视频。精确复现本作品必须同时携带批准 PNG、Grok 原片及已生成旁白；单个 MD 不包含二进制素材。

复制到另一台机器时，保留下面的目录结构。本文附录已内嵌所有核心提示词和重建脚本；路径、工具版本和账户权限需要按接手环境核实。

```text
output/
  fruit-personification-v2/
    01-durian.png ... 06-noni.png
    07-opening.png
    08-ending.png
  fruit-personification-grok/
    originals/01-opening.mp4 ... 08-ending.mp4
    assemble.ps1
    edit-decisions.json
    tasks.json
    segment-prompts.md
    voiceover/
      narration.json
      01-hook.mp3 ... 08-choice.mp3
      generate_narration.py
      mix_narration.py
      python-deps/          # 依赖可重新安装，不必跨机器复制
      audio-qc.json
```

## 2. 选题与创意收敛

用户先提出约 20 秒的视频，再扩为 30 秒；水果和蔬菜拆成两集，每集六种。本次只完成水果集。第一轮名单被认为争议性不够，后续确定以上六种；香瓜归水果。

历史要求是网上调研且不使用小红书。当前交接材料没有完整、可追溯的原始调研来源清单，因此不能把六种水果包装成“全网最讨厌排名”或虚构票数。这是用户认可的创作名单。

制作新一期时：查实际讨论中的分歧理由，保存 URL、时间及代表观点，区分气味、涩感、口感、籽多等原因，再选择视觉特征差异足够大的六种。避免把少量帖子当统计排名。沿用用户“不使用小红书”的约束，直到其明确变更。

将“讨厌的理由”转为人物反差：坚硬外壳与温柔内在、涩与回甘、粗糙表皮与粉色内心。每个角色至少有独特轮廓、颜色材质和一种容易读懂的动作。

## 3. 人物与静帧制作

### 3.1 角色锁定表

| 文件 | 水果/人物 | 必须保留的果实特征与服装 | 标志动作 | 常见错误 |
|---|---|---|---|---|
| 01-durian.png | 榴莲男 | 青绿尖刺分瓣宽肩披风、奶黄厚内衬、棕色籽扣 | 拨开披风、伸手邀请 | 金属盔甲、整颗水果头 |
| 02-melon.png | 香瓜女 | 白绿纵条圆润分瓣裙、藤蔓盘发饰、金黄籽腔腰饰 | 轻提裙缘、转身回望 | 橙肉网纹哈密瓜、普通婚纱 |
| 03-amla.png | 油柑男 | 淡黄绿浅纵沟、六分瓣肩衣、乳白内衣、细叶枝环 | 触枝环、回头微笑 | 柑橘切瓣、葡萄 |
| 04-guava.png | 芭乐女 | 粗糙浅绿短外套、粉色高领内搭和阔腿裤、浅籽腰链、叶脉肩饰 | 翻开外套领口 | 裤装变长裙、纹理变鳞片 |
| 05-wampee.png | 黄皮男 | 小型金黄椭圆分枝果串、琥珀外套、乳白内层、籽扣和长巾 | 扶长巾转身、伸手 | 巨大圆球、满身挂果 |
| 06-noni.png | 诺丽果女 | 乳白圆钝多边形外衣与小凹点、深绿宽叶披肩、白花发饰 | 抬前臂舒展宽叶 | 凹点变眼睛/疮孔、惊悚器官 |

所有角色为成年人、正常人头与人体、完整着装。六人脸和服装各自锁定，不能在不同段互换。

### 3.2 实际迭代方式

1. 第一版用于建立方向；用户认可后提出“更有特色”。
2. 第二版沿用第一版对应脸部与风格参考，显著加强果实特征与服装轮廓。
3. 用户当时明确要求多个子任务，因此六个角色曾由六个子任务并行生图；每个只负责自己的文件。开场独立生成，结尾依赖六角色完成后制作。
4. 使用内置 imagegen。完整实际 Prompt 见附录 A。新 agent 应读取其环境适用的图像生成工具规范；本次的并行授权不能推定为其他新任务自动授权。
5. 若从零开始且没有第一版参考，不能原样声称“Input image 1”。应先建立人物基准，或将相应句改为文字人物描述；若直接复用本次 V2，则上传 V2 PNG 锁定人物。
6. 图像文件曾输出约 941×1672。不要把 Prompt 中的“9:16”当成已验证的精确尺寸，交付时检查实际输出。

### 3.3 开场与结尾

开场 `07-opening.png`：暖金温室、深色圆石桌、六水果同框，一只成年人的手从下方犹豫伸向果实。不写问题字幕；问题后来由旁白表达。

结尾 `08-ending.png`：严格两列三行、细黑分隔线，统一脸部大小与留白。左上榴莲、右上香瓜、左中油柑、右中芭乐、左下黄皮、右下诺丽果。

本次结尾采用两步图像合成：先以五个角色参考制作五格，右下留空；再用中间合成图和诺丽果参考，仅填右下，保持其他五格。原因是当时工具采用最多五张参考的调用方式；后续使用时重新核实工具能力，不把此限制当永久事实。附录 A 保留了实际两步 Prompt，执行顺序按“第一步 → 填右下”。

## 4. Dola 失败路线：保留历史，不当成成功前置步骤

曾计划将 8 张图送 Dola 直接生成 30 秒。第一批账号 008/013/014 登录失效；第二批 005/007/009 的本地提交流程卡住，随后管理器账号条目消失。没有确认到可用视频或可靠的供应商生成任务，不应凭本地记录声称已扣费或成功提交。

用户随后改为 Grok 分段生成。成功成片来自 Grok，不来自 Dola。重建本片无需再次尝试 Dola。历史 `video-prompt.txt` 是 Dola 版整片提示词，不要把它与附录 B 的 Grok 分段版混用。

## 5. Grok 项目建立、参考绑定与生成

### 5.1 项目准备

本次从侧栏“添加项目”创建项目，上传全部 8 张 PNG 到项目文件，保存项目制作约束，再从项目内 Imagine 会话生成。接手已有项目先确认原任务及文件；制作新项目时按当前 UI 执行，不使用过期元素编号。

可复制的项目约束如下（由本次要求整理，非导出的逐字设置快照）：

```text
制作约30秒、9:16的水果拟人短片。使用项目中的8张PNG参考。
六种水果依次是榴莲、香瓜、油柑、芭乐、黄皮、诺丽果。
保持对应原创成年人物的脸、发型和服装，统一暖金植物温室风格。
每条消息只执行指定的一段，生成一个实际可播放视频，不自动生成额外图片或重复任务。
段序为开场、六种水果、两列三行六宫格结尾。
视频素材阶段无文字、无对白、无音乐、无音效，旁白在后期单独处理。
使用当前账户已有权限/额度；不购买升级或额外付费包。
```

本次没有记录供应商底层模型精确版本，准确称呼为“Grok Imagine 项目入口”；不能自动写成 Seedance 或自行填模型号。

### 5.2 实际上传经验

项目文件上传成功。进入 Imagine 画布后，聊天框“添加图片”打开的媒体菜单曾把 PNG 当成音频，出现“无法读取该音频文件”，并未附上有效图片。

后来成功方式：**画布工具栏“上传图片” → 文件选择器 → 指定 PNG → 等待“上传中”消失并在画布确认角色 → 发送该段文本。** 图像进画布不一定出现聊天附件标签，应确认实际图像和对应文件。仅在确实上传时使用“刚上传/附图”的表述。

使用本环境 CUA 时，可采用下面的交互模式。必须先按工具要求绑定当前页、读取文档和实时状态；不要在无该 API 的环境直接执行。

```javascript
// tab 是已按当前浏览器工具文档绑定的项目会话。
const pending = tab.playwright.waitForEvent('filechooser', {timeoutMs:10000})
  .catch(error => ({error:String(error)}));
await tab.playwright.getByRole('button', {name:'上传图片', exact:true}).click();
const chooser = await pending;
if (chooser.error) throw new Error(chooser.error);
await chooser.setFiles([absoluteReferencePngPath]);
await tab.getAXState();
// 等当前状态确认上传结束、图像正确，再填入该段文本并发送。
await tab.playwright.getByRole('textbox', {name:'Ask Grok anything'}).fill(segmentPrompt);
await tab.playwright.getByRole('button', {name:'发送', exact:true}).click();
await tab.getAXState();
```

不要强点隐藏的 `input[type=file]`：本次尝试未触发选择器，未处理的异步拒绝还曾使浏览器 REPL 重置。为等待文件选择器的 Promise 捕获异常，失败先检查状态。

### 5.3 分段生成与验收节奏

1. 请求开场 6 秒，确认有真实可播放视频，并下载保存。
2. 试制榴莲变身，检查参考绑定、果实到服饰的变化、人物终态。通过本次采样检查后再继续其他角色。
3. 按附录 B 逐段提交，文本限制每次只生成一条，不让网页 agent 自行重试或扩展。
4. 各段本次实际约 6.041667 秒、24fps，视频主流 720×1264；即使写 9:16，也必须按实际媒体检查。
5. 保存全部原片，剪辑只从真实运动中选区间，保留末端动作与观众眼神。
6. 生成未结束时检查同一个任务，不能重新发送当作查询。状态未知先恢复，不重复生成。
7. 原片即使网页自述“完全无声”，实际仍可能有 AAC 音轨；本次明确在剪辑时移除，后期再加入旁白。

“果实变人”主要依赖纹理匹配、前景遮挡和揭示：果壳展开为披风、条纹匹配裙摆、叶片掠过后显露人物。并非每一帧都是连续物理变形；例如榴莲、香瓜开头可能已在果实后方隐约露出人物。这是本次实际生成表现，后续可按审美要求返修。

### 5.4 识别并下载正确的视频

页面同时可能存在旧项目媒体、缩略图副本、多个 video 标签。先通过新回复、播放画面和 DOM 中的 `currentSrc` 确认新媒体，不能永远下载 `video.nth(0)`。本次新媒体恰好处于首个位置，但它不是可靠通用规则。

使用当前浏览器工具支持的 `downloadMedia({timeoutMs:20000})` 或页面下载按钮，取得实际本地下载路径后，再复制到 `originals/`。本次浏览器下载偶有明显延迟，应先查询已有下载，不能因此重复生成。实际源媒体 ID 见附录 E；不要拼接猜测资源地址，也不要复制登录凭据到文档。

## 6. 镜头时间线与剪辑

以下为本次已导出成片的实际选段，不是新生成素材的通用剪辑点。帧号按 24fps、从零计数、结束帧不包含。

| 成片时间 | 内容 | 原片起始帧 | 取帧数 | 原片取用区间（秒） |
|---|---|---:|---:|---|
| 0–2 | 开场 | 0 | 48 | 0–2 |
| 2–6 | 榴莲 | 12 | 96 | 0.5–4.5 |
| 6–10 | 香瓜 | 18 | 96 | 0.75–4.75 |
| 10–14 | 油柑 | 6 | 96 | 0.25–4.25 |
| 14–18 | 芭乐 | 12 | 96 | 0.5–4.5 |
| 18–22 | 黄皮 | 12 | 96 | 0.5–4.5 |
| 22–26 | 诺丽果 | 6 | 96 | 0.25–4.25 |
| 26–30 | 六宫格 | 0 | 96 | 0–4 |

完整 PowerShell 脚本见附录 C。关键顺序是：`fps=24 → trim(帧) → setpts=N/(24*TB) → scale/pad → setsar=1 → format=yuv420p`，输出明确 `-r 24 -fps_mode cfr`。

本次踩坑：仅用 `setpts=PTS-STARTPTS`、在不合适位置做 fps 转换，先出现 712 帧/29.67 秒；另一次虽有 720 帧，拼接后约 29.71 秒并出现重复时间戳警告。改为按帧序号重建时间戳与明确恒定帧率后，验证为 720 帧/30.000 秒，无警告。后续仍要检查实际输出，不能只相信脚本里的数字。

720×1264 原片等比放入 720×1280，主要上下各补 8 像素黑边，保护完整构图。各段 H.264、CRF18、medium，去除原音轨，仅映射 `0:v:0`，避免附带封面视频流。统一后 concat，`-c:v copy -an -movflags +faststart` 输出无声片。

## 7. 配音：文案、发音、时序与合成

用户最终选择温柔、有神秘感的女声。实际采用 Microsoft Edge 在线 TTS，通过开源 [edge-tts](https://github.com/rany2/edge-tts) 调用；这是合成女声，不是真人录音，也没有克隆某个人。该项目文档是当时实现的 API 依据，当前服务可用性需要实测。

本次参数：`zh-CN-XiaoxiaoNeural`，rate `-8%`，pitch `-3Hz`。voice 查询曾返回 Warm 标签。注意 `direction` 中文文字仅用于制作说明；本次脚本实际传入的是文本、voice、rate、pitch，**没有把“神秘感”作为模型 style 参数传入**。实际听感不能用这些参数代替试听验收。

| 旁白起点 | 文案 | 本次音频长度 | 结束时间 |
|---:|---|---:|---:|
| 0.10 | 讨厌的水果，变成人呢？ | 3.024 | 3.124 |
| 3.40 | 榴莲，也有温柔。 | 2.424 | 5.824 |
| 6.20 | 香瓜，把清甜穿在身上。 | 3.168 | 9.368 |
| 10.20 | 油柑，冷淡之后，是回甘。 | 3.480 | 13.680 |
| 14.20 | 芭乐，藏着粉色的心。 | 2.880 | 17.080 |
| 18.20 | 黄皮，把阳光带给你。 | 2.808 | 21.008 |
| 22.20 | 诺丽果，藏着什么秘密？ | 2.832 | 25.032 |
| 26.20 | 现在，你还会拒绝谁？ | 2.784 | 28.984 |

开场问题有意跨过 2 秒切点进入榴莲画面，到 3.124 秒结束；3.40 秒开始榴莲短句。其余每句留出段内停顿，最后约一秒给观众看六宫格。

实际制作：先生成开场和榴莲试音文件，核时长，再生成其余六句。每句独立 MP3，通过 `asyncio.Semaphore(2)` 限制并发请求（不是子 agent）；再按绝对起点 `adelay` 混合。旁白不足 30 秒的尾部补的是静音，不是视频静帧。

混音目标参数：48kHz 单声道，`loudnorm=I=-18:TP=-1.5:LRA=9`，输出 PCM16 WAV，另存 MP3，封装 AAC 到 MP4，视频流用 `copy`。-18 LUFS/-1.5dBTP 是处理目标，未做独立综合响度复测，不能把它写成实测值。实测 WAV 样本峰值约 -2.16dBFS，削波样本 0。

旁白脚本、JSON 和混音脚本全文见附录 D。当前脚本会跳过已有同名 MP3，所以修改文字/声线后应另建版本目录或先将旧 MP3 移入备份目录，避免误用缓存。生成异常若留下空/损坏文件，也应先核验再重试，不要把“存在”当“完整”。

## 8. 本机重建命令

如果只想看成片，直接打开现成 MP4，不要运行重建命令。下面命令会覆盖同名衍生输出；编辑新版时请先另存完整制作目录。

在 PowerShell 中：

```powershell
Set-Location -LiteralPath 'C:\Users\pc\Documents\ChatGPT\互动视频'
$projectPython = 'C:\Users\pc\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe'
$projectFfmpeg = 'C:\Users\pc\Documents\ChatGPT\豆包exe\page-bridge\bin\ffmpeg.exe'

# 复用原片，重建无声片。
& 'output/fruit-personification-grok/assemble.ps1'

# 若该目录没有 Python 依赖，安装到本项目目录。
& $projectPython -m pip install --target 'output/fruit-personification-grok/voiceover/python-deps' edge-tts==7.2.8 mutagen==1.48.1 numpy

# 仅在确实需要重新配音时查询 voice、生成音频；复用现有 MP3 时跳过这两步。
& $projectPython 'output/fruit-personification-grok/voiceover/generate_narration.py' --voices
& $projectPython 'output/fruit-personification-grok/voiceover/generate_narration.py'

# 复用现有逐句 MP3 合成 WAV、MP3 和最终有声片。
& $projectPython 'output/fruit-personification-grok/voiceover/mix_narration.py'

# 完整解码检查视频和音轨；控制台应显示30秒，结束时720帧，无解码报错。
& $projectFfmpeg -hide_banner -i 'output/fruit-personification-grok/fruit-personification-30s-voiced.mp4' -map 0:v:0 -map 0:a:0 -f null -
```

换机器时应替换 Python 与 FFmpeg 路径，尤其 `assemble.ps1` 和 `mix_narration.py` 内的 FFmpeg 常量。缺少在线 TTS 能力时，现有旁白仍可复用；不要未经说明改用另一个声线再称为同一版本。在线服务无额外付款流程不等于永久免费或商业授权保证，遇到付费门槛应核对当前用户授权。

## 9. 交付与版本记录

交付至少包含有声成片、纯旁白、文案、保留的无声版；制作包还应保留 PNG、8 个原片、实际 prompts、任务 ID、选帧表、脚本、哈希和 QC。

`production-plan.md` 和 `tasks.json` 的 target.audio=false、final 指向无声片，是无声阶段记录，**不是最新有声交付的总状态**。最新配音结果在 `voiceover/audio-qc.json`，本手册明确最新交付为 `fruit-personification-30s-voiced.mp4`。接手者不能只读旧 target 就删掉配音。

本次浏览器曾拒绝访问 `file://` 本地预览页。没有通过换协议/本地服务等办法绕过该安全拒绝；最终用 Codex 本地文件/媒体预览交付。后续 agent 必须遵循自己的工具约束，不照搬未经允许的浏览器操作。Grok 项目结果截图保存在 `grok-project-proof.png`。

## 10. 已验证结果与仍需试听的事项

**已验证：** 8 个 Grok 原片已下载；最终有声片 30 秒、720×1280、24fps、720 帧，H.264 + 48kHz 单声道 AAC；完整解码无报错；旁白按各自窗口放置，没有时间重叠和截断；WAV 长 30 秒，削波样本 0；有声版与无声版的视频流哈希一致，添加声音未重编码画面。

**视觉检查覆盖：** 原片每秒约 2 帧采样，初版剪辑每秒约 3 帧采样；六种水果到对应人物的变化和六宫格存在。最终时间戳修复保持选帧和构图。没有完成独立连续实时播放的运动验收，不能声称所有中间帧、手部、口型都已通过。

**声音检查覆盖：** 已做文件解码、时长、峰值和时序检查；没有独立听审声线、发音和情绪。重点听“芭乐”的多音字、“油柑”“诺丽果”的读音，结尾是否像自然提问，句间是否生硬。不能把 TTS 文本或 JSON 当成实听证据。

**已知风格偏差：** 芭乐阔腿裤的裤腿分界在部分中景不清楚；变身多采用遮挡揭示；开场仅取前 2 秒，手部完整收回未全部入剪。若这些是下一版硬要求，应对应返修，不要在记录中掩盖。

### 后续 agent 的交付前检查

1. 核实际媒体尺寸、时长、帧数和音轨，不只看 Prompt 或网页成功文案。
2. 连续看完 30 秒，检查每次变身、人物脸、手、服装、相邻剪切和六宫格格位。
3. 佩戴耳机听完旁白，核发音、语气、停顿、音量与剪点；有问题仅重做受影响句。
4. 音频返修后重新封装，比较视频流哈希，确认认可画面不受影响。
5. 已有结果状态不明先查询/校验；不要重复付费生成，不扩大到发布或账户购买。
6. 把“实际完成、技术验证、人工视听、用户批准”分开记录。

## 11. 可直接发给下一个 agent 的接手指令

```text
请先完整读取《水果拟人视频_全流程复现与Agent交接.md》。
当前最新交付是30秒女声配音版；六水果与第二版人物造型已经确定。
先确认本机素材和输出文件，优先复用原片和已生成逐句旁白。
若我只要求重建，不重新调用imagegen或Grok；按附录脚本生成衍生视频并核验。
若我要求制作新一期，复用制作方法，重新确认选题、参考身份、平台能力、预算和动作区间；不要把历史权限、媒体ID和硬编码路径当通用配置。
保留原版，修改另存版本。不得把平台成功提示、采样画面或技术解码当成完整视听验收。
交付可播放的实际文件、文案和检查结论，指出真实未验证项。
```

---

以下附录从本次实际落盘文件读取，供其他 agent 在没有原脚本副本时复制。图像 Prompt 包含上一版参考依赖，执行前按第 3 节处理；Grok 视频 Prompt 是冻结的制作文本，实际发送时曾加入“附图/刚上传”等参考说明。

## 附录 A：第二版 8 张静帧的完整 Prompt（含结尾两步合成）

原文件：`output/fruit-personification-v2/final-prompts.md`。

```text
# 水果拟人化第二版：最终 Prompt 集

使用内置 imagegen。六个角色由六个子任务并行生成，开场独立生成，结尾按角色参考分两步生成。图像保持无字，第一版完整保留。

## 01-durian-prompt

Use case: stylized-concept.
Asset type: pure image, vertical 9:16 character review still for a fruit-personification short film.
Input image 1 is a reference for the adult East Asian man's approximate facial identity, grounded human anatomy, refined semi-realistic cinematic botanical-fantasy style and color family. Create a significantly more distinctive second design, do not simply add decorations or copy the old pose.

Subject: a handsome adult East Asian man, the gentle guardian inside a durian's thorny shell. Normal human head, realistic unobstructed face, dark tousled hair and warm intent eye contact. Fully dressed. The defining new silhouette is a broad-shouldered short botanical mantle made of FIVE large deep olive-green durian-rind lobes that open along natural fruit seams; organically layered and sculptural, matte coarse rind texture with short pyramidal thorns, never metallic armor. Each lobe is a believable garment panel, not a literal fruit attached to the body. Beneath the shell-like mantle is thick cream-yellow soft silk in plump rounded folds recalling durian flesh segments, with one polished brown seed-kernel-shaped fastening. Deep green tailored trousers and understated boots. The shell-versus-soft-interior contrast must be immediately visible from the silhouette and garment construction. No excessive gold embroidery or random jewels.
Action: dynamic half-turned stance, one hand gently drawing a mantle lobe aside to reveal the pale cushioned silk interior; the other hand extending toward the viewer in a generous invitation. Exactly two natural arms and anatomically convincing hands. Face remains clear, expression quietly protective and warm rather than threatening.
Scene: dark warm-gold botanical courtyard, soft leaf shadows, restrained foreground leaves; a small real whole durian and one real opened durian wedge on a low stone plinth at the lower left provide fruit identification without competing with the character. Golden dappled cinematic side light separates the thorny exterior from the luminous creamy interior. Consistent sophisticated botanical fantasy fashion, lifelike face and fabric, cinematic depth, restrained atmosphere.
Composition: single character, vertical 9:16, near full body with head, shoulders, both hands and mantle silhouette clearly inside frame. Eye-level portrait lens. The character is the dominant subject. A pure finished image.
Constraints: no writing anywhere, no text, no numbers, no labels, no logo, no watermark, no collage, no split panels. No fruit head, no monster anatomy, no literal enormous durian body, no metal armor, no bare torso, no nudity, no extra characters.


## 02-melon-prompt

Use case: stylized-concept
Asset type: pure no-text portrait concept still for a 30-second fruit personification video, version 2.
Input image 1 is a STYLE AND FACE reference, not an edit target. Preserve the same adult East Asian woman's approximate face identity, natural human face, polished semi-realistic cinematic botanical fantasy rendering, detailed luxurious natural materials. Create a new distinctive design and dynamic pose.
Primary request: a fresh, agile vine dancer embodying Chinese white-flesh thin-skin sweet melon (香瓜), a clear visual evolution of the reference.
Composition: vertical 9:16 portrait, full character silhouette visible, medium-full view, woman turning lightly and lifting a small corner of her skirt. Her face clearly visible to camera in three-quarter view, gentle confident playful smile, anatomically natural hands. Tendrils and skirt swirl in the same direction, elegant motion frozen as a still.
Costume: unmistakable round pale ivory-and-green smooth melon peel transformed into a short structured curved shoulder cape and a voluminous rounded sculptural skirt, NOT a generic wedding gown. Modest closed opaque ivory underdress, covered chest, refined botanical couture. Several prominent light green vertical melon segment lines run down cape and skirt, smooth waxy melon-skin texture with fresh dew. Outer skirt layers have a water-fresh lightly translucent material only OVER the fully opaque clothing, no exposed transparency. Distinctive arching curled vine tendril hair ornaments rise in airy loops above the dark updo, small melon leaves. Golden flattened teardrop melon seed beads form an elongated central seed-cavity oval on the waist in a truly unique ornamental motif. Natural human head and face, no fruit head.
Backdrop: charcoal brown botanical greenhouse with restrained pale green luminous accents, warm cinematic rim light and soft natural face light, shallow depth of field. Beside her on a stone plinth, one small REAL white-green striped round thin-skin melon and a cut half exposing WHITE flesh and golden central seeds, subtle but unmistakable identification cue. NOT orange-fleshed melon, NOT netted cantaloupe.
Palette: ivory, pale fresh green, tiny golden seed accents, warm charcoal background. Rich photographic skin texture, detailed botanical couture, cinematic grounded fantasy, realistic human anatomy.
Constraints: one adult woman only, coherent costume fully dressed and tasteful, strong melon morphology, no text, numbers, captions, graphic labels, logos, watermarks, panel layouts, ordinary white wedding dress, orange flesh or cantaloupe netting.


## 03-amla-prompt

# 油柑拟人化 · 第二版

Use case: stylized-concept
Asset type: 9:16 portrait fruit-personification video review still, one image without text.
Input images: Image 1 is identity and aesthetic reference only. Keep approximately the same handsome adult East Asian man's face, black hair, slim physique, and cinematic semi-real botanical fantasy realism. Create a substantially more distinctive second design, change costume, pose, and environment details.
Primary request: Amla / Indian gooseberry / 油柑 personified as a reserved elegant young adult man whose initial cool gaze softens into a tiny warm smile, expressing astringent first taste and sweet aftertaste.
Scene: Dark charcoal atmospheric botanical chamber, a large elegant round loop of slender woody branches behind and beside him, several small intact pale yellow-green amla fruits hanging separately from delicate twigs, neat rows of tiny elongated oval leaflets. Quiet jade-green glow.
Costume: unmistakably designed around amla's small round translucent pale yellow-green fruit with six shallow longitudinal grooves. A distinctive structured pale chartreuse cropped upper jacket with six curved petal-shaped tailored panels, rounded crisp silhouette and a long split cape dropping behind him; it must be clearly different from the reference straight long robe. Semi-translucent fruit-like material ONLY in sculptural outer ornament panels with soft subtle six-groove ridges; fully opaque ivory inner shirt and opaque trousers. This is refined botanical couture, NOT armor. A prominent small spherical pale jade chest clasp showing six shallow longitudinal grooves. Cuffs with orderly miniature elongated oval leaflets.
Pose and composition: three-quarter body or near-full body vertical 9:16, figure turns his torso sideways and looks back at camera, one hand lightly rests on the circular twig ring, face clearly visible and sharply resolved, clean anatomically correct hand. Beautiful calm expression with slight softness at mouth. The amla clasp and cape silhouette clearly readable.
Lighting: cinematic cool pale jade rim light and warm soft skin key light, subtle haze, exquisite tangible fabric, clean natural skin, premium fantasy portrait.
Constraints: only one fully clothed adult human, normal human head and face, preserve reference identity approximately; no fruit head. Fruit cues integrated into tailoring, material, silhouette and leaflets. Actual fruits shown only as small WHOLE round amla with six shallow vertical grooves. Absolutely no citrus cross-section, lime wedges, citrus segments, grapes, text, Chinese characters, captions, numbers, typography, logos, watermark, frames, collage. No nudity, no armor, no giant fruit helmet. Output one pure image in portrait 9:16.


## 04-guava-prompt

Use case: identity-preserve / stylized-concept.
Asset type: second-version pure visual fruit-personification character review still, vertical 9:16.
Input image 1 is the previous guava character; preserve her adult East Asian face identity, shoulder-length dark wavy hair and cinematic semi-realistic botanical fantasy image quality. Reimagine her outfit, silhouette, pose and scene with much stronger guava specificity and a confident outgoing personality.
Primary request: one adult East Asian woman embodying pink-fleshed guava. Human face and human head. Full-length central portrait, head and shoes visible, elegant anatomy. She looks straight at camera with a bright confident small smile. One hand naturally lifts and folds out one outer jacket lapel to reveal the vivid interior, the other hand relaxed.
Outfit: a cropped structured geometrical pale-green jacket whose tailored surface unmistakably echoes actual guava skin: dense fine uneven dimples and mottled matte green, refined high-detail not scales and not spikes. Its opened lapels have white-green cut-edge piping resembling a sliced guava rind. Beneath is an opaque high-neck, fully sleeved saturated rose-pink outfit with volume, clean angular pleats and broad flowing wide-leg trousers. She is completely and tastefully clothed. The open green exterior and rich pink inner volume give a bold cut-open-guava silhouette. A strand of small hard ivory guava-seed shaped pearls hangs naturally from her waist. One asymmetrical shoulder ornament is made from two broad oval guava leaves with striking parallel lateral veins; a tiny dried guava calyx forms the lapel clasp. Keep accessories coherent and subordinate to face.
Fruit identity prop: a small real fresh pink guava half on a dark stone ledge in foreground, pale green bumpy rind, white rind band, bright pink flesh and hard cream seeds grouped toward center. A few oval guava leaves in scene; no other fruit.
Scene: warm charcoal dark botanical atelier background, restrained pink rim light, soft cinematic key light, lifelike skin, detailed fruit materials and fabric, editorial fantasy realism. Clear silhouette, close enough to read facial expression, dramatic yet not scary, graceful contrast of rough exterior and bright interior.
Constraints: single coherent image, one woman only; no captions, labels, numerals, text, typography, signage, logo or watermark; no grid, no collage. No fruit head, no literal fruit replacing anatomy, no transparency, no bare chest, no low neckline, no long pale-green gown similar to first version. Natural two hands and five fingers, normal limbs.


## 05-wampee-prompt

# 黄皮拟人化 V2

Use case: stylized-concept.
Asset type: second-version fruit personification review still, portrait 9:16, cinematic semi-realistic botanical fantasy.
Input image 1 is a reference for the established adult East Asian male face identity and cinematic realism. Preserve broadly the same handsome face and tousled brown-black hair, but substantially redesign costume and pose to be more distinctive.

Subject: an adult East Asian man personifying WAMPEE (黄皮, Clausena lansium), a sunny nimble botanical wanderer. Fully human face and head, fully dressed, believable hands. Three-quarter to full-length portrait, ample room around the distinctive shoulder ornament. His body is taking a half-step turn toward the camera, one hand holding the shawl at his shoulder, bright smiling eyes and a relaxed confident lively expression.

Botanical identity must be unmistakable: wampee fruits are SMALL yellow-brown oval fruits with delicate velvety fuzz, borne on NATURALLY BRANCHING CLUSTERS, paired with pointed oval dark green leaves. Opaque creamy-white flesh surrounding a glossy dark green-brown seed. Construct a prominent elegant organic branching fruit-cluster ornament across ONE shoulder and an asymmetric diagonal sash, several connected small yellow-brown oval fruit shapes and slender branches visibly integrated as wearable botanical couture. This must be much more striking than a tiny brooch. Avoid covering the entire body with hanging balls; keep a light graceful silhouette.

Costume: warm golden-yellow suede cropped jacket, asymmetric flowing shawl, opaque ivory inner shirt and tailored ivory trousers, green leaf collar detail, glossy dark-green seed-shaped clasps; layered tactile botanical couture with fine fruit skin fuzz in the ornaments. Less ordinary suit, more imaginative wanderer design, no armor. Clear readable contrast between brown-gold fruit skin, ivory fruit-flesh fabric, dark seed accents and pointed green leaves.

Scene: dark warm charcoal botanical studio background, amber cinematic side and rim light, natural skin texture, rich detailed suede and fabric. A few real wampee clusters with leaves on a small stone pedestal beside him; one fruit peeled open to reveal OPAQUE creamy-white flesh and glossy dark seed. Keep these props unobtrusive. Strong character-focused fashion editorial, lively pose, no other people.
Constraints: pure image with NO letters, NO numbers, NO captions, NO labels, NO logos, NO watermarks. No orange or citrus wedges, no citrus cross-section, no spines, no armor, no fruit-shaped head, no cartoon, no excessive jewelry, no floating fruit storm. Accurate wampee botanical cues; cohesive premium cinematic semi-realistic style.


## 06-noni-prompt

# 诺丽果拟人化 V2

生成方式：内置 imagegen；V1 诺丽果角色为身份与画风参考。

Use case: identity-preserve.
Asset type: second-version fruit-personification character review still, pure image, vertical 9:16.
Input image 1 is identity and cinematic style reference. Preserve the approximate face identity of the adult East Asian woman, dark long hair, natural beauty, elegant half-realistic botanical fantasy photography. Reimagine her clothing, silhouette, pose and lighting to make a much more distinctive noni fruit embodiment.
Primary request: a mysterious ivory noni fruit queen without a crown, sculptural and botanical rather than a generic floral ball gown. Mature Morinda citrifolia is ivory to pale yellow-white, an irregular elongated oval compound fruit, its surface divided into broad rounded polygonal segments with sparse small dark dimples. Translate this morphology directly into an asymmetrical outer mantle composed of large softly rounded, three-dimensional ivory polygonal sections with visibly raised curved seams, smooth translucent fruit-like waxy material. Only a few subtle dimples, absolutely not dozens of eye-like rings or a small repeated printed pattern. Distinctively sculptural silhouette: the mantle wraps from one shoulder diagonally around the torso and opens into a few broad ivory petal-like outer panels around the lower body. Opaque pale jade-white high-neck long-sleeved inner garment beneath, fully covered and dignified. Ivory outer panels glow softly but reveal no skin.
A single enormous glossy dark green noni leaf forms an arched shoulder cape and falls along the raised forearm, its strong central vein and broad rounded leaf shape clearly readable. The woman gently lifts one forearm and spreads this leaf cape across the arm, calm direct eye contact, relaxed anatomically correct hands. A few tiny white tubular noni flowers only by her hair at one side, no floral crown, no flower clusters across the dress.
Scene: charcoal-dark botanical chamber, subtle dark green shadows, ethereal ivory key light and deep green reflected fill, cinematic realistic fabric and botanical textures. At lower foreground include one authentic ripe ivory irregular oval noni fruit with broad polygonal surface and a few sparse dimples, and one small unripe green noni fruit beside broad glossy leaves as quiet identification clues.
Composition: single character, full length head to hem, 9:16 vertical, prominent readable face and distinctive clothing silhouette, editorial cinematic realism, sophisticated unusual botanical fantasy, premium film still, rich visual depth.
Constraints: no text, no labels, no numbers, no watermark, no logos, no borders. Adult human face, no fruit-shaped head. No eyeballs, disease lesions, frightening holes, trypophobia texture, pink noni, nudity, transparent skin-revealing clothes, additional people, exaggerated queen crown, repeated generic tiny floral-pattern long dress.


## 07-opening-prompt

Use case: illustration-story. Asset: second-version OPENING still of a vertical 9:16 cinematic fruit-personification short video. Use the supplied first-version opening image as a style and fruit-identity reference. Retain the same six actual fruit types and premium semi-realistic warm cinematic rendering, but create a more distinctive magical conservatory still life.
Composition: a single dark circular stone tasting table seen from a slight elevated front angle. Six fruit identities clearly arranged around its edge like a semicircle of choices, with realistic relative scale: left rear large thorny green-brown durian and a creamy gold flesh segment; right rear smooth pale white-green thin-skinned oriental melon with faint longitudinal stripes and a white-flesh cut half with tan seeds; center pink-flesh green guava cut half and small whole guava; front-left pale yellow-green small grooved whole amla fruits; front-center branching bunch of small oval fuzzy golden wampee; front-right a mature ivory-white irregular elongated noni with polygonal small recessed eye marks and broad glossy green leaf. No other fruit types.
Light choreography suggests transformations will begin: subtle amber light at durian, soft celadon around melon and amla, a rose-pink reflected glint near guava, warm golden halo at wampee, pearl-white rim at noni. Elegant natural vine curves frame the sides; one adult hand hesitantly reaches from bottom toward center then slightly withdraws, proper anatomy. The fruits remain intact real fruits with no human faces, no limbs, no visible mutation. Clear dark breathing space in top quarter for a future question added during editing; absolutely no typography generated. Atmospheric particles sparse and tasteful, sharp believable edible fruit texture, no rot or slime. Single clean picture with no words, labels, numbers, watermark, logo, inset or grid.


## 08-ending-prompt

# V2 结尾选择静帧

## 第二步

Use case: compositing. Input image 1 is the precise edit target: a vertical 9:16 two-column three-row selection grid with five finished portrait panels and bottom-right blank. Input image 2 is the noni woman's identity and clothing insert reference.
EDIT ONLY THE BOTTOM-RIGHT EMPTY PANEL of image 1. Fill that panel with a head-to-waist portrait of the adult East Asian noni woman from image 2, keeping her face identity, long dark wavy hair, tiny white flowers at hair side, ivory sculptural asymmetrical noni-polygon mantle with broad curved 3D seams and a few tiny dark dimples, fully opaque pale jade-white high-neck inner garment, enormous glossy dark green leaf cape at her shoulder and along gently raised forearm. Calm direct gaze. Crop so face scale and headroom exactly match other five portraits; same warm cinematic botanical lighting and dark leafy background. Do not include full body or small face. Keep her leaf cape recognizable.
ABSOLUTE INVARIANTS: preserve image 1's other FIVE panels, all five faces, hair, clothes, poses, backgrounds, lighting and colors unchanged. Preserve exactly two columns, three rows, identical panel boundaries, fine black gutters, tiny empty black top strip, same overall vertical canvas. Do not rearrange, retouch or redraw any other cell. No text, no names, no labels, no numbers, no watermarks, no logos anywhere. Final image contains six single-person portraits, one per panel, no empty panel.

内置 imagegen 分两步合成，最多五张参考。第一步五角色，第二步局部插入诺丽果，保留其余五格。

## 第一步

Use case: compositing. Asset type: fruit-personification video ending selection still. Generate a pure photographic cinematic vertical 9:16 image with an exactly TWO COLUMNS by THREE ROWS grid of equally sized rectangular portrait panels, separated by fine black gutters. Tiny empty black strip across the very top, containing no writing. Six panels total but bottom-right panel left empty charcoal-black for later insertion. The five input images are identity and clothing references, each corresponds ONLY to its designated panel. Crop each character from head through waist / mid-torso and use the same face scale across panels, clean visible hair and recognizable clothes, no full-body tiny figures. Consistent warm botanical movie light, natural adult East Asian facial features, rich half-realistic botanical fantasy.
Top-left: input image 1 durian man, preserve face and tousled brown dark hair, olive-green sharply spiked split-shell shoulder cloak, pale golden flesh inner clothing, confident calm expression.
Top-right: input image 2 melon woman, preserve face, updo with spiraling vine tendrils and tiny white flowers, pale ivory melon-inspired dress with green vertical stripes, tan seed ornaments, lively sweet smile.
Middle-left: input image 3 amla man, preserve face and straight dark hair, pale translucent yellow-green ribbed shoulder garment with white seams, cream inner clothing, compound small leaves around arch background, cool calm glance.
Middle-right: input image 4 guava woman, preserve face and dark wavy bob, pale textured green tailored cropped jacket pulled open with one hand, coral-pink opaque high-neck clothing, leaf brooch, friendly warm smile.
Bottom-left: input image 5 wampee man, preserve face and dark wavy hair, ochre outer robe and cream inside, oval golden berry clusters and narrow green leaves on shoulder, gently inviting smile.
Bottom-right: EMPTY dark panel, no person no object. This is an intermediate intentional blank cell.
Constraints: exactly the stated grid and character placement. Match each reference identity and distinct clothes. No text, no labels, no symbols, no numbers, no watermark, no logo, no typography anywhere. No extra faces, duplicated people, fruit heads, floating portraits, graphic cards or rounded edges. Separate panels photographically with slender black lines. Equal face prominence and headroom in all five filled panels.
```

## 附录 B：Grok 八段完整视频 Prompt

原文件：`output/fruit-personification-grok/segment-prompts.md`。

```text
# Grok 分段提示词

共同约束：项目中PNG为AI生成的原创虚构成年人物；9:16竖屏、暖金逆光植物温室、电影感半写实奇幻风格；保持对应参考图的脸、发型和服装；无文字、无对白、无配乐、无音效。每条只生成一个视频，使用当前已有额度，不购买升级、不生成额外图片、不自行重复任务。请求6秒原片，最终按真实动作区间剪辑。

## 01 开场

只生成第01段「开场」，调用视频生成工具输出一个6秒、9:16竖屏视频。以项目文件07-opening.png作为实际图像参考和起始构图，保留六种水果的形态、位置、暖金植物温室和圆形深色石桌。镜头开始于略俯视石桌，一只成年人的手从画面下方试探伸向水果，在前两秒内轻轻停顿并收回，镜头同时缓慢靠近后侧榴莲；之后保持果实清晰和轻微镜头推进，给剪辑留余量。不出现人物变身，本段只有真实水果和一只自然手部。不增加水果、不改变水果种类。完全无声，不生成任何文字、对白、口型、音乐或音效。请直接生成并返回可播放可下载视频，只执行这一段，不生成其他段或额外图片。

## 02 榴莲

只生成第02段「榴莲变身」，输出一个6秒、9:16竖屏视频。01-durian.png锁定人物脸型、深棕短发、尖刺绿色分瓣披风、奶黄色厚实内衬及棕色果籽腰扣；07-opening.png只参考榴莲果实和温室光色。开头是完整真实榴莲的青绿尖刺与果壳裂缝特写，淡奶黄色果肉可见。镜头沿裂缝推进，果壳向两侧展开，通过纹理匹配和遮挡衔接成原型男性的绿色尖刺肩披风，果肉颜色成为内衬。镜头拉至人物中景，他一手拨开披风，另一手向观众邀请，克制微笑。前4秒内完成从果实到人物的变化与邀请，随后自然呼吸稳定保持。参考图中的男性为唯一终态人物，不更换脸或服装。无水果头、裸露或惊悚肉体变形，无文字、对白、音乐、音效。只调用一次视频生成并返回实际视频，不另画人物。

## 03 香瓜

只生成第03段「香瓜变身」，输出一个6秒、9:16竖屏视频。02-melon.png锁定女性脸、盘发、瓜藤发饰、白绿纵条分瓣圆润裙摆与金黄色籽饰；07-opening.png只参考白肉薄皮香瓜。开头近看香瓜白绿色表皮的纵条纹，镜头沿条纹移动，圆润瓜皮弧线以材质匹配衔接为原型女性的分瓣裙，卷曲瓜藤成为发饰，籽饰保持在腰部。镜头拉至中景，她轻提一侧裙缘，轻转身体，裙摆顺势展开，回望观众微笑。前4秒内完成变身、轻转和回望，随后自然保持。温室暖金光保持统一。保持原型成人脸和服装，无橙肉网纹瓜、其他人物、文字、对白、音乐、音效。只生成一个视频，不额外生图或重复提交。

## 04 油柑

只生成第04段「油柑变身」，输出一个6秒、9:16竖屏视频。03-amla.png锁定男性脸、黑色侧分发型、淡黄绿色六分瓣肩衣、半透明外层与不透明乳白内衣，保留果实饰扣和带小叶的枝环。起始为完整淡黄绿色油柑表面浅纵沟的特写，细小叶片掠过镜头，浅沟纹理衔接成六分瓣肩部衣片。遮挡揭开后呈现原型男性，他侧身触碰油柑枝环，回头看向镜头，清冷表情转成轻微笑意。前4秒完成变化和回头，随后自然保持。油柑没有柑橘切瓣。保留同一温室、人物脸与衣服，无其他人物、文字、对白、音乐、音效。只生成一条视频，不另生图片或自行重试。

## 05 芭乐

只生成第05段「芭乐变身」，输出一个6秒、9:16竖屏视频。04-guava.png锁定女性脸、深色波浪短发、粗糙浅绿短外套、粉红高领不透明内搭、粉红阔腿裤、浅籽腰链及清晰叶脉肩饰。开头特写真实芭乐的粗糙绿色果皮与粉红果肉，镜头掠过粉色切面，绿色果皮纹理通过遮挡衔接到原型短外套，粉色果肉转成内搭和裤装，浅籽成为腰链。镜头拉至中景，她轻翻开一侧外套领口露出粉色内层，向观众明朗微笑。前4秒完成变身和翻领，随后自然保持。不要把阔腿裤变成长裙，不换脸或服装。统一暖金温室，无文字、对白、音乐、音效。只生成一条视频，不另画人物或重复提交。

## 06 黄皮

只生成第06段「黄皮变身」，输出一个6秒、9:16竖屏视频。05-wampee.png锁定男性脸、深色微乱短发、蜂蜜琥珀短外套、乳白内衣和长裤、分枝金黄椭圆果串肩饰、深色籽扣及长巾。开头特写枝上的小型金黄椭圆黄皮果串和细绒表面，果串划过前景遮挡镜头，揭开时已成为原型男性肩饰，果皮的暖黄材质对应外套。男性扶着肩侧长巾转身半步，长巾轻扬，另一手邀请观众，温暖微笑。前4秒完成变身和转身邀请，随后自然保持。黄皮保持小型椭圆果串，人物服装不改成新造型。统一温室暖金光，无其他人物、文字、对白、音乐、音效。只生成一条视频，不另生图片或重复提交。

## 07 诺丽果

只生成第07段「诺丽果变身」，输出一个6秒、9:16竖屏视频。06-noni.png锁定女性脸、长深色头发、小白花发饰、乳白色圆润多边形雕塑外衣、完整高领浅玉白内衣及宽大深绿叶片披肩。开头特写成熟乳白诺丽果的圆钝多边形外皮和小凹点，宽大绿色叶片掠过镜头，遮挡衔接到女性叶片披肩，白色果皮结构对应原型不对称外衣。镜头拉至中景，她轻抬前臂，宽叶顺势舒展，微抬下巴看向观众，安静而神秘。前4秒完成变身和抬臂，随后自然保持。不要把凹点变成人眼或恐怖疮孔，保持原型脸和服装。统一暖金温室，无文字、对白、音乐、音效。只生成一条视频，不额外生图或自行重试。

## 08 结尾选择

只生成第08段「六宫格选择结尾」，输出一个6秒、9:16竖屏视频。以08-ending.png作为实际起始画面，严格保持两列三行的等大六宫格及细黑分隔线，六个人物的脸、发型、服装和各自格位不变：左上榴莲男，右上香瓜女，左中油柑男，右中芭乐女，左下黄皮男，右下诺丽果女。固定镜头，六人各自轻微呼吸和自然眨眼、极少量衣料摆动，保持温暖邀请或回望姿态。不扩格、不拼成同一场景、不复制人物、不出现空格。不生成任何文字、名称、序号、对白、音乐或音效。保持整段稳定、无声，结束在完整六宫格。只生成一条视频，不另生图片或重复提交。
```

## 附录 C：assemble.ps1 完整剪辑脚本

原文件：`output/fruit-personification-grok/assemble.ps1`。

```powershell
$ErrorActionPreference = 'Stop'
$ffmpeg = 'C:\Users\pc\Documents\ChatGPT\豆包exe\page-bridge\bin\ffmpeg.exe'
$productionRoot = $PSScriptRoot
$cuts = @(
    @{ id='01-opening'; start=0; count=48 },
    @{ id='02-durian'; start=12; count=96 },
    @{ id='03-melon'; start=18; count=96 },
    @{ id='04-amla'; start=6; count=96 },
    @{ id='05-guava'; start=12; count=96 },
    @{ id='06-wampee'; start=12; count=96 },
    @{ id='07-noni'; start=6; count=96 },
    @{ id='08-ending'; start=0; count=96 }
)
$segmentsDir = Join-Path $productionRoot 'edited-segments'
New-Item -ItemType Directory -Path $segmentsDir -Force | Out-Null
foreach ($cut in $cuts) {
    $inputPath = Join-Path $productionRoot ('originals/' + $cut.id + '.mp4')
    $outputPath = Join-Path $segmentsDir ($cut.id + '.mp4')
    $lastFrame = $cut.start + $cut.count
    $filter = "fps=24,trim=start_frame=$($cut.start):end_frame=$lastFrame,setpts=N/(24*TB),scale=720:1280:force_original_aspect_ratio=decrease,pad=720:1280:(ow-iw)/2:(oh-ih)/2,setsar=1,format=yuv420p"
    & $ffmpeg -hide_banner -loglevel error -y -i $inputPath -map 0:v:0 -vf $filter -r 24 -fps_mode cfr -an -c:v libx264 -crf 18 -preset medium -movflags +faststart $outputPath
    if ($LASTEXITCODE -ne 0) { throw "Editing failed: $($cut.id)" }
}
$concatPath = Join-Path $productionRoot 'concat.txt'
$concatLines = $cuts | ForEach-Object { "file 'edited-segments/$($_.id).mp4'" }
[System.IO.File]::WriteAllLines($concatPath, $concatLines, [System.Text.UTF8Encoding]::new($false))
$finalPath = Join-Path $productionRoot 'fruit-personification-30s-silent.mp4'
& $ffmpeg -hide_banner -loglevel error -y -f concat -safe 0 -i $concatPath -map 0:v:0 -c:v copy -an -movflags +faststart $finalPath
if ($LASTEXITCODE -ne 0) { throw 'Assembly failed' }
$cuts | ConvertTo-Json -Depth 5 | Set-Content -LiteralPath (Join-Path $productionRoot 'edit-decisions.json') -Encoding UTF8
Get-Item -LiteralPath $finalPath | Select-Object FullName, Length
```

## 附录 D1：narration.json 旁白与时序

原文件：`output/fruit-personification-grok/voiceover/narration.json`。

```json
{
  "voice": "zh-CN-XiaoxiaoNeural",
  "rate": "-8%",
  "pitch": "-3Hz",
  "direction": "温柔、有一点神秘感；轻声邀请，句尾留白，避免夸张推销腔。",
  "lines": [
    {"id":"01-hook","start":0.10,"end":3.35,"text":"讨厌的水果，变成人呢？"},
    {"id":"02-durian","start":3.40,"end":5.90,"text":"榴莲，也有温柔。"},
    {"id":"03-melon","start":6.20,"end":9.85,"text":"香瓜，把清甜穿在身上。"},
    {"id":"04-amla","start":10.20,"end":13.85,"text":"油柑，冷淡之后，是回甘。"},
    {"id":"05-guava","start":14.20,"end":17.85,"text":"芭乐，藏着粉色的心。"},
    {"id":"06-wampee","start":18.20,"end":21.85,"text":"黄皮，把阳光带给你。"},
    {"id":"07-noni","start":22.20,"end":25.85,"text":"诺丽果，藏着什么秘密？"},
    {"id":"08-choice","start":26.20,"end":29.80,"text":"现在，你还会拒绝谁？"}
  ]
}
```

## 附录 D2：generate_narration.py 逐句语音生成

原文件：`output/fruit-personification-grok/voiceover/generate_narration.py`。

```python
import asyncio
import json
from pathlib import Path
import sys

ROOT = Path(__file__).resolve().parent
sys.path.insert(0, str(ROOT / 'python-deps'))
import edge_tts
from mutagen.mp3 import MP3

async def main():
    spec = json.loads((ROOT / 'narration.json').read_text(encoding='utf-8'))
    if '--voices' in sys.argv:
        voices = await edge_tts.list_voices()
        print(json.dumps([v for v in voices if v['ShortName'] == spec['voice']], ensure_ascii=True))
        return
    chosen = set(sys.argv[1:])
    lines = [x for x in spec['lines'] if not chosen or x['id'] in chosen]
    gate = asyncio.Semaphore(2)
    async def generate(line):
        async with gate:
            path = ROOT / (line['id'] + '.mp3')
            if not path.exists():
                await edge_tts.Communicate(line['text'], spec['voice'], rate=spec['rate'], pitch=spec['pitch']).save(str(path))
            print(json.dumps({'id':line['id'], 'duration':MP3(path).info.length, 'file':str(path)},ensure_ascii=True),flush=True)
    await asyncio.gather(*(generate(x) for x in lines))

if __name__ == '__main__':
    asyncio.run(main())
```

## 附录 D3：mix_narration.py 混音和有声片封装

原文件：`output/fruit-personification-grok/voiceover/mix_narration.py`。

```python
import hashlib
import json
from pathlib import Path
import subprocess
import sys
import wave

ROOT = Path(__file__).resolve().parent
sys.path.insert(0, str(ROOT / 'python-deps'))
from mutagen.mp3 import MP3
import numpy as np

FFMPEG = r'C:\Users\pc\Documents\ChatGPT\豆包exe\page-bridge\bin\ffmpeg.exe'
spec = json.loads((ROOT / 'narration.json').read_text(encoding='utf-8'))
inputs = []
filters = []
report = {'voice':spec['voice'], 'rate':spec['rate'], 'pitch':spec['pitch'], 'lines':[]}
for index, line in enumerate(spec['lines']):
    path = ROOT / (line['id'] + '.mp3')
    duration = MP3(path).info.length
    if duration > line['end'] - line['start']:
        raise ValueError(f"Speech exceeds its slot: {line['id']}, {duration}")
    inputs += ['-i', str(path)]
    filters.append(f"[{index}:a]aresample=48000,adelay={round(line['start']*1000)}:all=1[a{index}]")
    report['lines'].append({**line, 'duration_seconds':duration, 'actual_end':line['start']+duration})
joined = ''.join(f'[a{i}]' for i in range(len(spec['lines'])))
filters.append(f'{joined}amix=inputs=8:duration=longest:normalize=0,apad,atrim=duration=30,loudnorm=I=-18:TP=-1.5:LRA=9,aresample=48000[out]')
wav = ROOT / 'narration-30s.wav'
mp3 = ROOT / 'narration-30s.mp3'
video = ROOT.parent / 'fruit-personification-30s-voiced.mp4'
source = ROOT.parent / 'fruit-personification-30s-silent.mp4'
subprocess.run([FFMPEG,'-hide_banner','-loglevel','error','-y',*inputs,'-filter_complex',';'.join(filters),'-map','[out]','-ac','1','-c:a','pcm_s16le',str(wav)],check=True)
subprocess.run([FFMPEG,'-hide_banner','-loglevel','error','-y','-i',str(wav),'-c:a','libmp3lame','-b:a','192k',str(mp3)],check=True)
subprocess.run([FFMPEG,'-hide_banner','-loglevel','error','-y','-i',str(source),'-i',str(wav),'-map','0:v:0','-map','1:a:0','-c:v','copy','-c:a','aac','-b:a','192k','-t','30','-movflags','+faststart',str(video)],check=True)
def video_hash(path):
    return subprocess.check_output([FFMPEG,'-v','error','-i',str(path),'-map','0:v:0','-c','copy','-f','hash','-'],text=True).strip()
report['source_video_stream_hash'] = video_hash(source)
report['output_video_stream_hash'] = video_hash(video)
assert report['source_video_stream_hash'] == report['output_video_stream_hash']
with wave.open(str(wav),'rb') as reader:
    report['wav_sample_rate'] = reader.getframerate()
    report['wav_frames'] = reader.getnframes()
    report['wav_duration'] = reader.getnframes()/reader.getframerate()
    samples = np.frombuffer(reader.readframes(reader.getnframes()),dtype='<i2').astype(np.float64)/32768
report['sample_peak_dbfs'] = float(20*np.log10(np.max(np.abs(samples))))
report['clipped_samples'] = int(np.count_nonzero(np.abs(samples)>=32767/32768))
report['final_sha256'] = hashlib.sha256(video.read_bytes()).hexdigest()
report['validation'] = 'Technical decode, duration, no-overlap, peak, and identical video packet checks. Listening quality and pronunciation not independently auditioned.'
(ROOT / 'audio-qc.json').write_text(json.dumps(report,ensure_ascii=False,indent=2),encoding='utf-8')
print(json.dumps(report,ensure_ascii=True,indent=2))
```

## 附录 E：8 段实际媒体索引

| 原片 | 对应参考 | Grok 媒体 ID |
|---|---|---|
| `originals/01-opening.mp4` | `07-opening.png` | `d88b3c0c-d68a-4d12-96b0-415c90889d8a` |
| `originals/02-durian.mp4` | `01-durian.png` | `e5d686dc-b828-4e37-8d6a-c839a54aa0fc` |
| `originals/03-melon.mp4` | `02-melon.png` | `59a8740a-2e55-482b-86b0-0c01aab5c5e3` |
| `originals/04-amla.mp4` | `03-amla.png` | `e6c6bf98-9440-491a-8158-a4eeec7e11c2` |
| `originals/05-guava.mp4` | `04-guava.png` | `e7452d02-443e-4f79-ace1-b10affd21d32` |
| `originals/06-wampee.mp4` | `05-wampee.png` | `451cf63b-617c-4dbc-8b4d-13da7feb4334` |
| `originals/07-noni.mp4` | `06-noni.png` | `93a5103f-74da-4a6f-bbee-d2b6d5cf006f` |
| `originals/08-ending.mp4` | `08-ending.png` | `ae0a6bd6-6f46-498a-ad6d-01b5c9a8c84e` |

## 附录 F：本次交接时的文件核验

以下 SHA-256 于整理文档时从本机文件读取，可用于复制后核对；未来重建容器字节可能变化。

| 文件 | 大小（字节） | SHA-256 |
|---|---:|---|
| `output/fruit-personification-grok/fruit-personification-30s-voiced.mp4` | 19606652 | `d382befa5b314f71bea7935f3e5e03ee3dc82a936e8edbef74b0a45e365e9d0e` |
| `output/fruit-personification-grok/fruit-personification-30s-silent.mp4` | 19163411 | `81aa0518b67764a9ddb1887b061bc94d013d3a8ad1e40fb51e815704facd21ad` |
| `output/fruit-personification-grok/voiceover/narration-30s.wav` | 2880078 | `5260fb8f3aafe3e50501322330467b5c098478fe8a55f4e288b4454c957ffd08` |
| `output/fruit-personification-grok/voiceover/narration-30s.mp3` | 721196 | `cf6d31f3af06bed227f777da1646e3387dd943f6e4109fa0396a72dff6bd3a59` |
| `output/fruit-personification-v2/01-durian.png` | 2402544 | `eba7ebc4647aeb3216a5ade37cdfadce83b9095ed30602a5bd698bebdb17c61b` |
| `output/fruit-personification-v2/02-melon.png` | 2633337 | `056a734ba3843fe0cd65661ff9d73ea4bf24241e0d8e7a463068b653486c3a61` |
| `output/fruit-personification-v2/03-amla.png` | 2337581 | `833152fd78cb04fbe9d1a30a79d192e2a53931b23ec32eb66ef354d767c852af` |
| `output/fruit-personification-v2/04-guava.png` | 2409760 | `235cdd15e6fcd241ed6a83c89ac91ae9aad10e5c88a1f490c80049fa9ec0c919` |
| `output/fruit-personification-v2/05-wampee.png` | 2403129 | `6d1200277197e7a9f508c34c83f080d4223b15b9fe459e6576bd019a4f60c7d8` |
| `output/fruit-personification-v2/06-noni.png` | 2423255 | `9384c43bbae89c58228b5bcd26e35aba670748af3543bc680870b926ae959d39` |
| `output/fruit-personification-v2/07-opening.png` | 2522918 | `68e5c0a7e8ec08d43369658b26e521e1fe6f6dac6715fcfb200a5540f49f2a40` |
| `output/fruit-personification-v2/08-ending.png` | 2603763 | `b39dbb0f3bc1c9a6bcab8d88afba5b4a7920a4b644472319c0f38b3ddc6b0cac` |
| `output/fruit-personification-grok/originals/01-opening.mp4` | 5805447 | `3da901059b839749f41ed69c101d5bf3598e83b31771cfbdf6e42f820c78f2c8` |
| `output/fruit-personification-grok/originals/02-durian.mp4` | 3882690 | `a8a24ed89b8d7b103dcb93cbdd63bd6e98f2c9921097c54fef0468cf9d02a094` |
| `output/fruit-personification-grok/originals/03-melon.mp4` | 4275391 | `19b2044b9888627099cfa7ed66907557afe277d747a9c07fb2fbc7c83f9aa2d2` |
| `output/fruit-personification-grok/originals/04-amla.mp4` | 2936744 | `a1e1209b4aac3a72efe4e716d905a3100efdc58fe4873d4ea450ca903d89f8fd` |
| `output/fruit-personification-grok/originals/05-guava.mp4` | 4336856 | `53be91d8c6fa2ca4368e925277cf61632a36c9fba29d7fba9ccfe31a7b6bb9be` |
| `output/fruit-personification-grok/originals/06-wampee.mp4` | 5371735 | `430406da4fee41bb20f7cf34c5aa593efdd9881f13860a0b6e5c00c373466dd2` |
| `output/fruit-personification-grok/originals/07-noni.mp4` | 4179654 | `e6bd8aa4a5c533a556f93b8e893dfa82bad57acc704fdf4395ef75ba215334eb` |
| `output/fruit-personification-grok/originals/08-ending.mp4` | 4484730 | `5f040d998040109a9263119aaf26088c987b4f9e7731ffea61d9c761ecfc3a98` |

制作参考：本次曾按本机 `C:/Users/pc/.codex/skills/ai-short-drama-production/SKILL.md` 的版本记录和验收原则执行；用户明确要求后期配音覆盖其默认原生声画路线。浏览器操作遵循当前 CUA 工具文档。此手册不含密码、Cookie、API key 或支付授权。
