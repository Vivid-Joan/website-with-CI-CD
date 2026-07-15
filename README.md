# 静态网站自动化部署 (ECS)

本项目是一个简单的静态 HTML 页面，通过 GitHub Actions 实现自动化构建与部署。当代码推送到 `main` 分支时，CI/CD 流水线会自动将最新的文件同步至自建的 ECS 服务器。

## 目录结构

```text
.
├── .github/
│   └── workflows/
│       └── deploy.yml      # GitHub Actions 自动化部署脚本
├── index.html              # 网站主页
└── README.md               # 项目说明文档
```

## ️ 自动化部署流程

本项目使用 GitHub Actions 进行持续部署（CD），核心流程如下：

1. **触发机制**：当向 `main` 分支推送代码（Push）时自动触发。
2. **环境准备**：通过 SSH 连接到目标 ECS 服务器，检查并安装必要的依赖（如 `rsync`），确保目标目录存在且权限正确。
3. **文件同步**：使用 `easingthemes/ssh-deploy` 插件，通过增量同步（rsync）将仓库中的静态文件部署到服务器的 `/var/www/html` 目录。
4. **安全排除**：自动过滤掉 `.git`、`.github` 等敏感或无关目录，确保服务器环境的安全与整洁。

## 配置 SSH 密钥认证
在你的**本地电脑**执行：
```bash
# 生成 SSH 密钥对
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy"

# 将公钥id_rsa.pub内容复制到服务器
ssh-copy-id your-username@your-server-ip
```

## 环境变量配置

为了使自动化部署正常工作，需要在 GitHub 仓库中配置以下 Secrets（路径：`Settings -> Secrets and variables -> Actions`）：

| Secret 名称 | 描述 | 示例/说明 |
| ------ |------ |------ |
| `REMOTE_HOST` | ECS 服务器公网 IP | `192.168.1.100` |
| `REMOTE_USER` | SSH 登录用户名 | `root` 或 `ubuntu` |
| `SERVER_SSH_KEY` | SSH 私钥内容 | 完整的id_rsa私钥字符串（包含 BEGIN/END 标识） |

## ️ ECS 服务器环境要求

在首次部署前，请确保您的 ECS 服务器满足以下条件：

- **Web 服务器**：已安装并配置 Nginx ，且网站根目录指向 `/var/www/html`(服务器/etc/nginx/nginx.conf文件中配置 root	/var/www/html;)。
- **必要依赖**：服务器已安装 `rsync` 工具（流水线会在部署前尝试自动安装，但建议提前手动安装）。
- **端口放行**：安全组已放行 `22`（SSH）、`80`（HTTP）及 `443`（HTTPS）端口。
- **目录权限**：确保 `REMOTE_USER` 对 `/var/www/html` 目录拥有读写权限。

## 快速开始

1. **克隆仓库**：
2. **修改页面**：在本地编辑 `index.html` 或其他静态资源。
3. **提交推送**：
4. **查看进度**：前往 GitHub 仓库的 **Actions** 面板，查看部署流水线是否成功。

## 常见问题排查

- **报错 **`**rsync: command not found**`：请手动登录服务器执行 `sudo apt install rsync` (Ubuntu) 或 `sudo yum install rsync` (CentOS)。
- **报错 **`**Permission denied**`：请检查 ECS 上 `/var/www/html` 的所有者是否为部署使用的 SSH 用户。
- **页面未更新**：检查浏览器缓存，或确认 Nginx 配置中的 `root` 路径是否正确。

## 许可证

本项目仅供学习和内部使用。

---

- 提示：如需修改部署目标目录或排除规则，请编辑 `.github/workflows/deploy.yml` 文件。*
