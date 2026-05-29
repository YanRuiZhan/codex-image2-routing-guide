# Codex / image2 图片路由配置指南

这份文档用于帮助用户把“普通对话”和“图片生成”分开路由。当用户在使用codex对话中要求使用/image2技能生成图片时，走另一个中转站的图片路由：

- 普通 Codex 对话：继续使用用户当前正在使用的官方 API 或现有中转站。
- 图片生成：在明确调用 `/image2` 或 `$image2` 技能时，使用用户自己配置的图片中转站 API key，请求模型默认为gpt-image2而非其它。

文档不包含任何真实 API key 或私人中转站地址。请用户自行填写自己的 `URL` 和 `API_KEY`。

## 一、先理解三种可选路由

不同用户的网络和中转站能力不同，不一定都需要 ccswitch 本地路由。

### 方案 A：直连中转站，推荐优先尝试

适合中转站域名可直接访问的用户。

请求链路：

```text
Codex image2 技能 -> https://你的中转站域名/v1 -> 中转站 -> OpenAI 兼容图片接口
```

`image2-env.txt` 里这样填：

```text
URL=https://your-relay.example.com/v1
API_KEY=sk-your-relay-api-key
MODEL=gpt-image-2
```

### 方案 B：通过 ccswitch 本地端口转发

适合已经用 ccswitch 管理多个 API key、模型供应商或本地转发端口的用户。

请求链路：

```text
Codex image2 技能 -> http://127.0.0.1:15721/v1 -> ccswitch -> 中转站 -> OpenAI 兼容图片接口
```

`image2-env.txt` 里这样填：

```text
URL=http://127.0.0.1:15721/v1
API_KEY=sk-your-ccswitch-or-relay-api-key
MODEL=gpt-image-2
```

注意：本地端口不一定是 `15721`，请以用户自己的 ccswitch 配置为准。

### 方案 C：官方 OpenAI API

适合用户有官方 OpenAI API key，并且希望图片也走官方接口。

请求链路：

```text
Codex image2 技能 -> https://api.openai.com/v1 -> OpenAI
```

`image2-env.txt` 里这样填：

```text
URL=https://api.openai.com/v1
API_KEY=sk-your-openai-api-key
MODEL=gpt-image-2
```

## 二、创建私密配置文件 image2-env.txt

这一节不要直接套用作者电脑上的路径。配置前应先询问用户：

```text
你希望把 image2 的密钥文件放在哪个文件夹？
例如可以放在桌面的 your_file 文件夹：
C:\Users\<你的用户名>\Desktop\your_file\secrets\image2-env.txt
```

推荐规则：

- 让用户自己指定一个固定文件夹，例如 `your_file`、`codex_private`、`ai_secrets`。
- 密钥文件名建议固定为 `image2-env.txt`。
- 文件夹和文件名确定后，后续 `SKILL.md`、脚本、测试命令都必须使用同一个路径。
- 不要让 Codex 自动扫描当前正在使用的 Codex API key 或 URL。
- 不要从 `config.toml`、ccswitch 配置、环境变量或浏览器缓存里猜测图片 API 配置。

如果用户自定义文件夹为 `your_file`，密钥文件为 `image2-env.txt`，则推荐路径为：

```text
C:\Users\<你的用户名>\Desktop\your_file\secrets\image2-env.txt
```

如果 Windows 用户名是 `Alice`，路径就是：

```text
C:\Users\Alice\Desktop\your_file\secrets\image2-env.txt
```

