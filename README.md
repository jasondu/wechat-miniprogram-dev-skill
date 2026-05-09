# wechat-miniprogram-dev

A Claude Code skill for WeChat Mini Program development with Tencent CloudBase (微信小程序云开发).

一个用于微信小程序云开发的 Claude Code Skill。

## Features / 功能

- 🚀 CloudBase initialization configuration / CloudBase 初始化配置
- ☁️ Cloud functions development & deployment / 云函数开发与部署
- 🗄️ Database operations / 数据库操作
- 📦 Cloud storage handling / 云存储处理
- 🔄 CI/CD with GitHub Actions / GitHub Actions CI/CD 配置
- 🖥️ Admin dashboard development / 管理后台开发

## Installation / 安装

### Method 1: Direct Copy / 方法一：直接复制

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/wechat-miniprogram-dev-skill.git

# Copy to Claude Code skills directory
cp -r wechat-miniprogram-dev-skill ~/.claude/skills/wechat-miniprogram-dev
```

### Method 2: Manual / 方法二：手动安装

1. Create the skills directory:
```bash
mkdir -p ~/.claude/skills/wechat-miniprogram-dev
```

2. Copy `SKILL.md` to that directory:
```bash
cp SKILL.md ~/.claude/skills/wechat-miniprogram-dev/
```

## Usage / 使用方法

After installation, Claude Code will automatically use this skill when you:

安装后，当你使用以下关键词时，Claude Code 会自动调用此 Skill：

- 小程序 / miniprogram
- 云开发 / CloudBase
- wx.cloud
- 云函数 / cloud function
- 微信小程序 / WeChat Mini Program

Example prompts / 示例提示：

```
帮我创建一个微信小程序的云函数
How to handle share scene authentication in mini program?
搭建小程序的 CI/CD 部署流程
```

## Project Structure / 项目结构

```
wechat-miniprogram-dev-skill/
├── SKILL.md              # Core skill documentation
├── README.md             # This file
├── LICENSE               # MIT License
├── CONTRIBUTING.md       # Contribution guidelines
├── evals/
│   └── evals.json        # Test cases for skill evaluation
└── examples/             # Example projects (coming soon)
```

## Evals / 测试用例

The `evals/evals.json` file contains test cases to evaluate the skill's performance:

```json
{
  "skill_name": "wechat-miniprogram-dev",
  "evals": [
    {
      "id": 1,
      "prompt": "分享页面调用云函数报错 -501023...",
      "expected_output": "分享场景认证解决方案..."
    }
  ]
}
```

## Contributing / 贡献

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

欢迎贡献！请查看 [CONTRIBUTING.md](CONTRIBUTING.md) 了解详情。

## Related Skills / 相关 Skills

- `ai-model-wechat` - AI capabilities in WeChat Mini Program
- `auth-wechat-miniprogram` - Authentication patterns
- `cloud-functions` - CloudBase cloud functions details
- `cloudbase-document-database-in-wechat-miniprogram` - Database SDK usage

## Resources / 资源

- [WeChat Mini Program Documentation](https://developers.weixin.qq.com/miniprogram/dev/framework/)
- [CloudBase Documentation](https://cloud.tencent.com/document/product/876)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)

## License / 许可证

[MIT License](LICENSE)

## Author / 作者

Created by the CloudBase community.

---

If this skill helps you, please give it a ⭐️!

如果这个 Skill 对你有帮助，请给个 ⭐️！