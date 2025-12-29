# NotCoreWebz - 预配置权限的 .NET 运行容器

# 使用ARG传递.NET版本
ARG DOTNET_VERSION=10.0

# 基础镜像阶段
FROM mcr.microsoft.com/dotnet/aspnet:${DOTNET_VERSION}-alpine AS base

# 创建应用用户和组
ARG USER_ID=1000
ARG GROUP_ID=1000

RUN addgroup -g $GROUP_ID appgroup && \
    adduser -D -u $USER_ID -G appgroup appuser

# 创建工作目录
WORKDIR /app

# 预先创建所有需要的目录
RUN mkdir -p \
    /app/bin \
    /app/config \
    /app/certs \
    /app/Log \
    /app/wwwroot \
    /app/data \
    /app/temp \
    /app/uploads

# 设置所有权和完全写入权限（所有目录 775）
RUN chown -R appuser:appgroup /app && \
    chmod -R 775 /app

# 环境变量
ENV APP_DLL=NotApp.dll
ENV TZ=Asia/Shanghai

# 暴露端口
EXPOSE 8080

# 切换到应用用户
USER appuser

# 启动应用
ENTRYPOINT dotnet "${APP_DLL}"