创建目录：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\Desktop\your_file\secrets"
```

创建文件：

```powershell
notepad "$env:USERPROFILE\Desktop\your_file\secrets\image2-env.txt"
```

可以先给用户一个“虚假密钥模板.txt”，让用户照着填自己的真实信息。模板内容如下：

```text
URL=https://your-relay.example.com/v1
API_KEY=sk-your-own-image-api-key
MODEL=gpt-image-2
```

也可以使用其它分隔符，但最推荐一行一个字段，使用换行分隔，格式保持 `KEY=value`。例如：

```text
URL=https://your-relay.example.com/v1
API_KEY=sk-your-own-image-api-key
MODEL=gpt-image-2
RESPONSES_MODEL=gpt-5.5
```

字段说明：

- `URL`：OpenAI 兼容接口根路径，通常以 `/v1` 结尾。可以是官方 OpenAI、中转站，或 ccswitch 本地端口。
- `API_KEY`：用户自己的图片生成专用 key。
- `MODEL`：图片模型，默认建议 `gpt-image-2`。
- `RESPONSES_MODEL`：用于调用 `/responses` 的模型。多数用户可以不写；如果中转站要求特定模型名，再自行添加。

用户创建好文件后，可以把“文件路径”发给 Codex，例如：

```text
我已经创建好了密钥文件：
C:\Users\Alice\Desktop\your_file\secrets\image2-env.txt
请只读取这个文件来配置 image2，不要扫描我当前 Codex 正在使用的 API key 和 URL。
```

不要把 `image2-env.txt` 发给别人，不要提交到 Git 仓库。转发教程时只能给虚假模板，不要包含真实 API key 或私人中转站 URL。

## 三、安装 image2 技能

在 Codex skills 目录创建 `image2` 技能：

```text
C:\Users\<你的用户名>\.codex\skills\image2
```

目录结构：

```text
image2
├─ SKILL.md
└─ scripts
   └─ generate_image2.py
```

PowerShell 创建目录：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.codex\skills\image2\scripts"
```

## 四、SKILL.md 模板

把下面内容保存为：

```text
C:\Users\<你的用户名>\.codex\skills\image2\SKILL.md
```

```markdown
---
name: image2
description: Use only when the user explicitly invokes /image2, $image2, or links this image2 skill. Generate images through the user's configured OpenAI-compatible image relay API using the private local image2-env.txt config. Do not use this relay for ordinary image requests unless this skill is explicitly invoked.
---

# image2

仅在用户明确调用 `/image2`、`$image2` 或点名本技能时使用。普通图片请求不要自动走这个中转站。

## 配置

- 私有配置文件：`C:\Users\<你的用户名>\Desktop\your_file\secrets\image2-env.txt`
- 输出目录：`C:\Users\<你的用户名>\Desktop\your_file\AIGC`
- 默认模型：`gpt-image-2`
- 默认尺寸：`1024x1024`
- 默认质量：`medium`
- 默认数量：`1`

不要在回复中展示 API key。

`image2-env.txt` 格式：

```text
URL=https://your-relay.example.com/v1
API_KEY=sk-your-own-api-key
MODEL=gpt-image-2
```

## 生成命令

请根据自己的 Python 路径调整命令。如果已经有可用 Python，通常可以直接用 `python`。

```powershell
python "$env:USERPROFILE\.codex\skills\image2\scripts\generate_image2.py" `
  --prompt "<提示词>" `
  --size "1024x1024" `
  --quality "medium" `
  --n 1
```

## 路由说明

- 如果 `image2-env.txt` 的 `URL` 是 `https://你的中转站/v1`，则图片直连中转站。
- 如果 `URL` 是 `http://127.0.0.1:端口/v1`，则图片通过 ccswitch 本地端口转发。
- 这只影响 `/image2` 或 `$image2` 技能，不应影响普通 Codex 对话。

## 参数推断

- 方图、头像、图标、未说明：`1024x1024`
- 竖版、手机壁纸、竖海报：`1024x1536`
- 横版、封面、banner、桌面壁纸：`1536x1024`
- 用户指定数量时传 `--n`，否则保持 `1`

## 回复结果

脚本成功后会打印保存路径。生成完毕后必须在回复中展示图片预览。
Markdown 图片预览必须使用正斜杠路径，避免 Windows 反斜杠被解析成转义字符。

