# NotXxx 系列容器

一套预配置权限的多语言运行容器集合。每个容器都是「空容器模板」，内置非 root 用户、中文时区、目录权限、挂载卷，你只需要把自己的应用放进去就能跑。

---

## 包含哪些容器

| 语言 | 目录 | 镜像名 | 版本矩阵 | 基础镜像 |
|------|------|--------|----------|----------|
| .NET | `NotApp` | `notapp` | 6.0 / 7.0 / 8.0 / ... | `mcr.microsoft.com/dotnet/aspnet` |
| Java | `NotJavaApp` | `notjava` | 8 / 11 / 17 / 21 / 25 | `amazoncorretto` |
| Python | `NotPythonApp` | `notpython` | 3.9 / 3.10 / 3.11 / 3.12 / 3.13 | `python` |
| Node | `NotNodeApp` | `notnode` | 20 / 22 / 24 | `node` |
| Ruby | `NotRubyApp` | `notruby` | 3.1 / 3.2 / 3.3 / 3.4 | `ruby` |
| Deno | `NotDenoApp` | `notdeno` | latest | `denoland/deno` |
| Go | `NotGoApp` | `notgo` | 无 | `alpine` |

> 命名规则：**目录带 `App`，镜像名不带。**

---

## 设计原则

所有容器遵循同一套设计，保证部署体验一致：

| 原则 | 说明 |
|------|------|
| **非 root 运行** | 默认创建 UID/GID 1000 的 `appuser`，以普通用户身份启动 |
| **中文环境** | 时区 `Asia/Shanghai`，语言 `zh_CN.UTF-8`，编码 `UTF-8` |
| **统一目录** | `/app/Bin`、`/app/Log`、`/app/Temp`，权限 775 |
| **数据卷** | `/app` 声明为 VOLUME |
| **端口 8080** | 所有容器默认暴露 8080 |
| **参数透传** | `docker run` 时镜像名后面的参数会追加到启动命令之后 |

---

## 快速开始

以 Java 为例，其他语言完全同理。

### 1. 拉取镜像

```bash
docker pull ni-xue/notjava:17
```

### 2. 运行应用

```bash
docker run -d \
  --name myapp \
  -p 8080:8080 \
  -v /宿主机/应用目录:/app \
  ni-xue/notjava:17
```

### 3. 自定义入口

```bash
docker run -d \
  -e APP_JAR=myapp.jar \
  -v /宿主机/应用目录:/app \
  ni-xue/notjava:17
```

### 4. 追加参数

镜像名后面直接写，会自动追加：

```bash
docker run -d \
  -e APP_JAR=myapp.jar \
  -p 4808:4808 \
  -v /宿主机/应用目录:/app \
  ni-xue/notjava:17 \
  --server.port=4808
```

---

## 各语言入口变量对照

| 语言 | 入口变量 | 默认值 | 运行命令 |
|------|----------|--------|----------|
| .NET | `APP_DLL` | `NotApp.dll` | `dotnet NotApp.dll` |
| Java | `APP_JAR` | `NotApp.jar` | `java -jar NotApp.jar` |
| Python | `APP_PY` | `NotApp.py` | `python NotApp.py` |
| Node | `APP_JS` | `NotApp.js` | `node NotApp.js` |
| Ruby | `APP_RB` | `NotApp.rb` | `ruby NotApp.rb` |
| Deno | `APP_TS` | `NotApp.ts` | `deno run --allow-all NotApp.ts` |
| Go | `APP_BIN` | `NotApp` | `/app/NotApp` |

---

## 镜像标签规范

每个语言的大版本标签对应基础系统：

| 标签形式 | 基础系统 | 示例 |
|----------|----------|------|
| `{版本}` | 默认（Debian / Amazon Linux） | `notjava:17`、`notpython:3.12` |
| `{版本}-alpine` | Alpine | `notjava:17-alpine`、`notpython:3.12-alpine` |

> Go 例外：没有版本矩阵，标签为 `alpine`。

---

## CI 自动构建

每个语言有独立的 GitHub Actions workflow，手动触发，支持选择是否推送到 Docker Hub。

```
.github/workflows/
├── build-java.yml
├── build-python.yml
├── build-node.yml
├── build-ruby.yml
├── build-deno.yml
└── build-go.yml
```

**手动触发方式**：仓库 Actions 页面 → 选择对应 workflow → Run workflow。

**可配置项**：

| 输入 | 说明 | 默认值 |
|------|------|--------|
| `versions` | 要构建的版本范围 | 各语言不同 |
| `push` | 是否推送到 Docker Hub | `true` |

---

## 自己构建单个镜像

以 Java 为例：

```bash
docker build \
  --build-arg JAVA_VERSION=17 \
  -t notjava:17 \
  NotJavaApp/
```

其他语言把 `JAVA_VERSION` 换成对应的 `PYTHON_VERSION`、`NODE_VERSION`、`RUBY_VERSION`、`DENO_VERSION`。

---

## 目录结构

```
仓库根/
├── .github/
│   └── workflows/
│       ├── build-java.yml
│       ├── build-python.yml
│       ├── build-node.yml
│       ├── build-ruby.yml
│       ├── build-deno.yml
│       └── build-go.yml
├── NotApp/
│   └── Dockerfile
├── NotJavaApp/
│   └── Dockerfile
├── NotPythonApp/
│   └── Dockerfile
├── NotNodeApp/
│   └── Dockerfile
├── NotRubyApp/
│   └── Dockerfile
├── NotDenoApp/
│   └── Dockerfile
└── NotGoApp/
    └── Dockerfile
```

---

## 环境变量一览

所有容器共享的通用变量：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `TZ` | `Asia/Shanghai` | 时区 |
| `LC_ALL` / `LANG` | `zh_CN.UTF-8` | 语言编码 |
| `SERVER_PORT` | `8080` | 应用端口（需应用自身读取） |

各语言特有的入口变量和编码变量见上表。

---

## 关键提醒

**1. 挂载会覆盖 `/app`**

`-v /宿主机/目录:/app` 会完全覆盖容器内的 `/app`，所以 jar、二进制、脚本必须放在宿主机挂载目录里。

**2. 参数分工**

- **运行时参数**（如 JVM 的 `-Xmx`）→ 环境变量（如 `JAVA_OPTS`）
- **应用参数**（如 `--server.port`）→ 镜像名后面直接跟

**3. Go 需要静态编译**

Go 二进制要用 `CGO_ENABLED=0` 编译，才能在 Alpine 里跑。典型多阶段构建：

```dockerfile
FROM golang:1.23-alpine AS builder
WORKDIR /build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o NotApp .

FROM ni-xue/notgo:alpine
COPY --from=builder /build/NotApp /app/NotApp
```

**4. Action 版本**

CI 里用的是 `actions/checkout@v7`、`docker/setup-buildx-action@v4`、`docker/login-action@v4`，已适配 Node 24。如果你的仓库里这些版本不可用，回退到 `@v4`、`@v3` 也能跑。

---

## 扩展新语言

想加一个新语言（比如 PHP、Perl），按这个模式复制即可：

1. 新建目录 `NotPhpApp/`
2. 写 Dockerfile，遵守统一设计原则（非 root、中文时区、`/app` 目录、VOLUME、参数透传）
3. 新建 `.github/workflows/build-php.yml`，参考现有 workflow 改语言和版本矩阵
4. 更新本 README 的表格

---

## 一句话总结

**一套容器模板，覆盖七种语言，部署任何应用都是「拉镜像 + 挂目录 + 加参数」三步。**

---
