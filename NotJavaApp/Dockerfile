# NotJavaApp - 预配置权限的 Java 运行容器

# 使用ARG传递Java版本
ARG JAVA_VERSION=17
# 创建应用用户和组
ARG USER_ID=1000
ARG GROUP_ID=1000

# 基础镜像阶段
FROM amazoncorretto:${JAVA_VERSION} AS base

# 检测操作系统并安装相应包 - 完整全球化支持
RUN if [ -f /etc/alpine-release ]; then \
        apk add --no-cache \
        icu-libs \
        icu-data-full \
        tzdata; \
    fi

# 环境变量

# 运行主程序（jar 包名）
ENV APP_JAR=NotApp.jar
# 设置中国时区
ENV TZ=Asia/Shanghai
# 中文语言
ENV LC_ALL=zh_CN.UTF-8
ENV LANG=zh_CN.UTF-8
# Java 编码 + 时区
ENV JAVA_OPTS="-Dfile.encoding=UTF-8 -Duser.timezone=Asia/Shanghai"
# 容器默认端口 8080
ENV SERVER_PORT=8080
# 表示是在容器中运行
ENV JAVA_RUNNING_IN_CONTAINER=true

# 设置时区
RUN ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo "${TZ}" > /etc/timezone

# 需要重新声明 ARG 才能在这个阶段使用
ARG USER_ID
ARG GROUP_ID

# 创建用户组（兼容Alpine和Debian/AmazonLinux）
RUN set -eux; \
    \
    # 确保 GID 1000 存在
    if ! getent group ${GROUP_ID} >/dev/null 2>&1; then \
        if [ -f /etc/alpine-release ]; then \
            addgroup -g ${GROUP_ID} appgroup; \
        else \
            groupadd -g ${GROUP_ID} appgroup; \
        fi; \
    fi; \
    \
    # 确保 UID 1000 存在（名字不重要）
    if ! getent passwd ${USER_ID} >/dev/null 2>&1; then \
        if [ -f /etc/alpine-release ]; then \
            GROUP_NAME="$(getent group ${GROUP_ID} | cut -d: -f1)"; \
            adduser -D -u ${USER_ID} -G "$GROUP_NAME" appuser; \
        else \
            useradd -u ${USER_ID} -g ${GROUP_ID} -m -s /bin/bash appuser; \
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
#ENTRYPOINT java ${JAVA_OPTS} -jar "${APP_JAR}"
ENTRYPOINT ["/bin/sh", "-c", "java ${JAVA_OPTS} -jar \"${APP_JAR}\" \"$@\"", "--"]
