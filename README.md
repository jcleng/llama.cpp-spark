# myai

> 本地 Spark X2.5 大模型推理环境：模型文件、llama.cpp 推理引擎与 Ollama 推理服务的集成目录。

本目录是一个围绕 **Spark X2.5** 大语言模型（LLM）的本地推理环境，整合了「模型文件」「C/C++ 推理引擎（llama.cpp）」与「Go 推理服务（Ollama）」三部分内容，可用于模型加载、GGUF 转换、推理运行与模型服务部署。

---

## 目录结构（`tree -L 2`，已排除各子目录 .git）

```
.
├── .vscode
│   └── settings.json
├── llama.cpp-spark
│   ├── .devops
│   ├── .gemini
│   ├── .github
│   ├── .pi
│   ├── app
│   ├── benches
│   ├── ci
│   ├── cmake
│   ├── common
│   ├── conversion
│   ├── docs
│   ├── examples
│   ├── ggml
│   ├── gguf-py
│   ├── grammars
│   ├── include
│   ├── licenses
│   ├── media
│   ├── models
│   ├── pocs
│   ├── requirements
│   ├── scripts
│   ├── skills
│   ├── src
│   ├── tests
│   ├── tools
│   ├── vendor
│   ├── .clang-format
│   ├── .clang-tidy
│   ├── .dockerignore
│   ├── .ecrc
│   ├── .editorconfig
│   ├── .flake8
│   ├── .gitignore
│   ├── .gitmodules
│   ├── .pre-commit-config.yaml
│   ├── AGENTS.md
│   ├── AUTHORS
│   ├── CLAUDE.md
│   ├── CMakeLists.txt
│   ├── CMakePresets.json
│   ├── CODEOWNERS
│   ├── CONTRIBUTING.md
│   ├── LICENSE
│   ├── Makefile
│   ├── README.md
│   ├── SECURITY.md
│   ├── build-xcframework.sh
│   ├── convert_hf_to_gguf.py
│   ├── convert_hf_to_gguf_update.py
│   ├── convert_llama_ggml_to_gguf.py
│   ├── convert_lora_to_gguf.py
│   ├── flake.nix
│   ├── mypy.ini
│   ├── pyproject.toml
│   ├── pyrightconfig.json
│   ├── requirements.txt
│   └── ty.toml
├── ollama-spark
│   ├── .github
│   ├── anthropic
│   ├── api
│   ├── app
│   ├── auth
│   ├── build
│   ├── cmake
│   ├── cmd
│   ├── discover
│   ├── docs
│   ├── envconfig
│   ├── format
│   ├── fs
│   ├── harmony
│   ├── integration
│   ├── internal
│   ├── llama
│   ├── llm
│   ├── logutil
│   ├── manifest
│   ├── middleware
│   ├── ml
│   ├── mlx
│   ├── model
│   ├── openai
│   ├── parser
│   ├── progress
│   ├── readline
│   ├── runner
│   ├── scripts
│   ├── server
│   ├── template
│   ├── thinking
│   ├── tokenizer
│   ├── tools
│   ├── types
│   ├── version
│   ├── x
│   ├── .dockerignore
│   ├── .gitattributes
│   ├── .gitignore
│   ├── .golangci.yaml
│   ├── AGENTS.md
│   ├── CLAUDE.md
│   ├── CMakeLists.txt
│   ├── CMakePresets.json
│   ├── CONTRIBUTING.md
│   ├── Dockerfile
│   ├── LICENSE
│   ├── LLAMA_CPP_VERSION
│   ├── MLX_C_VERSION
│   ├── MLX_VERSION
│   ├── README.md
│   ├── SECURITY.md
│   ├── go.mod
│   ├── go.sum
│   ├── main.go
│   └── ollama
├── Modelfile.spark
├── README.md
└── Spark-X2.5-4B-Q4_K_M.gguf
```

## 组件关系

```
        ┌─────────────────────────────────────────────┐
        │              Spark-X2.5 模型                  │
        │   Spark-X2.5-4B-Q4_K_M.gguf  (2.5 GB, Q4_K)  │
        │      ↑ 由 Modelfile.spark 声明来源             │
        └────────────────────────┬────────────────────┘
                                 │ 加载该 GGUF 权重
          ┌─────────────────────┼─────────────────────┐
          ▼                      ▼                      ▼
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │ llama.cpp-spark│     │ ollama-spark  │     │ Modelfile.spark│
  │ (C/C++ 推理)  │     │  (Go 服务/推理)│     │ (模型文件定义) │
  └──────────────┘     └──────────────┘     └──────────────┘
```

- **模型文件**是核心资产，由 `Modelfile.spark` 统一声明，被 `llama.cpp-spark` 与 `ollama-spark` 两个推理引擎共同加载与使用。
- **llama.cpp-spark** 提供轻量、高性能的本地推理能力（CPU/GPU 后端），适合直接生成文本。
- **ollama-spark** 提供统一的模型管理与 REST 服务能力，便于通过 API 调用与多模型/多集成集成。

---

## 运行中的进程

当前目录下存在一个正在运行的 GGUF 推理服务进程，由 **`ollama-spark`** 项目编译的 `llama-server`（基于 llama.cpp）加载 Spark 模型进行推理，持续为模型服务提供运行时。

