# Contributing to wechat-miniprogram-dev-skill

Thank you for your interest in contributing! / 感谢你有兴趣贡献！

## How to Contribute / 如何贡献

### Reporting Issues / 报告问题

If you find a bug or have a suggestion:

如果你发现 bug 或有建议：

1. Check if the issue already exists / 检查问题是否已存在
2. Open a new issue with a clear title and description / 创建新 issue，标题清晰，描述详细
3. Include code examples if applicable / 如适用，包含代码示例

### Submitting Changes / 提交更改

1. **Fork the repository** / Fork 仓库

2. **Create a branch** / 创建分支
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes** / 进行修改
   - Update `SKILL.md` for content changes / 修改内容时更新 SKILL.md
   - Add test cases to `evals/evals.json` / 添加测试用例到 evals/evals.json
   - Update `README.md` if needed / 必要时更新 README.md

4. **Test your changes** / 测试你的修改
   - Verify the skill works correctly in Claude Code / 验证 Skill 在 Claude Code 中正常工作
   - Run eval test cases / 运行 eval 测试用例

5. **Commit your changes** / 提交修改
   ```bash
   git commit -m "Add: description of your changes"
   ```

6. **Push and create PR** / 推送并创建 PR
   ```bash
   git push origin feature/your-feature-name
   ```

## Guidelines / 指南

### Skill Content / Skill 内容

- **Accuracy**: Ensure all code examples work correctly / 确保所有代码示例正常工作
- **Clarity**: Write clear, concise explanations / 编写清晰、简洁的说明
- **Bilingual**: Support both Chinese and English where possible / 尽可能支持中英双语
- **Best Practices**: Follow WeChat Mini Program and CloudBase best practices / 遵循微信小程序和 CloudBase 最佳实践

### Code Examples / 代码示例

- Include error handling / 包含错误处理
- Add comments in Chinese for complex logic / 复杂逻辑添加中文注释
- Use modern JavaScript syntax / 使用现代 JavaScript 语法

### Evals / 测试用例

When adding new scenarios, add corresponding eval test cases:

添加新场景时，添加对应的 eval 测试用例：

```json
{
  "id": 6,
  "prompt": "用户的问题或场景描述",
  "expected_output": "期望的回答要点"
}
```

## Development Setup / 开发环境配置

1. Clone the repository / 克隆仓库
   ```bash
   git clone https://github.com/YOUR_USERNAME/wechat-miniprogram-dev-skill.git
   ```

2. Link to Claude Code for testing / 链接到 Claude Code 进行测试
   ```bash
   ln -s $(pwd) ~/.claude/skills/wechat-miniprogram-dev
   ```

3. Test in Claude Code / 在 Claude Code 中测试
   - Start Claude Code / 启动 Claude Code
   - Ask questions related to WeChat Mini Program / 提问微信小程序相关问题
   - Verify the skill is triggered and provides correct answers / 验证 Skill 被正确触发并提供正确答案

## Questions? / 有问题？

Feel free to open an issue for any questions!

有问题随时开 issue！