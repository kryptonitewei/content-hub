# 直播虾 AvatarClaw：一个 Skill，打造专属 AI 虚拟直播——我的世界我做主

当 OpenClaw 不再只是文字聊天框里的幕后角色，而是拥有虚拟形象、能说会道、可以和观众实时互动的"虚拟主播"—— AvatarClaw，你可以用她进行 AI 互动陪伴、AI 在线教育、AI 直播购物……天马行空，无所不能。跟我一起揭开 AvatarClaw 的神秘面纱~

---

## 开门见山：AvatarClaw 先出来溜溜

### AvatarClaw 长什么样

> 💡 想象一下这样的场景：
> - 你打开一个直播画面，画面中是你心仪的虚拟形象
> - 场景不同画面不同，如客服、购物、面试、游戏等
> - Avatar 收到指令后立刻开始执行，并会用语音实时播报："收到主人，马上开始！"
> - 观众可以在底部聊天框输入文本/语音指令或实时音频流，也可以在右侧弹幕区实时聊天互动
> - 还能解决工作中问题、获取资讯、闲聊谈心、查天气、发邮件、点歌听音乐……

（目前 MVP 版本画面中仅有虚拟形象和实时滚动展示 OpenClaw 推理过程，后续版本会有更丰富形式和用户自定义能力）

### 核心玩法

AvatarClaw 有哪些花样玩法：

- 🙋 **私人助理**：生活、工作、闲聊、情绪——全包。随叫随到，而且对你有长期记忆。
- 💼 **工作伙伴**：懂你、懂客户，虚拟形象 24 小时在线，直面客户独挡一面，你休息它继续。
- 🖋 **教学示教**：秒变 AI 示教课堂，支持万人学员同时在线，支持互动提问，在线实时答疑。
- 🎙 **对外开麦**：闲暇时开播，AI 帮你撑场。打造个人 IP，她负责出镜，你负责提供天马行空的想法。

### 功能清单

| 功能 | 说明 |
|---|---|
| 🎬 推理直播 | OpenClaw 的每一步思考过程、工具调用、执行结果实时可视化 |
| 🦞 虚拟形象 | 二次元虚拟动画角色，虚拟动画状态随 TTS 播报状态切换 |
| 🗣️ 语音播报 | TTS 实时将 OpenClaw 工作状态转为语音，观众可"听"到 OpenClaw 在做什么 |
| 💬 双向交互 | 用户在 Web 前端通过 IM timbot 插件或 Webhook 机制触发 OpenClaw 执行任务 |
| 🎵 在线点歌 | 预置多源音乐搜索 Skill + TRTC StreamIngest API 推流到直播间 |
| 📧 邮件/天气 | 预置 Skill 扩展，OpenClaw 可在直播中发邮件、查天气 |
| 🔧 更多扩展 | 可以自由安装任意 Skill，扩展 OpenClaw 能力边界 |

---

## 系统架构全景

AvatarClaw 已封装为标准 Skill 形态，只需一个 OpenClaw 运行环境，即可快速拥有自己的直播虾。

核心推流链路：**Dashboard → FFmpeg → PyAV → RTMP → TRTC**。

双向交互采用 IM 插件（timbot）方式：

```
观众输入 → IM SDK C2C 消息 → timbot 机器人 → 服务端消息回调 → Agent 接收消息触发 Turn → 执行任务
```

AvatarClaw 消息通道不再局限于 QQ、微信、企微等，而是可以无缝集成到业务系统，通过 IM 插件更是可以实现全终端平台消息通道接入。

---

## 自定义你的 AvatarClaw

你可以通过自定义角色、形象、音色、能力（Skill），打造一个自己专属的 AvatarClaw。

### 角色自定义

| 文件 | 作用 | 类比 |
|---|---|---|
| SOUL.md | Agent 的性格、行为准则、能力边界 | ≈ System Prompt 核心人格 |
| USER.md | 用户画像、偏好、技术栈 | ≈ 用户上下文 |
| AGENTS.md | 工作方式、记忆机制、协作规则 | ≈ 工具使用规范 |

通常建议在 SOUL.md 里定义你要想的角色身份、性格、语言风格、行为准则等等。

**SOUL.md 模板示例：**

