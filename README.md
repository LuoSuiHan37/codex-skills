# Codex Skills

可复用的 Codex Skills 集合，面向软件开发、自动化、打包部署、文档处理和效率工具等工作流。

这些 Skills 用于为 Codex 提供特定领域的操作规范、经验总结、故障排查方法和可复用流程。

## Skills

### Windows Portable EXE Packaging

用于将 Python 桌面应用打包为 Windows EXE，重点解决应用在其他电脑上无法运行的问题。

主要内容：

- PyInstaller 打包流程
- PySide6、Qt 和 Shiboken6 DLL 处理
- Playwright 浏览器依赖处理
- 隐藏导入和运行时 Hook
- `sys._MEIPASS` 与冻结路径处理
- 配置文件、数据库、日志和用户数据路径管理
- 跨电脑运行失败排查
- onedir 与 onefile 打包模式说明
- 单文件 EXE 的验证方法

目录：

```text
windows-portable-exe-packaging/
├─ SKILL.md
├─ agents/
│  └─ openai.yaml
└─ references/
   └─ packaging-lessons.md

使用方式：
C:\Users\<用户名>\.codex\skills\windows-portable-exe-packaging\

提示词使用：
请使用 windows-portable-exe-packaging Skill，
帮助我完成当前项目的 Windows EXE 打包。

后续计划
后续将持续添加不同功能的 Codex Skills，例如：
- Python 项目开发
- Web 应用开发
- 自动化脚本
- 文档处理
- 数据分析
- Windows 软件部署
- UI/UX 设计
- 测试与故障排查
- 视频和图像处理