```markdown
![image2 output](C:/Users/<你的用户名>/Desktop/your_file/AIGC/<file>.png)
```

并列出所有输出文件路径。
```

## 五、generate_image2.py 脚本模板

把下面内容保存为：

```text
C:\Users\<你的用户名>\.codex\skills\image2\scripts\generate_image2.py
```

```python
#!/usr/bin/env python3
"""Generate images through a private OpenAI-compatible image relay."""

from __future__ import annotations

import argparse
import base64
import datetime as dt
import json
import mimetypes
import os
from pathlib import Path
import re
import sys
from typing import Iterable
from urllib.parse import urlparse

import requests


HOME = Path.home()
DEFAULT_ENV_FILE = Path(os.environ.get("IMAGE2_ENV_FILE", HOME / "Desktop" / "your_file" / "secrets" / "image2-env.txt"))
DEFAULT_OUTPUT_DIR = Path(os.environ.get("IMAGE2_OUTPUT_DIR", HOME / "Desktop" / "your_file" / "AIGC"))


def load_env_file(path: Path) -> dict[str, str]:
    values: dict[str, str] = {}
    if not path.exists():
        return values
    for line in path.read_text(encoding="utf-8-sig").splitlines():
        line = line.strip()
        if not line or line.startswith("#") or "=" not in line:
            continue
        key, value = line.split("=", 1)
        values[key.strip()] = value.strip().strip('"').strip("'")
    return values


def config(env_file: Path) -> tuple[str, str, str, str]:
    file_values = load_env_file(env_file)
    base_url = (
        os.environ.get("IMAGE2_BASE_URL")
        or os.environ.get("URL")
        or file_values.get("IMAGE2_BASE_URL")
        or file_values.get("URL")
    )
    api_key = (
        os.environ.get("IMAGE2_API_KEY")
        or os.environ.get("API_KEY")
        or file_values.get("IMAGE2_API_KEY")
        or file_values.get("API_KEY")
    )
    model = (
        os.environ.get("IMAGE2_MODEL")
        or os.environ.get("MODEL")
        or file_values.get("IMAGE2_MODEL")
        or file_values.get("MODEL")
        or "gpt-image-2"
    )
    responses_model = (
        os.environ.get("IMAGE2_RESPONSES_MODEL")
        or os.environ.get("RESPONSES_MODEL")
        or file_values.get("IMAGE2_RESPONSES_MODEL")
        or file_values.get("RESPONSES_MODEL")
        or "gpt-5.5"
    )
    if not base_url:
        raise SystemExit(f"Error: URL is missing in env or {env_file}")
    if not api_key:
        raise SystemExit(f"Error: API_KEY is missing in env or {env_file}")
    return base_url.rstrip("/"), api_key, model, responses_model


def slugify(value: str) -> str:
    return re.sub(r"[^A-Za-z0-9._-]+", "-", value).strip("-") or "image2"


def extension_from_format(output_format: str) -> str:
    return {"png": "png", "jpeg": "jpg", "jpg": "jpg", "webp": "webp"}.get(output_format.lower(), "png")


def extension_from_url(url: str, default: str) -> str:
    suffix = Path(urlparse(url).path).suffix.lower().lstrip(".")
    if suffix in {"png", "jpg", "jpeg", "webp"}:
        return "jpg" if suffix == "jpeg" else suffix
    return default


def save_b64_image(value: str, path: Path) -> None:
    if "," in value and value.strip().startswith("data:"):
        value = value.split(",", 1)[1]
    path.write_bytes(base64.b64decode(value))


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--prompt", required=True)
    parser.add_argument("--reference-image", action="append", default=[], help="Path to a local reference image.")
    parser.add_argument("--size", default="1024x1024")
    parser.add_argument("--quality", default="medium", choices=["low", "medium", "high", "auto"])
    parser.add_argument("--n", type=int, default=1)
    parser.add_argument("--output-format", default="png", choices=["png", "jpeg", "webp"])
    parser.add_argument("--output-dir", type=Path, default=DEFAULT_OUTPUT_DIR)
    parser.add_argument("--env-file", type=Path, default=DEFAULT_ENV_FILE)
    parser.add_argument("--timeout", type=int, default=300)
    return parser.parse_args()


