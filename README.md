[README.md](https://github.com/user-attachments/files/29800120/README.md)
# PullDockerYourself

自建 Docker Registry 镜像加速与会员管理系统，为 fnOS/NAS 场景设计。支持将 Docker 镜像同步到本地目录或 WebDAV 存储，以 Registry v2 兼容接口提供拉取服务，含用户专属加速链接、套餐流量控制、限速和后台管理。

> GitHub: https://github.com/cjm2004/PullDockerYourself

## 功能

- Registry v2 拉取兼容
- 测试链接 `/test` + 用户专属链接 `/<后缀>`
- Docker Hub / registry-v2 / mirror-only 多上游
- `docker pull` → `docker save` 同步 mirror-only 源
- 直接粘贴完整 `docker pull` 命令同步
- 本地目录与 WebDAV 存储（如 123 云盘）
- 镜像版本管理——增删改、AI 补充说明、自定义图标
- 同步任务实时速度显示
- 用户注册/登录、套餐购买、订单审核
- 免费/赠送/套餐流量分桶，赠送用完再扣套餐计时
- 测试链接和用户专属链接独立限速
- 邮箱验证码、站内消息
- 后台备份/恢复
- 深色模式，手机端适配
- fnOS FPK 一键安装

## 技术栈

| 层 | 技术 |
|---|---|
| 后端 | Python 3 标准库 |
| 前端 | Vue 3 SPA |
| 数据库 | JSON 文件 |
| 存储 | 本地目录 / WebDAV |
| 打包 | fnOS FPK |

## 目录

```text
backend/server.py         # 后端
frontend/index.html       # Vue SPA
frontend/vue.min.js       # Vue 运行时
fpk/cmd/                  # fnOS 生命周期脚本
fpk/config/               # 运行权限
fpk/manifest              # 包元信息
ui/                       # 图标素材
```

## 安装

将 FPK 上传飞牛 fnOS 后台安装。默认端口 `8088`，需 root 权限（访问 Docker daemon）。

```text
com.cjm2004.mirror-1.0.0.fpk
```

## 首次登录

| 项 | 值 |
|---|---|
| 用户名 | admin |
| 密码 | admin123 |

建议首次登录后立即修改密码。

## 管理员配置流程

1. 进入管理面板
2. 添加目标源（本地目录或 WebDAV）
3. 添加上游源（Docker Hub / registry-v2 / mirror-only）
4. 在「上游同步」中同步镜像到目标源
5. 在「镜像管理」查看、编辑版本
6. 在「设置」配置域名、限速、邮箱、支付等

## 同步镜像

### 从上游同步

```text
镜像名：library/mysql
tag：latest
```

非官方镜像必须完整命名空间：

```text
镜像名：halohub/halo        ✅
镜像名：halo                ❌
镜像名：library/halo        ❌
```

### 多镜像同步

镜像名和 tag 多行一一对应：

```text
library/mysql
halohub/halo
library/nginx
```

```text
latest
2.25.4
1.25
```

### 通过 docker pull 命令同步

```bash
docker pull docker.jiaxin.site/halohub/halo:2.25.4
```

系统自动解析 registry、镜像名、tag 并在 NAS 本机执行。

## 用户使用

开通套餐后在控制台开启专属加速源，获得专属链接：

```text
https://your-nas.example.com/user01
```

Docker pull **不能**带 `https://`：

```bash
# 正确
docker pull your-nas.example.com/user01/library/mysql:latest

# 错误
docker pull https://your-nas.example.com/user01/library/mysql:latest
```

### 配置 Docker 镜像加速

```bash
sudo nano /etc/docker/daemon.json
```

```json
{
  "registry-mirrors": ["https://your-nas.example.com/user01"]
}
```

```bash
sudo systemctl restart docker
```

## WebDAV / 本地源扫描

推荐目录结构：

```text
halohub/halo/2.25.4/image.tar
library/mysql/latest/manifest.json
library/nginx/1.25/manifest.v2.json
```

扫描器识别版本目录中包含以下标记文件时生成镜像记录：

```text
image.tar
manifest.json
manifest.v2.json
meta.json
```

## 常见问题

**docker pull 提示 permission denied**
```bash
sudo usermod -aG docker <用户名>
# 重新登录生效
```

**docker pull 返回 NOT_FOUND**
检查镜像名、tag、是否使用了 `/test` 或用户专属后缀。

**docker pull 不能带 https://**
Docker 镜像引用不含协议头。

**halo 镜像同步失败**
必须用 `halohub/halo`，不是 `library/halo` 或 `halo`。

**WebDAV 大文件上传失败**
检查服务商文件大小限制、网络、目标目录权限。

## 打包

```bash
# 构建 app.tgz
tar -czf app.tgz backend frontend ui com.cjm2004.mirror.Application

# 构建 FPK
tar -czf com.cjm2004.mirror-版本号.fpk manifest ICON.PNG ICON_256.PNG app.tgz cmd config wizard
```

## 注意

- 需要 root 权限访问 Docker daemon
- mirror-only 上游依赖 NAS 本机 Docker
- JSON 数据库存储在应用数据目录
- WebDAV 性能取决于服务商和网络
- 非官方镜像必须完整命名空间
