# Siemens TIA Portal Skill for OpenClaw

西门子博图 TIA Portal / STEP 7 自动化工程技能，适用于 OpenClaw AI 助手。

[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-OpenClaw-blue)](https://github.com/openclaw/openclaw)

## 适用版本

- TIA Portal V16 / V17（主要）
- V18+ 需注意界面、库、CPU 固件和编译诊断差异

## 支持

- ✅ S7-1200 / S7-1500 全系列
- ✅ S7-300 / S7-400 旧项目维护
- ✅ LAD / FBD / SCL / STL / GRAPH 多语言
- ✅ PLCSIM 仿真测试
- ✅ HMI / WinCC 排查
- ✅ PROFINET / PROFIBUS 通信诊断
- ✅ SCL 代码生成（UTF-8 BOM、# 前缀、FB 多实例）

## 安装

```bash
# 克隆仓库
git clone https://github.com/JasonStaniu/siemens-tia-skill.git

# 复制到 OpenClaw skills 目录
cp -r siemens-tia-skill ~/.openclaw/skills/siemens-tia

# 或直接克隆到 skills 目录
git clone https://github.com/JasonStaniu/siemens-tia-skill.git ~/.openclaw/skills/siemens-tia
```

安装后重启 Gateway 或开新会话即可生效。

## 使用

在 OpenClaw 对话中直接提到博图相关问题，Agent 会自动加载此技能。也可以手动调用：

```
/skill siemens-tia
```

## 文件结构

```
siemens-tia/
├── SKILL.md           # 技能主文件（包含完整工作流）
├── languages.md        # LAD/FBD/SCL/STL/GRAPH 语言参考
├── scl-rules.md        # SCL 代码铁律、模板和示例
└── troubleshooting.md  # 编译、下载、仿真、HMI、通信排查
```

## 安全提醒

⚠️ 此技能涉及真实 PLC 操作。使用前请务必确认：

- 操作对象是真实 PLC 还是 PLCSIM
- 已备份现场程序和 DB 当前值
- 设备处于手动/停机/安全状态
- 已获得现场负责人授权
- 不绕过任何安全功能（急停、安全门、光栅等）

## License

MIT