def stream_json(url: str, api_key: str, payload: dict[str, object], timeout: int) -> requests.Response:
    return requests.post(
        url,
        headers={
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json",
            "Accept": "text/event-stream",
        },
        json=payload,
        timeout=timeout,
        stream=True,
    )


def print_http_error(response: requests.Response) -> None:
    print(f"Error: HTTP {response.status_code}", file=sys.stderr)
    try:
        print(response.json(), file=sys.stderr)
    except ValueError:
        print(response.text[:2000], file=sys.stderr)


def mime_type_for_image(path: Path) -> str:
    mime_type, _ = mimetypes.guess_type(path.name)
    if mime_type in {"image/png", "image/jpeg", "image/webp", "image/gif"}:
        return mime_type
    return "image/png"


def encode_image_as_data_url(path: Path) -> str:
    mime_type = mime_type_for_image(path)
    b64 = base64.b64encode(path.read_bytes()).decode("ascii")
    return f"data:{mime_type};base64,{b64}"


def build_input(prompt: str, reference_images: Iterable[str]) -> str | list[dict[str, object]]:
    refs = [Path(p) for p in reference_images]
    if not refs:
        return prompt

    content: list[dict[str, str]] = [{"type": "input_text", "text": prompt}]
    for ref in refs:
        if not ref.exists():
            raise SystemExit(f"Error: reference image not found: {ref}")
        content.append({"type": "input_image", "image_url": encode_image_as_data_url(ref)})
    return [{"role": "user", "content": content}]


def extract_image_item(value: object) -> dict[str, str] | None:
    if not isinstance(value, str) or not value:
        return None
    if value.startswith("http://") or value.startswith("https://"):
        return {"url": value}
    return {"b64_json": value}


def response_image_items(data: dict[str, object]) -> list[dict[str, str]]:
    items: list[dict[str, str]] = []
    for output in data.get("output") or []:
        if not isinstance(output, dict) or output.get("type") != "image_generation_call":
            continue
        item = extract_image_item(output.get("result"))
        if item:
            items.append(item)
    return items


def streamed_image_items(response: requests.Response) -> list[dict[str, str]]:
    final_items: list[dict[str, str]] = []
    partial_items: list[dict[str, str]] = []
    event_name = ""
    for raw_line in response.iter_lines(decode_unicode=True):
        if not raw_line:
            event_name = ""
            continue
        if raw_line.startswith("event:"):
            event_name = raw_line.split(":", 1)[1].strip()
            continue
        if not raw_line.startswith("data:"):
            continue
        body = raw_line.split(":", 1)[1].strip()
        if body == "[DONE]":
            break
        try:
            data = json.loads(body)
        except json.JSONDecodeError:
            continue

        item = extract_image_item(data.get("partial_image_b64"))
        if item:
            partial_items.append(item)
            continue

        for key in ("b64_json", "result"):
            item = extract_image_item(data.get(key))
            if item:
                final_items.append(item)
                break
        else:
            item = None
        if item:
            continue

        item_obj = data.get("item")
        if isinstance(item_obj, dict):
            item = extract_image_item(item_obj.get("result"))
            if item:
                final_items.append(item)
                continue

        if event_name == "response.completed":
            response_obj = data.get("response")
            if isinstance(response_obj, dict):
                final_items.extend(response_image_items(response_obj))
    return final_items or partial_items