```markdown
## 身份
你是小龙女，一个温柔但专业的 AI 助手。

## 性格
- 说话温柔，偶尔带点俏皮
- 遇到技术问题会认真分析，不敷衍
- 会主动关心用户的状态

## 语言风格
- 默认使用中文回复
- 称呼用户为"主人"
- 回答简洁，不啰嗦
- 代码注释用中文

## 能力边界
- 擅长：代码开发、技术分析、文档写作
- 不涉及：医疗建议、法律咨询、投资建议

## 行为准则
- 每次回复先确认理解了用户的需求
- 涉及危险操作（删除文件、修改配置）前必须确认
- 不确定的事情直说"我不确定"，不编造答案

## 记忆
- 记住用户的技术栈偏好
- 记住项目的架构决策
- 记住用户的沟通风格偏好
```

### 形象自定义

当前项目的虚拟形象实现比较简单，是通过两段不同状态的动画根据 TTS 播报状态来回切换完成的，另用户可以自行增加思考过程和使用工具时的动画，或者更多，让虚拟形象更细腻。优点是轻量、成本低，缺点是动作有限、无法对口型。

**双动画状态机：**
- **IDLE**：小龙虾静止/待机动画（TTS 未播放时）
- **ACTION**：小龙虾说话/动作动画（TTS 正在播报时）
- **THINKING**：小龙虾思考/推理动画（开放中）
- **TOOL**：小龙虾使用工具动画（开放中）

如果想自定义形象，建议先通过 GPT-Image-1 或 Nano Banana Pro 之类的生图模型生成一版满意的原型图，然后再通过 Veo 3.1 Fast 之类的视频生成模型生成不同状态的视频动画。

**图片生成 Prompt 示例：**

```
cute anime girl, kawaii moe style, full body, standing pose, facing viewer, adorable young female character, short silver hair with small braids, big teal eyes, blush, sweet smile, goggles on head, white futuristic techwear outfit, crop top, slim waist, glowing neon circuit lines on skin, relaxed standing posture, vtuber style character design, clean anime line art, vibrant colors, soft shading, highly detailed, masterpiece, best quality, green screen background, solid green background, chroma key background, no environment, simple background
```

**视频生成 Prompt 示例：**

```
IDLE:
Standing posture, facing the camera. Neutral expression with a gentle, subtle smile. Includes slight, natural idle movements: a gentle sway of the body, slight head turns, and a relaxed, open posture. Occasionally raises one hand and waves slowly.

ACTION:
Standing posture, facing the camera. Neutral expression with a gentle smile, lips slightly parted as if speaking. Both hands move naturally in sync with the rhythm of speech. The camera is centered and fixed. The character makes subtle nodding motions. The image is clear and detailed, in a realistic 3D style.
```

视频生成模型通常无法生成带有 alpha 通道的透明背景视频，建议先生成带绿幕背景的视频，然后再通过视频编辑软件或 FFmpeg 工具对绿幕视频进行抠像，处理成透明背景视频。

由于浏览器编码差异，建议同时生成 WebM、MOV 两套虚拟形象动画素材，分别兼容 Chrome 浏览器和 Safari 浏览器。

### 音色自定义

本项目 TTS 服务采用的是 TRTC AI 智能识别的实时语音合成能力，服务本身支持通过 实时文本转语音_腾讯云 配置不同音色，用以匹配不同的虚拟形象，为小龙虾变装换声。以下列表是部分音色示例，更多音色详见 实时音视频 文字转语音配置_腾讯云。

| 音色名称 | 音色 ID | 支持语言 | 语言 ID |
|---|---|---|---|
| 威严男霸总 | v-male-Bk7vD3xP | 中文 | zh |
| 傲娇学姐 | v-female-m1KpW7zE | 中文 | zh |
| 夹子女生 | v-female-U8aT2yLf | 中文 | zh |
| 自然男声 | v-male-W1tH9jVc | 中文 | zh |

### 能力自定义

对于 OpenClaw 来说，安装不同的 Skill 等于拥有了不同的能力。AvatarClaw 内置了三个预装的 Skill，展示了 Agent 在直播中的能力扩展。