| 项目 | 信息 |
| --- | --- |
| 进程 ID | 237086 |
| 进程父级 | 186267 |
| 所属项目 | `ollama-spark`（`build/llama-server-local/bin/llama-server`） |
| 运行用户 | root |
| 运行状态 | `Sl+`（多线程睡眠，前台进程组） |
| CPU 占用 | 338% |
| 内存占用 | 约 25.2%（RSS ≈ 40.6 GB，VSZ ≈ 109.2 GB） |
| 运行时长 | 约 2 小时 51 分 28 秒 |

**运行命令与配置：**

```sh
/home/jcleng/work/mywork/myai/ollama-spark/build/llama-server-local/bin/llama-server \
    -m ./Spark-X2.5-4B-Q4_K_M.gguf \
    -ngl 99 \
    --port 8000 --host 0.0.0.0 \
    -c 128000
```

- **模型加载**：`-m ./Spark-X2.5-4B-Q4_K_M.gguf` —— 加载 myai 目录下的 Spark X2.5 4B 参数、Q4_K 量化 GGUF 权重文件。
- **显存加速**：`-ngl 99` —— 将 99 层模型权重 offload 到 GPU，实现全量 GPU 推理。
- **服务监听**：`--port 8000 --host 0.0.0.0` —— 监听 8000 端口，对所有网络接口开放，可用于 REST API 调用。
- **上下文长度（128k 关键参数）**：`-c 128000` —— **体现 128k 上下文**，设定模型支持的最大上下文长度为 128,000（128k = 128 × 1000）token，即模型可同时处理约 12.8 万 token 的上下文内容进行生成与推理，满足长文档、长对话的上下文窗口需求。

该进程持续占用较高 CPU 与内存资源（瞬时 CPU 338%、RSS 约 40.6 GB），表明 Spark 模型的推理服务正在正常运行中，且具备 128k 长上下文支持能力。如需停止该服务，可终止进程 PID 237086，或通过 8000 端口对应的服务管理器停止运行。

---

## 使用说明

1. **推理（llama.cpp-spark）**
   - 先根据需求编译 `llama.cpp-spark`（CPU 或 NVIDIA GPU 后端）。
   - 确认模型权重与 tokenizer 文件就位后，使用 `llama-completion` / `llama-generate` 等命令进行推理，可指定上下文、采样参数等。
2. **推理与调用（ollama-spark）**
   - 启动 Ollama 服务，加载模型。
   - 通过命令行或 REST API（`/api/chat`、`/api/generate` 等）调用模型，支持流式输出与 OpenAI 兼容接口。

> 详细构建、转换与运行命令请参考各子目录内的 `README.md`。

---

## 许可证

- **llama.cpp-spark**：MIT License（[opensource.org/licenses/MIT](https://opensource.org/licenses/MIT)）
- **ollama-spark / Ollama**：MIT License（[opensource.org/licenses/MIT](https://opensource.org/licenses/MIT)）
- **Spark X2.5 模型权重**：请参考模型原始来源的许可证，使用前需遵守对应授权要求。

---

## 备注

- 本目录中的模型权重文件较大（约 2.5 GB），建议根据存储条件合理放置，并注意文件完整性校验。
- 两个推理引擎（llama.cpp 与 Ollama）可独立使用，也可按场景选择其一，或组合使用完成推理与服务部署。

## 模型下载

### GGUF 量化模型（llama.cpp / Ollama 直接加载）

| 模型 | 下载地址 | 包含文件 |
| --- | --- | --- |
| Spark-X2.5-4B-GGUF | [ModelScope](https://www.modelscope.cn/models/XHToken/Spark-X2.5-4B-GGUF) | `Spark-X2.5-4B-Q4_K_M.gguf`（约 2.48 GB）、`Spark-X2.5-4B-Q8_0.gguf`（约 4.17 GB）、`Spark-X2.5-4B.gguf`（FP16/BF16，约 7.85 GB） |
| Spark-X2.5-1.7B-GGUF | [ModelScope](https://www.modelscope.cn/models/XHToken/Spark-X2.5-1.7B-GGUF) | `Spark-X2.5-1.7B-Q4_K_M.gguf`（约 1.06 GB）、`Spark-X2.5-1.7B-Q8_0.gguf`（约 1.74 GB）、`Spark-X2.5-1.7B.gguf`（FP16/BF16，约 3.26 GB） |

> 本目录已包含两个 `Q4_K_M` 量化权重：`Spark-X2.5-4B-Q4_K_M.gguf`、`Spark-X2.5-1.7B-Q4_K_M.gguf`。

### 原始模型（BF16，用于转换或非 GGUF 推理框架）

| 模型 | 下载地址 | 说明 |
| --- | --- | --- |
| Spark-X2.5-4B | [ModelScope](https://modelscope.cn/models/XHToken/Spark-X2.5-4B) | 4B 参数原始权重，原生支持最长 1M token 上下文 |
| Spark-X2.5-1.7B | [ModelScope](https://modelscope.cn/models/XHToken/Spark-X2.5-1.7B) | 1.7B 参数原始权重，原生支持最长 1M token 上下文 |

### 下载方式

```sh
# 方式一：ModelScope CLI（推荐，可断点续传）
pip install modelscope
modelscope download --model XHToken/Spark-X2.5-4B-GGUF --local_dir ./Spark-X2.5-4B-GGUF

# 方式二：直接下载单个 GGUF 文件
wget https://www.modelscope.cn/models/XHToken/Spark-X2.5-4B-GGUF/resolve/master/Spark-X2.5-4B-Q4_K_M.gguf
```

> 注：`modelscope.cn` 与 `www.modelscope.cn` 两个域名等价，均可访问。四个模型均为 Apache-2.0 许可证。