def main() -> int:
    args = parse_args()
    if not 1 <= args.n <= 10:
        print("Error: --n must be between 1 and 10.", file=sys.stderr)
        return 2

    base_url, api_key, model, responses_model = config(args.env_file)
    response_endpoint = f"{base_url}/responses"
    tool: dict[str, object] = {
        "type": "image_generation",
        "action": "generate",
        "size": args.size,
        "quality": args.quality,
        "output_format": args.output_format,
    }
    payload = {
        "model": responses_model,
        "input": build_input(args.prompt, args.reference_image),
        "tools": [tool],
        "tool_choice": {"type": "image_generation"},
        "stream": True,
    }
    response = stream_json(response_endpoint, api_key, payload, args.timeout)
    if response.status_code >= 400:
        print_http_error(response)
        return 1

    items = streamed_image_items(response)
    if len(items) > args.n:
        items = items[-args.n:]
    if not items:
        print("Error: response did not contain image data.", file=sys.stderr)
        return 3

    args.output_dir.mkdir(parents=True, exist_ok=True)
    timestamp = dt.datetime.now().strftime("%Y%m%d-%H%M%S")
    base_name = slugify(model)
    default_ext = extension_from_format(args.output_format)
    saved: list[Path] = []

    for index, item in enumerate(items, start=1):
        ext = default_ext
        path = args.output_dir / f"{base_name}-{timestamp}-{index:02d}.{ext}"
        if item.get("b64_json"):
            save_b64_image(item["b64_json"], path)
        elif item.get("url"):
            image_url = item["url"]
            ext = extension_from_url(image_url, default_ext)
            path = args.output_dir / f"{base_name}-{timestamp}-{index:02d}.{ext}"
            image_response = requests.get(image_url, timeout=args.timeout)
            image_response.raise_for_status()
            path.write_bytes(image_response.content)
        else:
            continue
        saved.append(path)

    if not saved:
        print("Error: no images saved.", file=sys.stderr)
        return 3

    print("Saved image(s):")
    for path in saved:
        print(path)
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

## 六、安装依赖

脚本依赖 `requests`。

如果 Python 已经能运行但缺少 `requests`：

```powershell
python -m pip install requests
```

如果用户使用 conda、uv 或其它 Python 环境，请安装到实际执行脚本的那个环境中。

## 七、验证配置

### 1. 验证 txt 文件能被读取

```powershell
python "$env:USERPROFILE\.codex\skills\image2\scripts\generate_image2.py" --help
```

只看帮助不会发起 API 请求。

### 2. 生成一张测试图

```powershell
python "$env:USERPROFILE\.codex\skills\image2\scripts\generate_image2.py" `
  --prompt "一张简单的蓝色小花插画" `
  --size "1024x1024" `
  --quality "medium" `
  --n 1
```

成功时会打印：

```text
Saved image(s):
C:\Users\<你的用户名>\Desktop\your_file\AIGC\gpt-image-2-YYYYMMDD-HHMMSS-01.png
```

测试成功后可以发给用户看看。

## 八、在 Codex 对话中使用

安装好 skill 后，用户可以这样调用：

```text
/image2 生成一张赛博风城市夜景，横版
```

或者：

```text
$image2 生成一张极简产品海报
```

推荐让 Codex 在回复中展示图片预览：

```markdown
![image2 output](C:/Users/<你的用户名>/Desktop/your_file/AIGC/<file>.png)
```

