---
name: "repair-stacked-pr-ci-without-rewrite"
description: "修复已发布 stacked PR 的中间层CI红而完整tip绿，保持PR编号和base链、不force-push；排查测试入口后置、fixture和文档基线落在后层。"
version: 1
created: "2026-11-14"
updated: "2026-11-14"
---
## When to Use
完整tip通过但中间层失败，需要保留既有PR链并修复每层门禁。不是生产部署或合并授权。

## Procedure
1. 冻结原始每层head、完整tip和当前远端main；核对真实CI job annotations，不把旧失败视为新workflow的结果。
2. 在隔离worktree复现最底层失败，定位测试断言、fixture、测试入口或文档基线是否错放后层。
3. 第一层普通merge冻结main，之后逐层普通merge已验证父层；不rebase已发布历史，不force-push。
4. 仅把本层已具备行为的测试与文档前移，保留所有门禁。冲突逐块核对，不整文件选边。
5. 每层由父会话启动并监督完整测试，子代理只交付commit及小检查；跨会话后台task ID不能视为可接管证明。
6. 独立审查修复，普通push后立即挂CI监控；核对成功检查属于最新head，再推进下一层。
7. 最终按blob和mode对比新树与原tip加冻结main的预期树，列明每个额外差异。复核每层head/base链与CI，记录未合并和未部署边界。

## Pitfalls
- 完整tip绿不能证明中间层自洽；中间层CI绿也不能证明它可独立部署。
- 旧label job因计费未启动时，迁移workflow不自动改写旧结果。
- 测试替身必须符合测试契约，并有对应真实逻辑的独立测试，不能仅用exit0掩盖缺失依赖。
- merge文档时保留必要数字与条件；真实生成产物，不只更新摘要。
- 子会话结束后后台进程可能不再存活；父会话须启动自己的长验证并接收通知。

## Verification
1. 原始每层head及已验证父层仍是新head祖先，PR编号/base关系未改变。
2. 各层本地完整验证通过，最新远端head对应检查成功。
3. 最终树与预期树一致，任何差异逐项说明；共享checkout的他人改动未变。