# AI 接口测速工具

中文 · [English](README_EN.md)

测试一个大模型 API 要等多久才开始回复、输出有多快，以及缓存命中和估算成本，帮助你在相同条件下对比不同接口。

**使用前提：** Python 3.10+，以及你自己的模型接口地址、API Key 和模型名称。真实测速需要联网并可能产生费用；在线演示使用模拟数据。

[查看演示](https://ferretgeek.github.io/llm-api-benchmark/) · [本地启动](#本地启动) · [测量方法](docs/测量方法.md)

测速按固定场景逐条发送请求，将首字延迟、输出速度和用量分开统计；预热消耗计入成本，不计入正式速度结果。

## 界面

![面板界面](docs/images/dashboard.png)

## 你会得到什么

- **三种锁定的单流场景** — 无缓存极限、日常连续对话、Codex 高缓存。**不会把并发吞吐伪装成单请求速度。**
- **看得见过程** — 实时看板、质量门禁、逐请求明细、历史可比性判断，以及一次性 JSON 报告。
- **口径分得清** — 完整 usage 与中断估算严格分开；**预热会计入额度，但不进入正式速度分布。**
- **四套主题** — 鸢尾、青玉、晨曦三套浅色配色，以及背景精确为 `#17191d` 的全局暗色主题。
- **哪都能跑** — Windows 一键脚本、本地 Python、Docker Compose，以及 Linux systemd + SSH 隧道。

## 本地启动

需要 Python 3.10 或更高版本。

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements.txt
.\.venv\Scripts\python token_stress_test.py
```

Windows 也可以在装完依赖后双击 `一键启动.bat`。`一键结束服务.bat` 会先确认端口上的服务确实是本程序，**避免误杀你的其他进程**。

默认访问 [http://127.0.0.1:18976](http://127.0.0.1:18976)。

## Docker 与服务器

```powershell
Copy-Item .env.docker.example .env
docker compose up --build -d
```

Compose 只把端口映射到宿主机回环地址。远程服务器请通过 SSH 隧道访问：

```text
ssh -L 18976:127.0.0.1:18976 user@example.com
```

完整的 Docker、systemd、升级与备份步骤见[部署指南](docs/部署指南.md)。**不要把含 API Key 的面板直接暴露到公网。**

## 多端点命令行对比

为每个端点建一个**不会提交**的 `*.local.txt`，依次写入 HTTPS 接口地址、API Key、模型名称三行，然后显式传入：

```powershell
.\.venv\Scripts\python benchmark_token_speed.py `
  --configs endpoint-a.local.txt endpoint-b.local.txt `
  --mode daily-dialogue --samples 6 --warmups 1
```

程序**不会自动寻找凭据文件**。结果只记录配置文件名、模型和测量数据，不记录 Key 或 Base URL。

## 技术上值得一提的地方

**预热计费但不计分。** 预热请求会真实消耗额度（所以成本里算上了它），但不会进入正式的速度分布——否则第一次冷启动会把整组数据拉低，你得到的就是一个不可复现的数字。这两件事必须分开记。

**完整 usage 与中断估算是两个字段。** 请求正常结束时用上游返回的完整 `usage`；中途中断时给出估算值，并**明确标为估算**。把两者混进一列平均值是这类工具最常见的错误。

**WebSocket 要求同源 + 本机会话令牌。** 令牌由启动器生成。仅靠同源检查不够——一个本机端口在多用户机器上是所有用户都能访问的。

**端点地址会被校验。** 远程上游强制 HTTPS；**包含账号信息、查询参数或 fragment 的接口地址会被拒绝**，避免凭据经由 URL 泄露到日志里。

**定价表的边界写清楚了。** 仓库内置的是「标准服务层、输入不超过 272K token」的静态 API 等效价格快照，只用于统一换算，**不代表你的套餐账单**。超长上下文、Batch、Flex、Priority 和区域处理都不套用这张表；真实价格以 [OpenAI 官方定价](https://developers.openai.com/api/docs/pricing)为准。

**测试和截图不打真接口。** 自动化测试、Demo 和项目截图都不会调用真实模型——所以 CI 不烧钱，截图也不含真实响应。

## 隐私与安全

API Key 和 Base URL 只在当前浏览器会话与服务进程内存中使用，**不写入草稿、历史、报告或运行日志**；模型输出正文也不进入预览、历史或报告。

服务默认只监听 `127.0.0.1`。

> **这个工具会按你的操作向上游 API 发起真实请求，可能产生费用。** 请先阅读[隐私说明](PRIVACY.md)与[测量方法](docs/测量方法.md)。

## 它不做什么

- 不做并发压测排行榜（那会让单请求速度失去意义）。
- 不代理、不缓存、不改写你的模型请求内容。
- 不保存 API Key、Base URL 或模型输出正文。
- 不提供模型、额度或任何形式的接口服务。

## 更多文档

[部署指南](docs/部署指南.md) · [测量方法](docs/测量方法.md) · [隐私说明](PRIVACY.md) · [发布审计](docs/发布审计.md) · [版本变更](CHANGELOG.md) · [参与开发](CONTRIBUTING.md) · [安全策略](SECURITY.md) · [第三方声明](THIRD_PARTY_NOTICES.md)

## 许可与声明

见 [LICENSE](LICENSE)。

这是独立的社区项目，与 OpenAI 没有隶属、授权或背书关系，也不绕过任何额度限制。