注意必须使用正斜杠 `/`，不要用 Windows 反斜杠 `\`，否则部分 Codex 预览面板会显示 `Unable to load file`。

## 九、ccswitch 用户的额外检查

如果选择方案 B，通过 ccswitch 本地端口转发，请检查：

1. ccswitch 正在运行。
2. 本地端口存在，例如 `127.0.0.1:15721`。
3. ccswitch 对该 API key 的路由能访问目标中转站。
4. `image2-env.txt` 的 `URL` 写成本地端口，例如：

```text
URL=http://127.0.0.1:15721/v1
```

检查端口：

```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 15721
```

如果端口不通，先修 ccswitch，不要先怀疑 image2 skill。

## 十、常见问题

### 1. `/v1/images/generations` 返回 404

很多中转站或 ccswitch 只支持 Responses 路径，不支持旧式图片路径。

本模板默认使用：

```text
POST /v1/responses
```

并通过 `image_generation` 工具生成图片，所以不会依赖 `/v1/images/generations`。

### 2. Cloudflare 524 timeout

图片生成时间较长时，非流式请求可能被 Cloudflare 超时断开。

本模板默认使用：

```json
"stream": true
```

这样可以降低长时间无响应导致的 524 风险。

### 3. 明明 `--n 1`，为什么出现多张图？

Responses 流式接口可能返回过程图和最终图。

本模板会优先保存最终图，并且当返回数量超过 `--n` 时，只保留最后 `n` 张。

### 4. 右侧预览无法加载

通常是 Markdown 图片路径用了反斜杠。

错误：

```markdown
![image](C:\Users\Alice\Desktop\your_file\AIGC\a.png)
```

正确：

```markdown
![image](C:/Users/Alice/Desktop/your_file/AIGC/a.png)
```

### 5. 普通对话会不会也走图片中转站？

不会，只要不要修改 Codex 主配置里的模型供应商。

这个 skill 只在用户明确调用 `/image2` 或 `$image2` 时运行脚本，脚本再读取 `image2-env.txt`。

普通聊天继续走用户原本的 Codex API 路由。

## 十一、安全提醒

- 不要把真实 `API_KEY` 写进 `SKILL.md`。
- 不要把真实 `API_KEY` 写进 `generate_image2.py`。
- 不要把 `image2-env.txt` 发给别人。
- 不要把 `image2-env.txt` 提交到 GitHub。
- 转发本文档给别人前，确认里面没有你的真实 URL 和 API key。
- 如果中转站 URL 也是私人地址，也请使用示例域名替代。

## 十二、给其它用户的最短配置清单

1. 先询问用户希望把密钥文件放在哪个文件夹，例如桌面的 `your_file`。
2. 创建 `C:\Users\<你的用户名>\Desktop\your_file\secrets\image2-env.txt`。
3. 在里面填自己的 `URL`、`API_KEY`、`MODEL`，一行一个字段。
4. 创建 `C:\Users\<你的用户名>\.codex\skills\image2\SKILL.md`。
5. 创建 `C:\Users\<你的用户名>\.codex\skills\image2\scripts\generate_image2.py`。
6. 安装 Python 依赖 `requests`。
7. 在 Codex 中用 `/image2 你的图片描述` 测试。


## 十三、最后提醒：image2 技能怎么用

配置完成后，只有在明确调用 `/image2` 或 `$image2` 时，才会走这条图片路由。

常用写法：

```text
/image2 生成一张赛博风城市夜景，横版
```

```text
$image2 画一张极简产品海报，方图
```

```text
/image2 生成一张手机壁纸，竖版，主题是雨夜霓虹街道
```

尺寸习惯：

- 不说明尺寸：默认 `1024x1024`。
- 说“横版、封面、banner、桌面壁纸”：建议用 `1536x1024`。
- 说“竖版、手机壁纸、竖海报”：建议用 `1024x1536`。
- 说“方图、头像、图标”：建议用 `1024x1024`。

生成完成后，Codex 应该回复：

1. 图片预览。
2. 图片保存路径。
3. 如果失败，简短说明是密钥、URL、端口、网络，还是模型路由问题。

预览图片时必须使用正斜杠路径，例如：

```markdown
![image2 output](C:/Users/Alice/Desktop/your_file/AIGC/gpt-image-2-20260528-120000-01.png)
```

不要这样写：

```markdown
![image2 output](C:\Users\Alice\Desktop\your_file\AIGC\gpt-image-2-20260528-120000-01.png)
```

最后再强调一次：普通聊天不要加 `/image2` 或 `$image2`；只有真正要生成图片时才调用这个技能。
