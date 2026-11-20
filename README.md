# aistudio-oai-bridge

给 [aistudio-api](https://github.com/DeconstructedCube/aistudio-api) 套一层 OpenAI 格式的翻译，让只认 OpenAI 接口的酒馆插件（比如写摘要的）也能用上它。

- 只用 Python 自带的库，一个文件，不用装任何东西
- 只在本机 `127.0.0.1:8765` 听，转给本机 `127.0.0.1:8080` 的 aistudio-api
- 钥匙原样转发，不存、不发给任何别的地方
- 四类安全设置全开到 `BLOCK_NONE`
- 流式请求（`stream: true`）接 aistudio-api 的流式口，边生成边吐；思考阶段会发保活注释，免得客户端以为断了

## Termux 安装

先把 aistudio-api 装好跑起来，然后一次粘：

```sh
curl -fsSL https://raw.githubusercontent.com/meowmana/aistudio-oai-bridge/main/oai-bridge.py -o ~/oai-bridge.py
sed -i '1i curl -sf -m 2 127.0.0.1:8765/health >/dev/null || (nohup ~/aistudio-api/.venv/bin/python ~/oai-bridge.py >~/oai-bridge.log 2>&1 &)' ~/.bashrc
nohup ~/aistudio-api/.venv/bin/python ~/oai-bridge.py >~/oai-bridge.log 2>&1 &
sleep 1; curl -s 127.0.0.1:8765/health
```

- 国内下载 GitHub 文件常连不上，记得开梯子
- aistudio-api 没装在 `~/aistudio-api` 的，把第二行里的 python 路径改成你自己的

最后一行看到 `"ok": true` 就是起来了。以后每次打开 Termux 它会自己起，日志在 `~/oai-bridge.log`。

## 电脑上（Linux / macOS / Windows）

程序本身不挑系统，有 Python 3.9 以上就能跑。下载 `oai-bridge.py`，在它所在的文件夹里：

```sh
python3 oai-bridge.py      # Linux / macOS
python oai-bridge.py       # Windows
```

看到 `oai-bridge on http://127.0.0.1:8765/v1` 就是起来了，窗口别关。aistudio-api 不在本机 8080 的话，先设 `OAI_BRIDGE_UPSTREAM` 指过去（Windows PowerShell 里是 `$env:OAI_BRIDGE_UPSTREAM="http://..."`）。

## 插件里怎么填

- 接口地址：`http://127.0.0.1:8765/v1`
- API Key：aistudio-api 的 `config.yaml` 里 `api_keys` 那把
- 模型：能直接拉列表，比如 `gemini-2.5-flash`

## 可调的环境变量

| 变量 | 默认 |
|---|---|
| `OAI_BRIDGE_UPSTREAM` | `http://127.0.0.1:8080` |
| `OAI_BRIDGE_PORT` | `8765` |
| `OAI_BRIDGE_HOST` | `127.0.0.1` |

## 自检

```sh
python check.py
```

会起一个假的 Gemini 端，把模型列表、错钥匙、普通对话、流式各走一遍。
