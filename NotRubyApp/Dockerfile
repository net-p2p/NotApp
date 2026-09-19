# NotRuby - 预配置权限的 Ruby 运行容器

# 使用ARG传递Ruby版本
ARG RUBY_VERSION=3.3
# 创建应用用户和组
ARG USER_ID=1000
ARG GROUP_ID=1000

# 基础镜像阶段
FROM ruby:${RUBY_VERSION} AS base

# 检测操作系统并安装相应包
RUN if [ -f /etc/alpine-release ]; then \
        apk add --no-cache \
        tzdata; \
    else \
        apt-get update && apt-get install -y --no-install-recommends \
        tzdata && \
        rm -rf /var/lib/apt/lists/*; \
    fi

# 环境变量

# 运行主程序（rb 文件名）
ENV APP_RB=NotApp.rb
# 设置中国时区
ENV TZ=Asia/Shanghai
# 中文语言
ENV LC_ALL=zh_CN.UTF-8
ENV LANG=zh_CN.UTF-8
# Ruby 编码
ENV RUBYOPT=-EUTF-8
# 容器默认端口 8080
ENV SERVER_PORT=8080
# 表示是在容器中运行
ENV RUBY_RUNNING_IN_CONTAINER=true

# 设置时区
RUN ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo "${TZ}" > /etc/timezone

# 需要重新声明 ARG 才能在这个阶段使用
ARG USER_ID
ARG GROUP_ID

# 创建用户组（兼容 Alpine 和 Debian）
RUN set -eux; \
    if [ -f /etc/alpine-release ]; then \
        # Alpine：BusyBox 自带 addgroup/adduser
        if ! getent group ${GROUP_ID} >/dev/null 2>&1; then \
            addgroup -g ${GROUP_ID} appgroup; \
        fi; \
        if ! getent passwd ${USER_ID} >/dev/null 2>&1; then \
            GROUP_NAME="$(getent group ${GROUP_ID} | cut -d: -f1)"; \
            adduser -D -u ${USER_ID} -G "$GROUP_NAME" appuser; \
        fi; \
    else \
        # Debian：先装 passwd 包，再用绝对路径调用
        apt-get update && apt-get install -y --no-install-recommends passwd; \
        rm -rf /var/lib/apt/lists/*; \
        if ! getent group ${GROUP_ID} >/dev/null 2>&1; then \
            /usr/sbin/groupadd -g ${GROUP_ID} appgroup; \
        fi; \
        if ! getent passwd ${USER_ID} >/dev/null 2>&1; then \
            /usr/sbin/useradd -u ${USER_ID} -g ${GROUP_ID} -m -s /bin/bash appuser; \
        fi; \
    fi

# 创建工作目录
WORKDIR /app

# 预先创建所有需要的目录
RUN mkdir -p \
    /app/Bin \
    /app/Log \
    /app/Temp

# 设置所有权和完全写入权限（所有目录 775）
RUN chown -R ${USER_ID}:${GROUP_ID} /app && \
    chmod -R 775 /app

# 暴露端口
EXPOSE 8080

# 切换到应用用户
USER ${USER_ID}

# 指定挂载数据
VOLUME ["/app"]

# 启动应用 支持参数传递
ENTRYPOINT ["/bin/sh", "-c", "ruby \"${APP_RB}\" \"$@\"", "--"]