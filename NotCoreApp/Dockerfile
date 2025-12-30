# NotCoreWebz - 预配置权限的 .NET 运行容器

# 使用ARG传递.NET版本
ARG DOTNET_VERSION=10.0

# 基础镜像阶段
FROM mcr.microsoft.com/dotnet/aspnet:${DOTNET_VERSION} AS base

# 检测操作系统并安装相应包 - 完整全球化支持
RUN if [ -f /etc/alpine-release ]; then \
        apk add --no-cache \
        icu-libs \
        icu-data-full \
        tzdata \
        curl \
        bash; \
    fi

# 设置时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 创建应用用户和组
ARG USER_ID=1000
ARG GROUP_ID=1000

# 创建用户（兼容Alpine和Debian）
RUN if command -v addgroup >/dev/null 2>&1; then \
        # Alpine
        addgroup -g $GROUP_ID appgroup && \
        adduser -D -u $USER_ID -G appgroup appuser; \
    else \
        # Debian/Ubuntu
        groupadd -g $GROUP_ID appgroup && \
        useradd -u $USER_ID -g appgroup -m -s /bin/bash appuser; \
    fi

# 创建工作目录
WORKDIR /app

# 预先创建所有需要的目录
#RUN mkdir -p \
    #/app/bin \
    #/app/config \
    #/app/certs \
    #/app/Log \
    #/app/wwwroot \
    #/app/data \
    #/app/temp \
    #/app/uploads

# 设置所有权和完全写入权限（所有目录 775）
RUN chown -R appuser:appgroup /app && \
    chmod -R 775 /app

# 环境变量

# 运行主程序
ENV APP_DLL=NotApp.dll
# 设置中国时区
ENV TZ=Asia/Shanghai
# 中文语言
ENV LC_ALL=zh_CN.UTF-8
# 中文语言
ENV LANG=zh_CN.UTF-8
# 禁止全球化时区
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false
# 赋空值
ENV ASPNETCORE_HTTP_PORTS=''
# 表示是在容器中运行
ENV DOTNET_RUNNING_IN_CONTAINER=true

# 暴露端口
EXPOSE 8080

# 切换到应用用户
USER appuser

# 启动应用
ENTRYPOINT dotnet "${APP_DLL}"