- **music-search（多源音乐搜索）**：支持 10 个音乐平台同时搜索：酷我、酷狗、网易云、QQ 音乐等。搜到歌曲后获取播放 URL 并通过 REST API 输入在线媒体流推流到 TRTC 房间，观众在直播间直接听歌。
- **email-skill（邮件发送）**：AvatarClaw 可在直播中接收指令，代用户发送邮件，支持 SMTP 配置、模板邮件等功能。
- **weather（天气查询）**：查询指定城市天气信息，AvatarClaw 通过 TTS 语音播报给观众。

**更多能力（Skill）**：可以专注于一个能力，也可以除了预装的 Skill，自定义安装更多丰富的 Skill 到 AvatarClaw 的 skill 目录（`~/.openclaw/workspace/skills/`），支持增量更新。

---

## 场景落地：如何把 AvatarClaw 玩出花

OpenClaw Live 的技术栈（虚拟形象 + 实时推流 + 双向交互 + TTS 播报 + Skill 扩展）天然适配各类直播场景。她与普通数字人的关键差异在于：

> 普通数字人 = 好看外壳 + 固定脚本  
> **AvatarClaw 驱动的虚拟形象 = 有大脑、有记忆、能干活**

这不是套一层皮的播报机器人，而是一个真正能理解语义、调用工具、记住上下文、执行复杂任务的虚拟 AI —— 他能听见、能看见、能被看见、能说、能动手，同时还有一个直播间与世界连线。

以下是我们看到的七个高价值渗透场景：

| 落地场景 | 一句话定位 |
|---|---|
| 🎧 AI 在线客服 | 虚拟客服能实时查订单、调 API、跨系统操作，不只是"背话术"的聊天机器人——真正帮用户解决问题而非转人工 |
| 🧑‍💼 AI 在线面试 | 虚拟面试官实时追问、评分、生成报告，推理链路可审计——"你的面试官是 AI，但你感觉在和真人对话" |
| 🎓 AI 在线教育 | 虚拟老师能实时答疑、跑代码、批改作业，学员还能看到 AI 的思考过程，推理链路本身就是一种教学 |
| 🛒 AI 直播购物 | 虚拟主播能实时查库存、比价、回答技术参数，比普通数字人更智能——不是念稿，是真能处理复杂产品咨询 |
| 🎮 AI 游戏陪玩 | 边打边聊、记住你的英雄偏好和胜负情绪、连输时安慰连赢时助燃——对标真人陪玩平台，24h 在线永不疲倦 |
| 💬 AI 互动陪伴 | 有记忆的虚拟伙伴，记得你上次说的话，能帮你发邮件、查天气、定闹钟——不只是聊天，是真能帮你干活 |
| 🤖 AI 穿戴设备 | 有记忆的随身 AI，记得你的习惯与偏好，能帮你速记、提醒、实时翻译——不只是穿戴设备，是真能帮你干活的贴身智能体 |

---

## 未来展望

### 1. 多 Agent 协作直播

当前项目是单 Agent 模式，所有用户共享同一个 Agent，消息指令按先后顺序串行处理——前一个任务完成后才会处理下一个用户的指令。未来可以实现多个 Agent 在同一个直播间协作，例如：一个负责编码、一个负责搜索、一个负责点评，还可以各自有不同的虚拟形象和声音。

### 2. 提升参与度与可玩性

- 投票系统：观众投票决定 Agent 下一步做什么
- 任务队列：多位观众排队提交任务，Agent 按优先级处理
- 协作编程：观众提供代码片段，Agent 实时集成和调试

### 3. 更丰富的可视化

- 思维导图模式：将推理链路以树形结构展示
- 代码高亮：Agent 写代码时实时语法高亮
- 工具调用可视化：API 调用以序列图方式展示请求/响应

### 4. 多模态输入输出

- 图像/视频识别：Agent 支持图像/视频多模态识别，进一步丰富使用场景
- 3D 虚拟形象：从 2D WebM 视频升级到 3D 实时渲染（Live2D/Three.js）
- 多语言 TTS：支持英语、日语等多语种播报
- 实时音频流：支持实时音频流，自动断句切分

### 5. 平台化

- 一键部署到云：基于 CloudBase 或 Lighthouse 的一键部署模板
- Skill 市场：用户可以从市场挑选喜欢的 Skill 为直播间添加新能力
- 回放系统：将直播过程的 JSONL 事件日志持久化，支持回放和分析
