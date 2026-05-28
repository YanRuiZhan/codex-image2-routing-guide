# codex-image2-routing-guide

让 Codex 在**普通对话**时继续走你当前正在使用的 API / 中转站，而在你**明确调用 `/image2` 或 `$image2`** 时，单独走你自己的图片生成中转站。

详细配置说明见：[`ccswitch-image2-routing-guide.md`](./ccswitch-image2-routing-guide.md)

## 这个仓库解决什么问题

很多人现在的使用方式是：

- 日常和 Codex 聊天、改代码：走当前默认 API 或当前现有中转站
- 在需要生图时：切到另一个支持图片的 OpenAI 兼容接口

这个仓库的目标就是把这两条链路分开：

- **普通 Codex 对话**：不受影响
- **`/image2` 图片生成**：单独读取私密配置文件，走你的图片 API
- 最主要的是边聊天边提要求生图，适合用image2制作ppt的工作流
## 你可以直接发给 Codex 的提示词

下面这段提示词的用途，是让 Codex **自动帮你完成 image2 技能的本地配置**。

使用方法：

1. 先把本仓库里的 `ccswitch-image2-routing-guide.md` 发给 Codex，或者让 Codex读取这个文件
2. 再把下面这段提示词原样发给 Codex
3. 按 Codex 的提示提供你自己的 `image2-env.txt` 路径

### 自动配置提示词

```text
请按照我提供的这份文档，为我的 Codex 本地安装并配置一个 image2 技能：

- 技能名固定为 image2
- 只在我明确调用 /image2 或 $image2 时使用
- 普通图片请求不要自动改走这个中转站
- 普通 Codex 对话继续使用我当前正在使用的 API / 中转站，不要去修改它

请你严格按文档执行，并满足下面要求：

1. 先问我：我希望把 image2 的私密配置文件 image2-env.txt 放在哪个文件夹
2. 只允许读取我明确告诉你的那个 image2-env.txt 文件
3. 不要扫描、猜测、复用我当前 Codex 正在使用的 API key、Base URL、config.toml、环境变量、ccswitch 配置或浏览器缓存
4. 如果缺少目录，就帮我创建 image2 技能目录和脚本目录
5. 根据文档内容写好 SKILL.md 和 generate_image2.py
6. 输出目录推荐单独放在我桌面的一个固定 AIGC 文件夹中，并提醒我要不要更改AI生图输出路径
7. 配置完成后，做一次最小化验证：
   - 先不要用真实图片提示词大量出图
   - 只验证技能结构、脚本参数、配置读取逻辑是否正常
8. 最后明确告诉我：
   - image2 现在读取的是哪个配置文件
   - /image2 图片请求会走哪条 URL
   - 普通 Codex 对话是否完全不受影响
   - 我下一次该怎么触发这个技能

如果你需要模板、路径规则、SKILL.md 示例、Python 脚本示例，就以我给你的文档为准执行。
```

## 最终效果

配置完成后，实际效果应该是这样的：

### 1. 普通对话不变

你平时继续直接对 Codex 说：

```text
帮我解释这段代码
帮我改这个 bug
帮我写一个 Python 脚本
```

这些请求还是走你当前默认的聊天链路，不会切换到 image2 的图片中转站。

### 2. 只有明确调用 `/image2` 才走图片链路

例如你这样说：

```text
/image2 画一个极简风格的蓝白配色应用图标，中心是闪电，纯色背景
```

或者：

```text
$image2 生成一张横版 banner，科技感、深色背景、发光线条
```

这时才会：

- 调用 `image2` 技能
- 读取你指定的 `image2-env.txt`
- 使用其中的 `URL`、`API_KEY`、`MODEL`
- 将图片保存到你的本地输出目录
- 在回复里展示图片预览和输出路径

### 3. 两套路由彻底分离

最终你得到的是下面这种结构：

```text
普通 Codex 对话
-> 继续走你原来的官方 API / 原中转站

/image2 或 $image2
-> 读取你指定的 image2-env.txt
-> 走你自己的图片中转站 / ccswitch 本地端口 / OpenAI 图片接口
```

## 仓库内容

- [`ccswitch-image2-routing-guide.md`](./ccswitch-image2-routing-guide.md)：完整配置指南
- `README.md`：快速说明、自动配置提示词、最终效果

## 适用场景

适合这些用户：

- 聊天和图片想走不同 API
- 已经在用 ccswitch，希望只把图片流量单独分流
- 希望把图片 key 和聊天 key 完全隔离
- 希望在 Codex 对话中随时通过 `/image2` 出图

## 注意事项

- 不要把真实 `image2-env.txt` 提交到 Git 仓库
- 不要在公开文档里写真实 API key
- 不要让 Codex 自动扫描你现有的聊天 API 配置
- 如果你的图片路由需要本地 ccswitch 端口，请先确认本地端口已经可用
