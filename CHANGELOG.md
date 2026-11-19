# Changelog

本文件记录 mini-netty 项目的所有改动，按分支组织。

## 格式说明

每个步骤记录包含：
- **分支名**: 对应的 Git 分支
- **改动内容**: 本步骤的具体改动
- **学习要点**: 本步骤的核心知识点

---

## [STEP-00] 项目初始化 (2026-11-19)

**分支**: `main`

**改动内容**:
- 初始化 Maven 工程脚手架（`mini-netty`，Java 17）。
- 引入 JUnit 5（`junit-jupiter`）依赖。
- 创建项目说明文档 `README.md` 与 `CHANGELOG.md`。

**学习要点**:
- Maven 工程结构与依赖管理
- JUnit 5 测试依赖的引入（`test` 作用域）