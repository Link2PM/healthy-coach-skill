# healthy-coach — AI 访谈生成你的体态矫正训练手册

一个 Claude Code skill：**访谈你的体态问题和训练条件 → 生成个人定制训练计划 → 自动部署成可安装的 PWA 训练手册**，全程几分钟，最后你会拿到一个 `https://<你的用户名>.github.io/<仓库名>/` 链接，iPhone 上「添加到主屏幕」即可当 App 用。

应用本体是 [healthy-app-template](https://github.com/Link2PM/healthy-app-template)：零后端、离线优先、数据全在本地浏览器，内置 AI 训练分析（自备 API Key，支持 Claude / Gemini / Qwen / DeepSeek）和本地 MCP server（让你自己的 Claude 直接读训练数据做复盘）。

## 前置要求

- [Claude Code](https://claude.com/claude-code)
- [GitHub CLI](https://cli.github.com)（`gh auth login` 登录你的 GitHub 账号）
- Node.js（跑计划校验器）

## 安装

```bash
git clone https://github.com/Link2PM/healthy-coach-skill.git ~/.claude/skills/healthy-coach
```

然后在 Claude Code 里输入（若当前会话未识别该 skill，新开一个会话即可）：

```
/healthy-coach
```

## 它会做什么

1. **访谈**：分批询问体态问题（含自查方法）、疼痛与病史（红旗信号会建议先就医）、目标、每周可训练天数、器械条件、经验水平
2. **生成**：按 [plan.js 数据契约](references/plan-format.md) 生成阶段化、渐进超负荷的个人计划，强制通过结构校验器
3. **部署**：从模板仓库在你的 GitHub 账号下建仓、写入你的计划、开启 GitHub Pages，交付可访问链接
4. **交待清楚**：iOS 安装方法、数据备份、AI 分析配置、MCP 接入，一次讲清

## 数据与隐私

- 训练数据只存在你手机/浏览器本地，导出导入都是一个 JSON 文件
- GitHub Pages 免费版要求公开仓库——你的**训练计划**会公开可见（训练**记录**不会，它们只在本地）
- AI 分析用你自己的 API Key，直连厂商，不经过任何第三方

## 免责声明

生成的训练计划仅供参考，不构成医疗建议。有伤病史或训练中出现疼痛请咨询医生或康复专业人士。

## License

[MIT](LICENSE)
