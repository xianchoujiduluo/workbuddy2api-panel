# Repository Guidelines

## 项目结构与模块职责

本项目是 Go 编写的 OpenAI 兼容网关，版本要求见 `go.mod`（当前 Go 1.22.5）。

- `cmd/server/`：服务入口、配置读取与模块装配；`cmd/login/`、`cmd/signin/`、`cmd/credit/`、`cmd/trial/`：辅助命令。
- `internal/server/`：HTTP 接口；`internal/upstream/`：上游协议；`internal/pool/`：账号池；`internal/session/`、`internal/scheduler/`：会话与调度。
- `internal/panel/`：管理面板及通过 `go:embed` 嵌入的 `index.html`、`app.js`；无需独立前端构建。
- 测试以 `*_test.go` 与源码同目录；`scripts/` 存放 Python 辅助脚本；`.github/workflows/` 管理 CI 与发布。

## 构建、测试与本地运行

以下命令在仓库根目录执行，需要 Go 与已配置的 `~/.local/bin/capped`。资源密集任务执行前确认本机资源；默认限制内存 6G、CPU 8 核、进程数 512，OOM 时先降低并发。

```bash
export GOMAXPROCS=4 GOFLAGS=-p=4
capped go build -o wb2api ./cmd/server                 # 构建服务
capped go run ./cmd/server -config config.json        # 本地运行
capped go test ./... -parallel=4                      # 运行已有测试
CAP_MEM=2G CAP_CPU=4 capped go vet ./...               # 静态检查
CAP_MEM=2G CAP_CPU=4 capped gofmt -w path/to/file.go    # 格式化修改的 Go 文件
```

服务默认端口为 `7863`，面板路径为 `/panel/`，健康检查为 `/healthz`。首次运行可自动生成配置；Redis 为可选依赖。

## 编码风格与命名

Go 使用 `gofmt` 的制表符缩进，包名小写，导出标识符使用 PascalCase，内部标识符使用 camelCase，文件名沿用 `snake_case.go`。JavaScript 沿用两空格缩进与 camelCase；复用现有模块，保持接口和业务行为一致。新增前端代码遵守源码目标 300 行、组件 200 行、函数 50 行的限制，超过 500 行须拆分。面板遵守现有 CSP，避免内联脚本和事件属性。

## 测试规范

使用 Go 标准库 `testing`、`net/http/httptest`，测试函数命名为 `TestXxx`，基准测试为 `BenchmarkXxx`。定位后端问题可运行 `capped go test ./internal/pool -run TestPick -parallel=4`。CI 执行完整 Go 测试；部分面板测试依赖 Node，缺失时跳过。目前未配置覆盖率门槛。优先运行相关已有检查，未经要求不新增测试功能。

## 提交与 Pull Request

历史采用 `fix(server): 中文说明`、`docs(panel): 中文说明` 等格式，并引用 issue。当前提交按用户规范使用分类前缀：`[✨ feat ]`、`[🐞 fix  ]`、`[⚡️ perf ]`、`[📚 docs ]`、`[🎨 style]`、`[🔧 chore]`、`[🛠️ build]`、`[✅ test]`，说明必须中文，例如 `[🐞 fix  ] 修正账号冷却状态`。

PR 应说明问题、行为变化、关联 issue 和实际验证结果；界面修改附截图，配置或持久化变化说明兼容性。未经明确授权不提交配置文件。

## 配置安全与代理约束

勿提交 `config.json`、`auths/`、`data/`、`backups/` 或真实密钥。配置字段变化需保持后端、面板表单与文档一致。仅处理授权范围；文档任务不修改源码。前端验证仅允许 `pnpm lint`，不得构建或启动前端；本仓库没有 `package.json`，不要自行安装依赖或引入工具链。
