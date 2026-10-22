# 实验 1 记录：第 2 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 2 组 |
| 仓库地址 | [RyanShen30/ops-lab1-2](https://github.com/RyanShen30/ops-lab1-2) |
| 代码载体 | 统一示例 `enroll-conflict`（Go） |
| runner 与语言版本 | `ubuntu-24.04`，Go `1.26.9` |
| 合并方式 | squash |

## 2. 任务 0：本地基线

在公网 IP 为 `124.71.190.176` 的服务器上，直接使用当前 `sra` 账户，在 `/home/sra/lab1-local` 操作。沿用现有 Go `1.26.9`（`linux/amd64`）、GNU make `4.3` 和 GNU tar `1.35`；本目录由课程模板仓库克隆而来，HEAD 为 `7b63b41c13f061439997b0e6e4eaa2af922a7aa1`。

两次均执行 `make package VERSION=1.0.0 BUILD_NO=1`，未指定的 `RUN_ID` 沿用默认值 `local`，`COMMIT` 由 Makefile 从 Git HEAD 取得。第一次构建前工作区干净；第二次仅修改 `conflict.go` 中的一行注释，不提交。两次取证完成并恢复源文件之后，才填写本节，避免本文件的修改影响第一次构建的 VCS 状态。

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | 见下方“第一次打包”的完整六行实际输出。 |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | `BUILD_INFO` 完全相同，`diff` 退出码为 0；二进制 SHA256 不同，两个完整值分别列于下方。 |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | `vcs.modified=false` 变为 `vcs.modified=true`；`vcs=git`、`vcs.revision` 和 `vcs.time` 均不变，实际输出见下方。完整输出中的模块版本还增加了 `+dirty` 后缀。 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | `name`、`version`、`build_no` 能说明包名和声明的版本、构建号，`go` 能说明 Go 版本；没有构建机器和构建时间字段，无法回答在哪台机器、何时构建。`commit` 只能指出作为基线的提交，未记录未提交修改或工作区是否干净；若据此认定第二个包严格来自该提交的源码快照，就会判断错误。相同的版本号、构建号和 `commit` 在本次实验中对应了不同的二进制。 |

### 第一次打包：修改前，工作区干净

`cat dist/enroll-conflict_1.0.0_linux_amd64/BUILD_INFO`：

```text
name=enroll-conflict
version=1.0.0
build_no=1
run_id=local
commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
go=go1.26.9
```

`sha256sum dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict`：

```text
f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e  dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict
```

`go version -m` 输出中的全部 VCS 行：

```text
	build	vcs=git
	build	vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
	build	vcs.time=2026-10-12T16:26:07Z
	build	vcs.modified=false
```

### 第二次打包：修改注释，不提交

临时修改的一行如下；没有修改程序逻辑：

```diff
-// Weeks 表示一个教学班在哪些周上课。
+// Weeks 表示一个教学班在哪些周上课。（任务 0：临时注释修改，不提交。）
```

第二次打包前，`git status --porcelain=v1` 的实际输出：

```text
 M conflict.go
```

`cat dist/enroll-conflict_1.0.0_linux_amd64/BUILD_INFO`：

```text
name=enroll-conflict
version=1.0.0
build_no=1
run_id=local
commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
go=go1.26.9
```

`sha256sum dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict`：

```text
6a578f147d5581d0a95d3b4511436be491a8401f93f3f0d5d49ea5da1966bda5  dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict
```

`go version -m` 输出中的全部 VCS 行：

```text
	build	vcs=git
	build	vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
	build	vcs.time=2026-10-12T16:26:07Z
	build	vcs.modified=true
```

对比 `dist/task0-evidence/first/BUILD_INFO` 与第二次生成的 `BUILD_INFO`，`diff` 无差异且退出码为 0，操作脚本输出：

```text
BUILD_INFO 没有变化
```

本次只改了注释，但 Go 写入二进制的 VCS 状态和模块版本信息发生了变化，因此即使程序逻辑不变，二进制 SHA256 仍然改变。`vcs.time=2026-10-12T16:26:07Z` 是提交时间，不是构建时间；`vcs.modified=true` 能提示存在未提交内容，但不能还原这些内容。

### 恢复与证据保留

已从构建前保存的原文件恢复 `conflict.go`，`git diff --exit-code -- conflict.go` 退出码为 0。恢复后的文件 SHA256 与构建前一致：

```text
69a4ed5bf918cc8cfb9366ffc61f1872a3b9996aeb74d074c12028bc3fc0c1e3  conflict.go
```

恢复后、填写本节前，`git status --porcelain=v1` 无输出。两次打包的二进制、`BUILD_INFO`、压缩包、校验文件、构建日志及完整 `go version -m` 输出分别保存在本机 `dist/task0-evidence/first/` 和 `dist/task0-evidence/second/`；两份压缩包均通过 `sha256sum -c` 校验。该证据目录位于已忽略的 `dist/` 内，不影响第一次构建的干净状态；执行 `make clean` 会删除它。

以上是任务 0 在模板仓库克隆中的实际记录；任务 2 起使用小组仓库 `/home/sra/lab1`，其 origin 指向 `RyanShen30/ops-lab1-2`。

## 3. 证据

任务 2 的 [CI 工作流](https://github.com/RyanShen30/ops-lab1-2/blob/main/.github/workflows/ci.yml)与[运行列表](https://github.com/RyanShen30/ops-lab1-2/actions/workflows/ci.yml)：PR 和 push 到 `main` 均运行 `ci`，仅 push 到 `main` 时打包并上传制品，保留 14 天。

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | [main-gate](https://github.com/RyanShen30/ops-lab1-2/rules/25238784)，状态为 Active，作用于默认分支 `main`，Bypass list 为空。启用 `Restrict deletions`、`Block force pushes`、`Require a pull request before merging`（Required approvals = **1**）、`Require status checks to pass`（必需检查为 **`ci`**，来源为 GitHub Actions）。允许的合并方式仅为 **Squash**。 |
| 组长直接 `git push origin main` 被拒的终端输出 | 见下方“任务 3：直接推送被拒记录”的完整实际输出；GitHub 返回 `GH013`，指出必须通过 PR 且缺少必需的 `ci` 检查。 |
| 第一轮：失败的 PR 运行 | [PR #2](https://github.com/RyanShen30/ops-lab1-2/pull/2) 的 [ci #3](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100037612)，`Test` 步骤失败。 |
| 第一轮：修复后通过的 PR 运行 | [ci #4](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100223291) 通过；随后清理临时注释的 [ci #5](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100712651)、补充实验记录的 [ci #6](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39101086698) 也通过。 |
| 第一轮：合入 main 后的运行（含制品） | [ci #7](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39105597138)，事件为 push，分支 main，结果 success；Package 和 Upload 均成功，产出 build-7-attempt-1 制品。 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | 将 Valid 中拒绝 End < Start 改为拒绝 End <= Start，导致起止节次相同的合法单节课程被拒绝。Section 注释规定起止节次均包含；原有合法用例仅覆盖 Start < End，缺少 Start == End 的边界测试。 |
| 第二轮：仅含缺陷时仍然通过的运行 | [PR #3](https://github.com/RyanShen30/ops-lab1-2/pull/3) 的 [ci #8](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106019814)，success；仅含缺陷提交 d1e21a4，未审批、未合并。 |
| 第二轮：补测试后失败的运行 | [ci #9](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106084105)，Test 步骤失败，新增 TestValidSinglePeriod 在 all、odd、even 三种周次下均发现缺陷。 |
| 第二轮：撤销缺陷后通过的运行 | [ci #10](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106121523)，success；通过 git revert d1e21a4 撤销缺陷，保留新增测试。随后补充实验记录的 [ci #11](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106167982) 也通过。 |
| 第二轮：审批与合入 main 后的运行 | [PR #3 审批](https://github.com/RyanShen30/ops-lab1-2/pull/3#pullrequestreview-5561903762)；2026-10-18 15:14:54 +08:00 squash 合入；[ci #12](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106946423) 全部成功并产出制品。 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | [release #1](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39197867811)，测试 tag `v0.0.1` 指向未合入 main 的提交，`Release gate` 步骤失败，后续步骤跳过；测试 tag 和分支已清理，未生成 Release。 |
| Release v1.0.0 | [v1.0.0](https://github.com/RyanShen30/ops-lab1-2/releases/tag/v1.0.0)，由 [release #2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440) 成功发布；两个附件已下载，校验和及追溯检查均通过，实际值见第 4 节。 |
| Release v1.0.1 | [v1.0.1](https://github.com/RyanShen30/ops-lab1-2/releases/tag/v1.0.1)，由 [release #3](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369) 成功发布；两个附件已下载，校验和与追溯均通过，实际值见第 4 节。 |

### 任务 3：直接推送被拒记录

以下终端输出由组长完成验证后提供。规则集配置已通过 GitHub API 复核，与上表一致。

```text
git push origin main
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 192 bytes | 192.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: Review all repository rules at https://github.com/RyanShen30/ops-lab1-2/rules?ref=refs%2Fheads%2Fmain
remote:
remote: - Changes must be made through a pull request.
remote:
remote: - Required status check "ci" is expected.
remote:
To https://github.com/RyanShen30/ops-lab1-2.git
 ! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs to 'https://github.com/RyanShen30/ops-lab1-2.git'
```

验证结论：直接推送 `main` 因仓库规则被拒绝，该次推送没有更新远端 `main`。测试用的本地空提交已清理。本节记录与任务 4 第一轮证据一并通过 [PR #2](https://github.com/RyanShen30/ops-lab1-2/pull/2) 提交；后续修改通过 PR、另一位组员审批及 `ci` 检查后合入。

### 任务 4：第一轮 gate 演练

在从 `main` 创建的 `test/gate-round1` 分支上，首个提交同时新增“单双周部分重叠不冲突”测试，并将 `weeksOverlap` 的最后返回值临时改为 `true`。新增用例的两个课程节次部分重叠，但分别在单周、双周上课，预期不冲突。提交后发起 PR #2，ci #3 的格式检查和 `Go vet` 通过，`Test` 失败，后续打包和上传步骤未执行。

失败阶段的截图中，必需检查 `ci / ci (pull_request)` 显示失败，`Squash and merge` 按钮不可用；合并区显示 `Merging is blocked`，同时提示 `At least 1 approving review is required by reviewers with write access.`。该截图也说明当时尚缺组员审批；测试失败的具体证据见上表 ci #3。

随后在同一分支追加[修复提交](https://github.com/RyanShen30/ops-lab1-2/commit/da2d9170a2ed5fa93fdc02470c944335bcbcc8e1)，恢复 `return a == b`，保留新增测试，ci #4 通过。再追加[清理临时注释的提交](https://github.com/RyanShen30/ops-lab1-2/commit/42ed4a94f599a5653a3320a1abbae1b6cc9b790a)，ci #5 通过；最终生产代码恢复为原判断，仅保留新增测试。PR 的全部后续检查可在 [PR #2 检查列表](https://github.com/RyanShen30/ops-lab1-2/pull/2/checks)中查看。

组员 [Liuxiang400 于 2026-10-18 14:26:50 +08:00 审批通过](https://github.com/RyanShen30/ops-lab1-2/pull/2#pullrequestreview-5561740627)。最终更新记录的 ci #6 也通过。PR #2 于 2026-10-18 14:52:40 +08:00 squash 合入 main，生成提交 `fdbd6495969e9f18bc2823b5b03566a435a6c541`。最终标题为“test: 补充单双周课程冲突测试 (#2)”，正文仅保留新增测试和记录证据的说明。合并后 [ci #7](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39105597138) 于 14:52:42 触发，ci job 于 14:52:58 结束并成功，工作流于 14:52:59 更新为完成。打包和上传均成功，生成 artifact `enroll-conflict-fdbd6495969e9f18bc2823b5b03566a435a6c541-build-7-attempt-1`（ID 12128755180）。

### 任务 4：第二轮 gate 演练

本轮在服务器 /home/lx/ops-lab1-2 的 test/gate-round2 分支操作，PR 作者为 Liuxiang400。所有时间均为 Asia/Shanghai（UTC+08:00）。

1. 从最新 main（fdbd649）创建分支，仅提交 Valid 的缺陷。现有 make fmt-check vet test 全通过；PR #3 于 2026-10-18 14:59:51 创建，ci #8 于 15:00:13 结束并通过。当时 review 列表为空、PR 的 mergeable_state 为 blocked：自动检查未发现错误，仍由人工审批门禁阻止合入；此阶段未审批也未合并。
2. 在同一分支追加 TestValidSinglePeriod，使用 Start=End=2，分别覆盖 AllWeeks、OddWeeks、EvenWeeks。该测试依据“起始节次、结束节次均含”的规则，预期三种单节课程均合法。ci #9 的 Test 于 15:00:57 失败，错误显示三种输入均被错误拒绝。
3. 执行 git revert d1e21a4e9ad8c6019254e3b4f042394025d3a6d2，生成 2c174f9，保留新增测试。make fmt-check vet test 重新全部通过，git diff --exit-code origin/main -- conflict.go 无差异。对应 ci #10 通过。
4. PR #3 同时补齐第一轮审批、合并、main push 运行和制品记录。组员 [RyanShen30 于 2026-10-18 15:13:40 +08:00 审批通过](https://github.com/RyanShen30/ops-lab1-2/pull/3#pullrequestreview-5561903762)，并于 15:14:54 squash 合入 main，生成提交 [7abf6c457a5b6c093356f322c215952ca06e41f2](https://github.com/RyanShen30/ops-lab1-2/commit/7abf6c457a5b6c093356f322c215952ca06e41f2)。最终标题为“test: 补充单节课程合法性边界测试 (#3)”。该提交仅有一个父提交 fdbd649，代码差异仅新增 TestValidSinglePeriod，生产逻辑 conflict.go 与合并前一致。
5. 合入后的 [ci #12](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106946423) 为 main 的 push 运行，提交号与上述 squash 提交一致；[ci job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106946423/job/117614764107) 于 15:14:58 开始、15:15:14 结束，所有步骤成功，包含打包和上传。生成 artifact `enroll-conflict-7abf6c457a5b6c093356f322c215952ca06e41f2-build-12-attempt-1`（ID 12128717018）。

合并说明检查的实际偏差：PR #3 合并时未清理自动生成的正文，“introduce single-period validation defect”和 revert 等演练提交说明仍保留在 main 的提交正文中。这一点未满足指导书对合并说明清理的要求；最终代码没有保留缺陷。本记录如实保留该偏差，不通过强制推送改写受保护的 main 历史。

ci #9 新增测试的失败输出（服务器本地与 Actions 均发现相同问题）：

```text
--- FAIL: TestValidSinglePeriod (0.00s)
    conflict_test.go:60: single-period section should be valid: {ID: Weekday:1 Start:2 End:2 Weeks:all}
    conflict_test.go:60: single-period section should be valid: {ID: Weekday:1 Start:2 End:2 Weeks:odd}
    conflict_test.go:60: single-period section should be valid: {ID: Weekday:1 Start:2 End:2 Weeks:even}
FAIL
```

### 任务 5：发布工作流

负责人：`ashkirakira`，提交 [PR #5](https://github.com/RyanShen30/ops-lab1-2/pull/5)。新增 `.github/workflows/release.yml`，由符合 `v[0-9]+.[0-9]+.[0-9]+` 的 tag push 触发，执行顺序如下：

1. 使用 `ubuntu-24.04`、`actions/checkout@v7` 和 `fetch-depth: 0` 获取完整历史。首先运行 `git merge-base --is-ancestor "$GITHUB_SHA" origin/main`；不在主干历史上的提交立即失败，不进入编译或创建 Release 的步骤。
2. 使用 `actions/setup-go@v7` 安装固定版本 Go `1.26.9`，分别执行 `make fmt-check`、`make vet`、`make test`，任一步失败都会停止发布。
3. 从 tag 去掉开头的 `v` 得到 `VERSION`，将 `github.run_number`、`github.run_id`、`github.sha` 分别作为 `BUILD_NO`、`RUN_ID`、`COMMIT` 传给 `make package`。这些标识由事件上下文提供，通过环境变量进入脚本；`BUILD_INFO` 由已有 Makefile 写入制品。
4. 打印二进制和 `.tar.gz` 的 SHA-256，并执行 `sha256sum -c` 校验生成的压缩包。最后用 `gh release create --verify-tag` 创建 Release，上传 `.tar.gz` 和同名 `.sha256` 两个附件。仅本发布工作流申请 `contents: write`，原有 `ci` 仍为只读权限。

验证记录（2026-10-19）：

| 检查 | 实际结果 |
|---|---|
| 本地代码检查 | Go `1.26.6`（满足指导书本地最低版本 `1.26`）下 `make fmt-check vet test` 全部通过。 |
| 工作流静态检查 | `actionlint 1.7.12` 检查 `ci.yml` 与 `release.yml` 通过；`git diff --check` 通过。 |
| 本地打包预检 | 固定 `VERSION=1.0.0`、`BUILD_NO=local-validation`、`RUN_ID=local`、`COMMIT=9fc725b54d6d69e5ce08d639e1608df053dff778` 调用 `make package`，`sha256sum -c` 返回 `OK`；二进制 `-version` 输出的版本、提交号及构建号与参数一致。这里只是本地预检，不是正式 Release 附件。 |
| PR 的 Actions 检查 | 首次 [ci #15](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39197858128) 和补充文档后的 [ci #16](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39198002540) 均通过；使用 Go `1.26.9`，PR 事件未打包主干制品。 |
| 发布门禁反例 | [release #1](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39197867811) 在 `Release gate` 步骤失败，后续 Go 安装、检查、测试、打包及创建 Release 均为 skipped。 |
| 合入主干后的 CI | [ci #17](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39207874767) 为 `main` 的 push 运行，提交为 `5c68dd213a4389cf51500b43be2729ecc880a22e`，全部通过。 |
| 正式发布 | [release #2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440) 的门禁、格式检查、vet、测试、打包、校验和检查及创建 Release 全部成功，发布 `v1.0.0`。 |

为在审批前提供真实门禁证据，从任务 5 的实现提交 `9fc725b` 创建临时分支 `scratch/gate-test`，追加一个空提交 `d5f2bd15cb59aa15c55faf988f67f15867419066`，再打并推送带注释的测试 tag `v0.0.1`。该提交包含发布工作流，但不在 `origin/main` 历史上。Actions 于 `2026-10-19 12:07:07 +08:00` 拒绝发布，日志中的实际错误为：

```text
Tag v0.0.1 points to d5f2bd15cb59aa15c55faf988f67f15867419066, which is not in origin/main history.
Process completed with exit code 1.
```

随后删除远端 `v0.0.1` 和 `scratch/gate-test`，并删除本地对应 tag、分支。`git ls-remote origin refs/tags/v0.0.1 refs/heads/scratch/gate-test` 无输出；清理当时 GitHub Releases API 返回空列表，确认未留下测试 Release。失败运行记录予以保留，供现场检查核对。

PR #5 先后经 [Liuxiang400 于 2026-10-19 14:02:40 +08:00 审批](https://github.com/RyanShen30/ops-lab1-2/pull/5#pullrequestreview-5567500720)、[Shundi-Wu 于 14:04:37 +08:00 审批](https://github.com/RyanShen30/ops-lab1-2/pull/5#pullrequestreview-5567512409)，于 `14:04:57 +08:00` squash 合入 `main`，生成提交 `5c68dd213a4389cf51500b43be2729ecc880a22e`。合入后的 ci #17 通过，才进行正式发布。

在该主干提交上执行 `git tag -a v1.0.0 -m "v1.0.0"`。本次 Git 推送连接超时，确认远端尚无此 tag 后，通过 GitHub API 创建相同的带注释 tag 对象与 `refs/tags/v1.0.0` 引用；远端和本地 tag 对象 SHA 均为 `784eb630150219f2bed4924872044d33eb730328`，指向的提交均为上述主干提交。该操作触发 release #2，`v1.0.0` 于 `2026-10-19 20:55:07 +08:00` 发布；没有绕过发布门禁，也没有复用已发布版本号。

发布后下载两个正式附件并完成以下验证：

```bash
gh release download v1.0.0 --repo RyanShen30/ops-lab1-2 \
  --pattern enroll-conflict_1.0.0_linux_amd64.tar.gz \
  --pattern enroll-conflict_1.0.0_linux_amd64.tar.gz.sha256 --dir dist/release
(cd dist/release && sha256sum -c enroll-conflict_1.0.0_linux_amd64.tar.gz.sha256)
scripts/trace.sh dist/release/enroll-conflict_1.0.0_linux_amd64.tar.gz . origin/main
```

`sha256sum -c` 输出 `enroll-conflict_1.0.0_linux_amd64.tar.gz: OK`，`trace.sh` 退出码为 **0**。下载文件的 SHA-256 与 Release 的 `.sha256`、GitHub 附件 digest 和 Actions 打包日志四者一致，完整值已记入第 4 节。解压后实际二进制的 SHA-256 为 `f684cdaf5547ddc34178cb7782504e1b29f326eed850ef61e8dc2cdc3f3bef49`，与 Actions 日志一致；执行 `-version` 得到：

```text
enroll-conflict version=1.0.0 commit=5c68dd213a4389cf51500b43be2729ecc880a22e build_no=2 go=go1.26.9
```

以上完成任务 5 的发布与校验取证，作为任务 6 的旧版本对照基线。

### 任务 6：v1.0.1 的发布

负责人：`Shundi-Wu`。本次发布包含两个 PR，均由 Shundi-Wu 发起，由 RyanShen30 审批并 squash 合入。所有时间均为 Asia/Shanghai（UTC+08:00）。

| PR | 内容 | PR 上的 ci | 审批 | 合入与提交 | 合入后 main 的 ci |
|---|---|---|---|---|---|
| [#7](https://github.com/RyanShen30/ops-lab1-2/pull/7) | 任务 6 的小改动：`/version` 与 `-version` 增加 `run_id`，Makefile 通过 `-ldflags` 注入 `RUN_ID` | [ci #20](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627154197) | [2026-10-22 10:49:43](https://github.com/RyanShen30/ops-lab1-2/pull/7#pullrequestreview-5608701517) | 10:50:02，`7e413b9` | [ci #25](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713533558) |
| [#8](https://github.com/RyanShen30/ops-lab1-2/pull/8) | 选做 O1：`release.yml` 增加 `image` job，见第 7 节 | [ci #21](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627305858) | [2026-10-22 10:50:27](https://github.com/RyanShen30/ops-lab1-2/pull/8#pullrequestreview-5608705542) | 10:50:49，`20da6ee` | [ci #26](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713608095) |

改动 #7 的原因见第 4 节：`build_no` 在 `ci` 和 `release` 中各自计数，运行中的服务原先只能报出构建号，无法确定是哪次运行。

合入顺序为先 #7、后 #8，main 的最新提交是 #8 的 `20da6ee`。tag 只能打在已经包含 `image` job 的提交上，所以 `v1.0.1` 打在 `20da6ee`，`trace.sh` 追溯到的 PR 是 #8。`git log --oneline v1.0.0..v1.0.1` 显示本版本相对 `v1.0.0` 新增 #6、#7、#8 三个提交。

两个 PR 一起合入后的状态，此前没有在 CI 上一起运行过，因此先等 ci #26 通过，再执行 `git switch main && git pull`、`git tag -a v1.0.1 -m "v1.0.1"` 和 `git push origin v1.0.1`。tag 于 10:52 推送，触发 [release #3](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369)：

- [release job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369/job/119610787927)（10:52:08～10:52:36）：门禁、检查、测试、打包和校验全部通过，于 10:52:33 发布 `v1.0.1`。
- [image job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369/job/119610931604)（10:52:38～10:52:49）：成功推送镜像。

release job 打包日志中，二进制文件的 SHA-256 为 `99650f311d8fd045d33410c8cfd83ccd24a98e19018abffff474fde268637066`，`tar.gz` 的为 `d3a5338aac14b58e4e9e45a97531437694e78230e5d925da86246deae8b3a20c`。

## 4. 追溯与运行

`v1.0.0` 的任务 5 发布基线与实际追溯输出如下。任务 6 在与任务 0 相同的服务器上操作，使用 `wsd` 账户下的小组仓库克隆 `/home/wsd/projects/ops-lab1-2`。按指导书命令用 `curl` 匿名下载两个版本的附件到 `dist/release/`，2026-10-22 10:53:27 +08:00 下载完成。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | `1.0.0` | `1.0.1` |
| 构建号 | `2`（`release` 工作流） | `3`（`release` 工作流） |
| 提交号（SHA） | `5c68dd213a4389cf51500b43be2729ecc880a22e` | `20da6eed62da2492360d72559b1fae6040cde7c8` |
| 引入该提交的 PR | [#5](https://github.com/RyanShen30/ops-lab1-2/pull/5) | [#8](https://github.com/RyanShen30/ops-lab1-2/pull/8)；本版本还包含任务 6 的改动 [#7](https://github.com/RyanShen30/ops-lab1-2/pull/7) |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | [release #2，run_id=39255772440](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440) | [release #3，run_id=39713717369](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369) |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `d2936cc91dc6afbaaf90279f6501aedf7edb64770c7c0e64865c7589d8d3c415` | `d3a5338aac14b58e4e9e45a97531437694e78230e5d925da86246deae8b3a20c` |

仅凭构建号不能唯一确定运行。`v1.0.0` 的构建号为 `2`：在 `ci` 中对应 [ci #2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/38902466874)，它是 PR 事件，提交号为 `8ed1465a9357c4817964229622a9ecc849fc42dd`；在 `release` 中对应上述 [release #2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440)，它才构建了 `v1.0.0`。两条工作流各自编号，`BUILD_INFO` 的 `run_id=39255772440` 才能直接确定本次发布运行。`v1.0.1` 的构建号 `3` 同样重号：[ci #3](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100037612) 是任务 4 第一轮中失败的 PR 运行，release #3 才是本次发布。

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：`d2936cc91dc6afbaaf90279f6501aedf7edb64770c7c0e64865c7589d8d3c415`。与上表中发布当时的值是否一致？一致。解压后，`v1.0.0` 二进制文件的 SHA-256 仍为任务 5 记录的 `f684cdaf5547ddc34178cb7782504e1b29f326eed850ef61e8dc2cdc3f3bef49`。

```text
$ sha256sum dist/release/*.tar.gz
d2936cc91dc6afbaaf90279f6501aedf7edb64770c7c0e64865c7589d8d3c415  dist/release/enroll-conflict_1.0.0_linux_amd64.tar.gz
d3a5338aac14b58e4e9e45a97531437694e78230e5d925da86246deae8b3a20c  dist/release/enroll-conflict_1.0.1_linux_amd64.tar.gz
```

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=2
run_id=39255772440
commit=5c68dd213a4389cf51500b43be2729ecc880a22e
go=go1.26.9
== 3. 提交 5c68dd213a4389cf51500b43be2729ecc880a22e
作者: gaofei <motoori9@gmail.com>
时间: 2026-10-19T14:04:57+08:00
标题: ci: 添加任务 5 的 tag 发布与主干门禁 (#5)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 2
提交页面（含关联的 PR）: https://github.com/RyanShen30/ops-lab1-2/commit/5c68dd213a4389cf51500b43be2729ecc880a22e
构建它的运行: https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440
```

`trace.sh` 输出（v1.0.1），退出码为 **0**：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=39713717369
commit=20da6eed62da2492360d72559b1fae6040cde7c8
go=go1.26.9
== 3. 提交 20da6eed62da2492360d72559b1fae6040cde7c8
作者: Shundi Wu <wushundi0515@gmail.com>
时间: 2026-10-22T10:50:49+08:00
标题: ci: 发布后用 Release 附件构建并推送容器镜像 (#8)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/RyanShen30/ops-lab1-2/commit/20da6eed62da2492360d72559b1fae6040cde7c8
构建它的运行: https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369
```

同一次操作中，对 `v1.0.0` 重新运行 `trace.sh`，输出与上方任务 5 时的记录相同，退出码也为 0。

任务 6 第 4 步，两次 `/version` 的输出。按指导书命令在 `dist/release` 中解压两个附件，先运行 v1.0.1，停止后换回 v1.0.0；`#` 开头的行是记录时间的注释：

```text
# 2026-10-22 10:53:46 +0800 启动 v1.0.1
{"version":"1.0.1","commit":"20da6eed62da2492360d72559b1fae6040cde7c8","build_no":"3","run_id":"39713717369","go":"go1.26.9"}
# 2026-10-22 10:53:47 +0800 停止 v1.0.1，回退到 v1.0.0
{"version":"1.0.0","commit":"5c68dd213a4389cf51500b43be2729ecc880a22e","build_no":"2","go":"go1.26.9"}
```

回退时直接运行 `v1.0.0` Release 附件中的原二进制文件，没有重新构建。两个二进制文件的 SHA-256 如下，其中 `v1.0.0` 的值与任务 5 的记录一致：

```text
f684cdaf5547ddc34178cb7782504e1b29f326eed850ef61e8dc2cdc3f3bef49  enroll-conflict_1.0.0_linux_amd64/enroll-conflict
99650f311d8fd045d33410c8cfd83ccd24a98e19018abffff474fde268637066  enroll-conflict_1.0.1_linux_amd64/enroll-conflict
```

`v1.0.0` 的输出没有 `run_id` 字段，因为这个字段是 #7 才加入的，这也说明回退后运行的确实是旧版本的文件。两个服务均已用 `kill` 停止，`pgrep -a enroll-conflict` 无输出。

## 5. 数据

本轮时间统一使用 `Asia/Shanghai`（UTC+08:00）。GitHub 事件通过 API 读取，可在对应页面核对；`ci` 结束时间采用必需检查所在 `ci` job 的 `completed_at`。修复推送时间来自操作者提供的终端记录，按时间区间保留，不用提交时间替代推送时间。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | 2026-10-18 13:17:17 +08:00 | [PR #2](https://github.com/RyanShen30/ops-lab1-2/pull/2) |
| 第一次 `ci` 失败（ci job 结束） | 2026-10-18 13:17:35 +08:00 | [ci #3 的 ci job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100037612/job/117594257961) |
| 推送修复提交 | 2026-10-18 13:20:23～13:20:42 +08:00（终端记录区间） | [修复提交](https://github.com/RyanShen30/ops-lab1-2/commit/da2d9170a2ed5fa93fdc02470c944335bcbcc8e1)；原始记录见下方 |
| `ci` 通过（ci job 结束） | 2026-10-18 13:20:55 +08:00 | [ci #4 的 ci job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100223291/job/117594799272) |
| 合入 main | 2026-10-18 14:52:40 +08:00 | [PR #2](https://github.com/RyanShen30/ops-lab1-2/pull/2) |
| main push 的 ci 通过（ci job 结束） | 2026-10-18 14:52:58 +08:00 | [ci #7 的 ci job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39105597138/job/117610631633) |

修复推送前后的终端时间原文为 `2026-10-18T13:20:23+08:00`、`2026-10-18T13:20:42+08:00`。两者给出一个 19 秒的推送时间区间，不能将这 19 秒当作 CI 耗时。上表的通过事件对应首次修复后的 ci #4，后续清理注释的 ci #5 不替代该事件。

提交到发布的间隔，至少记录三个合入的 PR：

“第一个提交的时间”取 PR 中最早提交的作者时间，“合入时间”取 squash 合入时间，均来自 GitHub API；“对应 Release”是第一个包含该提交的版本。表中列出全部 7 个合入的 PR。

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| [#2](https://github.com/RyanShen30/ops-lab1-2/pull/2) | 2026-10-18 13:15:17 | 2026-10-18 14:52:40 | v1.0.0：2026-10-19 20:55:07 | 1 天 7 小时 39 分（31.7 小时） |
| [#3](https://github.com/RyanShen30/ops-lab1-2/pull/3) | 2026-10-18 14:59:32 | 2026-10-18 15:14:54 | v1.0.0：2026-10-19 20:55:07 | 1 天 5 小时 56 分（29.9 小时） |
| [#4](https://github.com/RyanShen30/ops-lab1-2/pull/4) | 2026-10-18 15:16:11 | 2026-10-18 15:18:58 | v1.0.0：2026-10-19 20:55:07 | 1 天 5 小时 39 分（29.7 小时） |
| [#5](https://github.com/RyanShen30/ops-lab1-2/pull/5) | 2026-10-19 12:06:46 | 2026-10-19 14:04:57 | v1.0.0：2026-10-19 20:55:07 | 8 小时 48 分（8.8 小时） |
| [#6](https://github.com/RyanShen30/ops-lab1-2/pull/6) | 2026-10-19 20:58:39 | 2026-10-19 21:33:58 | v1.0.1：2026-10-22 10:52:33 | 2 天 13 小时 54 分（61.9 小时） |
| [#7](https://github.com/RyanShen30/ops-lab1-2/pull/7) | 2026-10-21 23:35:49 | 2026-10-22 10:50:02 | v1.0.1：2026-10-22 10:52:33 | 11 小时 17 分（11.3 小时） |
| [#8](https://github.com/RyanShen30/ops-lab1-2/pull/8) | 2026-10-21 23:37:02 | 2026-10-22 10:50:49 | v1.0.1：2026-10-22 10:52:33 | 11 小时 16 分（11.3 小时） |

把间隔拆成“提交到合入”和“合入到发布”两段：

- #2～#5 的“合入到发布”分别为 30.0、29.7、29.6、6.8 小时，等的是任务 5 的发布工作流完成。
- #6 合入后等了 61.3 小时才随 v1.0.1 发布。
- #7、#8 的“提交到合入”约 11.2 小时，其中大部分是夜间等待组员审批；合入后 2 分 31 秒和 1 分 44 秒即发布。

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

“时长”按第 1 次尝试的 `run_started_at` 到 `updated_at` 计算，备注中另列 `ci` job 本身的时长。开始时间为 +08:00。下表采用 ci #1～#27 的首次尝试作为固定样本，统计截止时间为 2026-10-22 10:58:33 +08:00；后续运行不计入本表。O2 的重跑另列于第 7 节，ci #26 的第二次尝试见问题 3。

| 运行 | 事件 | 时长 | 备注 |
|---|---|---|---|
| [#1](https://github.com/RyanShen30/ops-lab1-2/actions/runs/38877464989/attempts/1) | push | 44 秒 | 10-16 17:00，main `ba1c247`，成功，ci job 40 秒；main 上第一次运行，缓存未命中，运行结束时保存缓存 |
| [#2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/38902466874/attempts/1) | PR | 28 秒 | 10-16 20:42，PR #1 `8ed1465`，成功，ci job 25 秒 |
| [#3](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100037612/attempts/1) | PR | 15 秒 | 10-18 13:17，PR #2 `e0bbb94`，失败，ci job 12 秒；第一轮演练，Test 失败 |
| [#4](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100223291/attempts/1) | PR | 17 秒 | 10-18 13:20，PR #2 `da2d917`，成功，ci job 14 秒 |
| [#5](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100712651/attempts/1) | PR | 20 秒 | 10-18 13:29，PR #2 `42ed4a9`，成功，ci job 17 秒 |
| [#6](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39101086698/attempts/1) | PR | 9 秒 | 10-18 13:35，PR #2 `343f804`，成功，ci job 6 秒 |
| [#7](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39105597138/attempts/1) | push | 17 秒 | 10-18 14:52，main `fdbd649`，成功，ci job 14 秒 |
| [#8](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106019814/attempts/1) | PR | 20 秒 | 10-18 14:59，PR #3 `d1e21a4`，成功，ci job 17 秒 |
| [#9](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106084105/attempts/1) | PR | 12 秒 | 10-18 15:00，PR #3 `4844565`，失败，ci job 10 秒；第二轮演练补测试后，Test 失败 |
| [#10](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106121523/attempts/1) | PR | 9 秒 | 10-18 15:01，PR #3 `2c174f9`，成功，ci job 7 秒 |
| [#11](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106167982/attempts/1) | PR | 11 秒 | 10-18 15:02，PR #3 `9b2e133`，成功，ci job 8 秒 |
| [#12](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106946423/attempts/1) | push | 19 秒 | 10-18 15:14，main `7abf6c4`，成功，ci job 16 秒 |
| [#13](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39107045097/attempts/1) | PR | 16 秒 | 10-18 15:16，PR #4 `152e7e5`，成功，ci job 13 秒 |
| [#14](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39107198788/attempts/1) | push | 14 秒 | 10-18 15:19，main `2fa5339`，成功，ci job 11 秒 |
| [#15](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39197858128/attempts/1) | PR | 11 秒 | 10-19 12:06，PR #5 `9fc725b`，成功，ci job 9 秒 |
| [#16](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39198002540/attempts/1) | PR | 15 秒 | 10-19 12:08，PR #5 `c26f469`，成功，ci job 11 秒 |
| [#17](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39207874767/attempts/1) | push | 20 秒 | 10-19 14:04，main `5c68dd2`，成功，ci job 16 秒 |
| [#18](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39256355198/attempts/1) | PR | 12 秒 | 10-19 20:59，PR #6 `9bc37b1`，成功，ci job 8 秒 |
| [#19](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39261248331/attempts/1) | push | 20 秒 | 10-19 21:34，main `e2b9edb`，成功，ci job 17 秒 |
| [#20](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627154197/attempts/1) | PR | 23 秒 | 10-21 23:36，PR #7 `8ca9120`，成功，ci job 18 秒 |
| [#21](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627305858/attempts/1) | PR | 13 秒 | 10-21 23:37，PR #8 `16e58f1`，成功，ci job 10 秒 |
| [#22](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627414974/attempts/1) | PR | 16 秒 | 10-21 23:37，PR #9 `2d9ef69`，成功，ci job 13 秒；O2 有缓存组，另有 5 次重跑见第 7 节 |
| [#23](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627904510/attempts/1) | PR | 33 秒 | 10-21 23:41，PR #9 `704c037`，成功，ci job 29 秒；O2 无缓存组（cache: false），另有 5 次重跑见第 7 节 |
| [#24](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39706540218/attempts/1) | PR | 18 秒 | 10-22 09:35，PR #10 `2353dcb`，成功，ci job 15 秒 |
| [#25](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713533558/attempts/1) | push | 32 秒 | 10-22 10:50，main `7e413b9`，成功，ci job 29 秒；Package 步骤 11 秒，此前 push 运行只需约 1 秒 |
| [#26](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713608095/attempts/1) | push | 33 秒 | 10-22 10:50，main `20da6ee`，成功，ci job 30 秒；Package 步骤 13 秒；问题 3 对其执行了一次重跑（第 2 次尝试） |
| [#27](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39714274727/attempts/1) | PR | 15 秒 | 10-22 10:58，PR #10 `83c6dde`，成功，ci job 14 秒 |

汇总：PR 运行共 19 次，时长 9～33 秒，中位数 15 秒；push 运行共 8 次，时长 14～44 秒，中位数 20 秒。push 运行多出打包和上传两步，所以略长。

- #1 是唯一一次没有可用缓存的运行，耗时 44 秒。#23 关闭了缓存，耗时 33 秒。
- #25、#26 的 Package 步骤比之前的 push 运行多出约 10 秒，原因未进一步验证。
- 两次失败（#3、#9）都在 Test 步骤，分别用时 15 秒和 12 秒就给出了结果。

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

所用数据：

- **拦住的：** 第一轮 `weeksOverlap` 缺陷由 [ci #3](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100037612) 的 Test 步骤拦住，运行 15 秒；第二轮补测试后，由 [ci #9](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106084105) 拦住。
- **没拦住的：**
  - 第二轮只含缺陷时，[ci #8](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106019814) 通过，只剩审批一项未满足。
  - PR #3 合并时没有清理提交说明（见第 3 节），gate 不检查提交说明。
  - PR #8 在 PR 中测试的是“#8 合入 `e2b9edb`”的状态，而 main 上实际得到的 `20da6ee` 是“#8 合入 #7 之后”的状态，后者只在合入后由 main 的 ci #26 验证（见问题 2）。
- **第一轮多花的时间：**
  - 第一次 ci 结束（13:17:35）到 ci #4 结束（13:20:55），共 3 分 20 秒（200 秒）。
  - ci #4 于 13:20:38 开始，所以修复推送不晚于 13:20:38；终端记录的推送区间为 13:20:23～13:20:42。
  - 等待 ci：17～32 秒，占 9%～16%。
  - 人工修改：13:17:35 到推送之间，2 分 48 秒～3 分 3 秒，占 84%～91%。
- **一次 ci 的时长：** PR 运行中位数 15 秒，push 运行中位数 20 秒（第 5 节）。
- **占“提交到发布的间隔”的比例：** 间隔为 8.8～61.9 小时。单次 ci 不到 0.07%（20 秒 ÷ 8.8 小时）。即使把 PR #2 的全部运行（ci #3～#7，共 78 秒）加起来，也只占其 31.7 小时间隔的 0.07%。

结论：gate 拦住的是测试能覆盖的缺陷：两轮演练中，只要有对应的测试，Test 步骤都在约 15 秒内报告失败并阻止合并。没有拦住的是测试覆盖范围之外的东西：第二轮缺陷在补测试之前一路通过，只剩人工审批这一道；提交说明的质量（PR #3）和多个 PR 先后合入后的组合状态（#8 合入后的 `20da6ee`）也都不在检查范围内。第一轮因失败多花约 3 分 20 秒，其中等待 `ci` 只有 17～32 秒（9%～16%），其余 84%～91% 是人工定位和修改，失败的成本主要在人而不在流水线。一次 `ci` 约 15～20 秒，即使把一个 PR 的全部运行加起来，也不到“提交到发布的间隔”的 0.1%；间隔的主体是等待审批（#7、#8 约 11 小时，跨越夜间）和等待发布（#2～#4 约 30 小时，#6 约 61 小时）。因此在本组，自动检查几乎不增加交付时间，真正决定交付速度的是人工审批和发布节奏。第二轮补测试之后，测试仍发现不了“测试和代码按同一个错误理解编写”的问题、没有想到的边界和规则、性能与并发问题，以及只在打 tag 时才运行的发布配置错误；这些要靠评审、main 上的再次验证，或要求合并前先更新到最新 main 来补足。成立条件是：示例很小且没有第三方依赖，`ci` 使用了缓存（无缓存时约 28 秒，见 O2），样本只有两轮演练和 7 个 PR，且组员在课外时间异步协作；在测试耗时更长、PR 更频繁的真实团队中，`ci` 时长和排队所占的比例会明显更高。

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

所用数据：本组只允许 squash 合并。以下取自三次 PR 运行日志中 “Print event and revision” 一步的输出；提交是否在 main 历史中，用 `git merge-base --is-ancestor` 判断。

| PR 与运行 | `github.ref` | `github.sha` | `pull_request.head.sha` | squash 后 main 上的提交 |
|---|---|---|---|---|
| #2，[ci #3](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100037612) | `refs/pull/2/merge` | `ff2bc18`，父提交为 `ba1c247` 和 `e0bbb94` | `e0bbb94`，父提交为 `ba1c247` | `fdbd649`，父提交为 `ba1c247` |
| #7，[ci #20](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627154197) | `refs/pull/7/merge` | `d5f1d90`，父提交为 `e2b9edb` 和 `8ca9120` | `8ca9120`，父提交为 `e2b9edb` | `7e413b9`，父提交为 `e2b9edb` |
| #8，[ci #21](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627305858) | `refs/pull/8/merge` | `44b6ed3`，父提交为 `e2b9edb` 和 `16e58f1` | `16e58f1`，父提交为 `e2b9edb` | `20da6ee`，父提交为 `7e413b9` |

- 每次 PR 运行中，两个值都不同。`github.sha` 是 GitHub 为 PR 生成的临时合并提交，有两个父提交；`head.sha` 是分支上的最后一个提交。
- 这两个提交都不在 main 的历史中。普通克隆默认不拉取 `refs/pull/*`；PR 分支删除后，仍可通过 PR 引用查询分支提交，或通过 GitHub API 查询相关提交。因此，普通克隆中没有这些提交，不等于它们无法追溯；但提交仍可查询，也不等于它属于 main 历史。
- squash 后 main 上的提交号与它们都不同。
- #7、#8 先后合入，#8 在 PR 中测试的基线是 `e2b9edb`，而 main 上 `20da6ee` 的父提交是 `7e413b9`。规则集没有要求合并前先更新到最新 main，所以 `20da6ee` 这个状态第一次被测试，是合入后的 main push 运行 [ci #26](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713608095)。
- PR 运行不执行 Package 和 Upload 两步（`if: github.event_name == 'push'`），不产出制品。

结论：本组只允许 squash 合并。PR 运行中的 `github.sha` 与 `pull_request.head.sha` 不同：前者是 GitHub 为 `refs/pull/N/merge` 生成的临时合并提交，父提交是当时的 main 和分支头，`ci` 测试的就是这份“假如现在合并”的代码；后者是分支上的最后一个提交。两者都不在 main 的历史中：squash 在 main 上另外生成一个只有一个父提交的新提交（如 #7 的 `7e413b9`），普通克隆默认不拉取 PR 引用，分支删除后仍可经 PR 引用或 GitHub API 查询相关提交；`trace.sh` 的“主干”检查会因它们不属于 main 历史而失败。而且 PR 中测过的状态不一定就是合入后的状态：#8 测的是基于 `e2b9edb` 的 `44b6ed3`，合入后的 `20da6ee` 却基于 `7e413b9`，这个组合只有 main 上的 ci #26 测过。所以如果用 PR 运行构建待发布的制品，制品里的提交号会指向一个不属于 main 历史的提交，即使它仍可查询，也无法通过 `trace.sh` 的主干检查，而且不一定代表 main 上实际合入的代码。本实验选择经 main 的 push 运行验证的主干提交或指向它的 tag 作为发布来源；本组 `ci` 只在 push 到 main 时打包，release gate 也拒绝不在 main 历史上的 tag，正是出于这个原因。上述主干归属结论以本组使用 squash 合并为条件，分支是否删除不改变提交的祖先关系：若改用 merge commit，`head.sha` 会作为第二个父提交进入 main 历史，但 main 上的合并提交仍与 PR 中的临时合并提交不同；若开启“合并前须更新到最新 main”，PR 中测试的状态与合入结果一致，但提交号仍然不同。

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

构建条件：2026-10-22 在任务 6 的同一台服务器上，从 GitHub 新克隆小组仓库并检出 `v1.0.1`（`20da6ee`），工作区干净。Go 1.26.9 linux/amd64，umask 为 `0002`。固定值为 `VERSION=1.0.1 BUILD_NO=100 RUN_ID=q3-local COMMIT=20da6ee…`，③ 将 `BUILD_NO` 改为 `101`。⑤ 在 ④ 完成 5 秒后构建。④、⑤ 完成后执行 `git checkout -- Makefile`，`git status --porcelain` 无输出。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | `919d66ff05afdcebb17c330c00bd6f92860b42bc84873fe0d0509982d8eee544` | `a6cdcc16034c280a41ba3394335efea4360e0e7939f391993eed56141fd7cf5a` |
| ② | 与 ① 完全相同，再构建一次 | `919d66ff05afdcebb17c330c00bd6f92860b42bc84873fe0d0509982d8eee544` | `a6cdcc16034c280a41ba3394335efea4360e0e7939f391993eed56141fd7cf5a` |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | `37d2e805ca880a3b7af0370f18a4a4bd7ff4c187a3f369a34a2fd70ade2a91f2` | `095d975703d47b0368c466d7fa18ad676bb5a8a654901dcbfc491d84469014fe` |
| ④ | 去掉 `--mtime=@0`，其余同 ① | `9c30d23388a15eb8988c62cf506bf41f01ea5c5d21ea0902027ad25f4f67aafe` | `6a55164a55024b4d6f53b85216fe615cb3d50e1d04b921c8b771b36908adb48e` |
| ⑤ | 同 ④，间隔几秒后再构建一次 | `9c30d23388a15eb8988c62cf506bf41f01ea5c5d21ea0902027ad25f4f67aafe` | `30c2c867b748f2cde397c48f739fb77f7d4a063b6468f0734c542058ccb3db2f` |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | [ci #26](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713608095) 第 [1](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713608095/attempts/1)、[2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713608095/attempts/2) 次尝试均为 `d7b37e6a51fa5fbb49f4aa73da6a42c1ecaa33099f3b1a750a0a92a3bdac9f31` | 两次均为 `62dad40215b1d5851488e3fe7105117eb63a92ce3a487e3173945c17ce060e29` |

补充数据：

- **① 与 ④ 的 vcs 信息：**
  - ①：`mod enroll-conflict v1.0.1`，`vcs.revision=20da6ee…`，`vcs.time=2026-10-22T02:50:49Z`，`vcs.modified=false`。
  - ④：`mod enroll-conflict v1.0.1+dirty`，`vcs.modified=true`；`vcs.revision` 和 `vcs.time` 与 ① 相同。
- **Actions 重跑：** 两次尝试的 `-ldflags` 均为 `-X main.buildNo=26 -X main.runID=39713608095`。重跑不改变 `run_number` 和 `run_id`。
- **按发布参数在本地重建两个版本：** 在新克隆中分别检出 `v1.0.0`、`v1.0.1`，用 release 当时的 `VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 执行 `make package`。

  | 重建 | 二进制 sha256 | tar.gz sha256 | 与 Release 附件比较 |
  |---|---|---|---|
  | v1.0.0，umask `0002` | `f684cdaf…` | `0688facb…` | 二进制相同，tar.gz 不同 |
  | v1.0.0，umask `0022` | `f684cdaf…` | `d2936cc9…` | 两者都相同 |
  | v1.0.1，umask `0002` | `99650f31…` | `d2d31579…` | 二进制相同，tar.gz 不同 |
  | v1.0.1，umask `0022` | `99650f31…` | `d3a5338a…` | 两者都相同 |

  `tar -tvzf` 显示，umask `0002` 时归档中记录的权限为 `rwxrwxr-x`、`rw-rw-r--`，Release 附件中为 `rwxr-xr-x`、`rw-r--r--`。两份 `BUILD_INFO` 内容相同。runner 的 umask 为 `0022`。
- **在 `git worktree` 中构建：** 同一提交在 `git worktree` 中构建时，`.git` 是文件而不是目录，Go 没有写入任何 vcs 信息（`mod enroll-conflict (devel)`）。此时 ④ 的二进制与 ① 相同，都是 `4cd45060…`。上表因此改在普通克隆中构建。

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

所用数据：上表与补充数据；任务 0 中只改注释、不提交时，`BUILD_INFO` 不变而二进制哈希改变（第 2 节）；O1 中镜像内的二进制与 Release 附件相同（第 7 节）。

结论：①② 和 Actions 的两次尝试完全相同，因为输入完全相同：同一提交、同一组参数、同一 Go 版本；Makefile 又固定了 `-trimpath`、`CGO_ENABLED=0`，以及 tar 的排序、属主和时间戳；重跑也不改变 `run_number` 和 `run_id`，`-ldflags` 因此不变。③ 两项都变，因为 `BUILD_NO` 经 `-ldflags` 写进了二进制，也写进了 `BUILD_INFO`，这是有意记录的身份信息。④ 的二进制与 ① 不同，原因不在 `--mtime`，而在于改了 Makefile 使工作区变脏，Go 写入了 `vcs.modified=true` 和 `v1.0.1+dirty`，与任务 0 只改注释时相同；④⑤ 的二进制相同而 tar.gz 不同，因为去掉 `--mtime=@0` 后，归档记录了每次构建时文件的修改时间。按发布参数在本地重建两个版本，二进制与 Release 附件逐位相同，tar.gz 却只有在 umask 与 runner 一致（`0022`）时才相同，说明归档还记录了文件权限。据此，制品中应当包含说明“它是什么”的信息：提交号、版本号、构建号、`run_id`、Go 版本和工作区是否干净；不应包含随构建时机和机器偶然变化的信息，如构建时间、文件修改时间、本机的用户、路径和 umask，这些需要时应记在制品之外的运行记录里。`release` 重新编译，是因为版本号要经 `-ldflags` 写进二进制，而 `ci` 的主干制品写的是 `0.0.0-main` 和 `ci` 的构建号，不能直接作为 v1.0.1 发布。这与“一次构建、多处部署”冲突：发布出去的二进制（`99650f31…`）并不是 `ci` 测过的那个文件（`d7b37e6a…`），两者等价只能靠“同一提交加可重复构建”来论证；代价是多一次构建，而且一旦 Go 补丁版本、umask 之类的环境差异破坏了可重复性，这种等价就不成立了。成立条件是 Go 版本固定为 1.26.9、在普通克隆中构建（在 `git worktree` 中 Go 不写 vcs 信息，④ 与 ① 的二进制相同），且环境与 runner 一致；O1 的镜像直接复用 Release 附件，则是“一次构建、多处部署”的做法。

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

所用数据：

- **回退时用的文件：** 两次 `/version` 的输出见第 4 节。回退时运行的是 `v1.0.0` Release 附件中的原二进制文件，SHA-256 为 `f684cdaf…`，与任务 5 记录一致，没有重新构建。v1.0.0 的输出没有 `run_id`，v1.0.1 的有。
- **从源码重建 v1.0.0：** 见问题 3 的补充数据。二进制与原附件相同；`tar.gz` 只有在 umask 为 `0022` 时才相同。
- **附件被替换（本地模拟，未改动真实 Release）：** 把 `v1.0.1` 的二进制放进 `v1.0.0` 的包，按 Makefile 的方式重新打包，并生成新的 `.sha256`。
  - 对这个文件运行 `trace.sh`：第 1 步输出 `enroll-conflict_1.0.0_linux_amd64.tar.gz: OK`，BUILD_INFO 仍显示 `version=1.0.0`、`commit=5c68dd2…`，退出码为 0。
  - 用第 4 节记录的发布当时的值 `d2936cc9…` 比对：`sha256sum -c` 输出 `FAILED`。
  - 包内二进制的 SHA-256 为 `99650f31…`，不是 `f684cdaf…`。
- **`trace.sh` 的校验方式：** 第 1 步只与同目录下的 `.sha256` 比对（`scripts/trace.sh` 的“1. 校验和”一段）。
- **Immutable releases：** 截至 2026-10-22，GitHub API 返回的 `v1.0.0`、`v1.0.1` 两个 Release 的 `immutable` 字段均为 `false`，仓库未启用 Immutable releases。

结论：v1.0.1 发布后发现缺陷，应先把服务回退到 v1.0.0，即直接运行已发布的旧附件；再通过 PR 修复并合入 main，发布新的修订版本 v1.0.2，同时在 v1.0.1 的 Release 说明中写明问题，v1.0.1 本身不删除、不复用。不能删除 tag 后用同一版本号重新发布，因为 v1.0.1 可能已被下载、部署或缓存（包括 `1.0.1` 镜像），同一个版本号若对应两份内容，仅凭版本号就不能唯一确定制品；如果附件及其配套的 `.sha256` 被同时替换，`trace.sh` 也可能仍然校验通过，但与第 4 节可信、未被篡改的发布时原始 SHA256 比较，就能发现附件内容发生了变化。任务 6 回退时运行的是 v1.0.0 Release 附件中的原文件（`f684cdaf…`），没有重新构建；两次 `/version` 分别显示 `1.0.1`、`20da6ee`、构建号 3 和 `1.0.0`、`5c68dd2`、构建号 2，后者也没有 v1.0.1 才加入的 `run_id` 字段，说明回退到的正是旧版本的原文件，整个切换只用了几秒。从源码重新构建 v1.0.0，本组实验中二进制与原附件逐位相同，但 tar.gz 只有在 umask 与 runner 一致时才相同；而且这要求 Go 版本、构建参数乃至 `run_id` 全部与当时一致，所以回退应当使用原附件，而不是现场重建。如果有人删掉 v1.0.0 的附件，换上新文件和对应的新 `.sha256`，`trace.sh` 发现不了，因为它只与同目录的 `.sha256` 比对，本地模拟中它照常输出 OK、退出码为 0；而用第 4 节记录的发布当时的值 `d2936cc9…` 比对，会立即失败。本组两个 Release 目前 `immutable` 均为 `false`；启用 Immutable releases 后，Release 发布后附件和 tag 都不能再修改或删除，能从平台层面阻止这种替换和对版本号的复用。成立条件是：第 4 节的记录保存在受保护的 main 历史中，没有被一起篡改；并且服务是无状态的，若新版本改了数据格式或数据库结构，回退时还要考虑数据兼容性。

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 已完成。[PR #8](https://github.com/RyanShen30/ops-lab1-2/pull/8) 增加 `image` job，`v1.0.1` 由 [image job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369/job/119610931604) 推送镜像，摘要为 `sha256:9898a0e1aabd30feaa871573a61b4272f89c8291ea5c829fe93fabf465c8e6d1`。运行容器的主机为 `x86_64`（linux/amd64），可匿名拉取，详见下方“选做 O1”。 |
| O2 缓存对反馈时间的影响 | 已完成。实验 [PR #9](https://github.com/RyanShen30/ops-lab1-2/pull/9) 不合并；预期、全部 12 次运行与结论见下方“选做 O2”。 |
| O3 迁移到小组项目 | 用组员 Shundi-Wu 已有的项目 PaperMate 对照，流程相同；仓库为私有，现场演示。见下方“选做 O3”。 |

### 选做 O1：容器镜像

**工作流。** [PR #8](https://github.com/RyanShen30/ops-lab1-2/pull/8) 在 `release.yml` 中新增 `image` job（`needs: release`）。权限只在该 job 内声明为 `contents: read` 和 `packages: write`，步骤如下：

1. 用 `gh release download` 下载本次 Release 的 `tar.gz` 和 `.sha256`，执行 `sha256sum -c` 后解压。
2. 以解压后的制品目录为构建上下文，用仓库中的 `Dockerfile` 构建镜像。镜像中不重新编译，放入的就是 Release 附件里的二进制文件和 `BUILD_INFO`。
3. 推送到 `ghcr.io/${GITHUB_REPOSITORY,,}:<版本号>`，并在日志中打印摘要。

**v1.0.1 的镜像。** [image job](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39713717369/job/119610931604) 校验附件输出 `enroll-conflict_1.0.1_linux_amd64.tar.gz: OK`，推送日志为 `1.0.1: digest: sha256:9898a0e1aabd30feaa871573a61b4272f89c8291ea5c829fe93fabf465c8e6d1 size: 735`。镜像即 [ghcr.io/ryanshen30/ops-lab1-2](https://github.com/users/RyanShen30/packages/container/package/ops-lab1-2)，按摘要引用为：

```text
ghcr.io/ryanshen30/ops-lab1-2@sha256:9898a0e1aabd30feaa871573a61b4272f89c8291ea5c829fe93fabf465c8e6d1
```

**匿名拉取与按摘要运行。** 2026-10-22 在任务 6 的同一台服务器上操作。该机 root 的 Docker 没有登录任何 registry，以下拉取均为匿名；另外向 `ghcr.io/token` 匿名申请 pull token 后请求该摘要的 manifest，返回 HTTP 200。实际输出：

```text
$ uname -m; docker info 架构
x86_64
x86_64 linux Docker 29.8.2
$ docker pull ghcr.io/ryanshen30/ops-lab1-2@sha256:9898a0e1aabd30feaa871573a61b4272f89c8291ea5c829fe93fabf465c8e6d1
ghcr.io/ryanshen30/ops-lab1-2@sha256:9898a0e1aabd30feaa871573a61b4272f89c8291ea5c829fe93fabf465c8e6d1
$ docker image inspect
linux/amd64 RepoDigests=[ghcr.io/ryanshen30/ops-lab1-2@sha256:9898a0e1aabd30feaa871573a61b4272f89c8291ea5c829fe93fabf465c8e6d1] labels: version=1.0.1 revision=20da6eed62da2492360d72559b1fae6040cde7c8 source=https://github.com/RyanShen30/ops-lab1-2
$ docker run --rm -d -p 127.0.0.1:18081:8080 ghcr.io/ryanshen30/ops-lab1-2@sha256:9898a0e1aabd30feaa871573a61b4272f89c8291ea5c829fe93fabf465c8e6d1
b9c033a37732
$ curl -s 127.0.0.1:18081/version
{"version":"1.0.1","commit":"20da6eed62da2492360d72559b1fae6040cde7c8","build_no":"3","run_id":"39713717369","go":"go1.26.9"}
```

运行容器的主机架构为 `x86_64`，镜像为 `linux/amd64`，二者一致，不需要 `--platform`。

**镜像中没有重新编译。** 用 `docker create` 加 `docker cp` 从镜像中取出 `/enroll-conflict`，它的 SHA-256 为 `99650f311d8fd045d33410c8cfd83ccd24a98e19018abffff474fde268637066`。这与 `v1.0.1` Release 附件中的二进制文件、release job 打包日志三者相同，镜像里运行的就是发布出去的那个文件。完成后已停止容器，并用 `docker rmi` 删除本地镜像。

**思考。**

- **为什么镜像 job 只能写在同一个工作流中：** release job 用 `GITHUB_TOKEN` 创建 Release。由 `GITHUB_TOKEN` 产生的事件（`workflow_dispatch` 和 `repository_dispatch` 除外）不会触发新的工作流运行，这是 GitHub 为防止工作流相互触发形成循环而设的限制。因此，另写一个监听 `release: published` 的工作流不会运行，镜像只能作为同一工作流中 `needs: release` 的后续 job。若改用个人访问令牌创建 Release 虽能触发，但要在仓库中保存长期有效的凭据。
- **可见性：** 实测该包可匿名拉取。镜像由本仓库工作流用 `GITHUB_TOKEN` 推送，并通过 `org.opencontainers.image.source` 标签关联到仓库；在本仓库公开的条件下，包没有停留在默认的私有状态。
- **按摘要运行而不按 tag 运行：** tag `1.0.1` 可以被重新推送、指向别的内容，摘要则由镜像内容决定。按摘要运行，保证运行的正是上面记录的这一份镜像。

### 选做 O2：缓存对反馈时间的影响

**预期。** 写在 [PR #9](https://github.com/RyanShen30/ops-lab1-2/pull/9) 的描述中。PR 于 2026-10-21 23:37:51 +08:00 创建，早于第一次运行（23:37:55 创建 job）。要点：示例没有第三方依赖，模块缓存为空；但 setup-go 同时缓存构建缓存 `GOCACHE`，而 Go 1.20 起不再附带预编译的标准库，`go vet`、`go test` 须先编译 `net/http` 等标准库包。预计无缓存时多花 10～20 秒编译，扣除下载缓存的 1～3 秒，有缓存时每次约快 10 秒。

**方法。** 两组在同一个 PR 的同一份代码上运行。提交 `2d9ef69` 显式写 `cache: true`（与默认值相同），为有缓存组，即 ci #22；提交 `704c037` 改为 `cache: false`，为无缓存组，即 ci #23。两组交替执行 Re-run all jobs，每组首次运行加 5 次重跑，共 6 次，全部在 23:37～23:48 +08:00 之间完成。时长取 `ci` job 及各步骤的 `started_at`、`completed_at`，精度为 1 秒；“排队”指 job 创建到开始的时间，不计入 job 时长。日志核对：有缓存组 6 次均为 `Cache restored from key: setup-go-Linux-x64-ubuntu24-go-1.26.9-e6d3eb…`，大小约 43 MB。两组的 Go 1.26.9 均取自 runner 预装的工具缓存（`Found in cache @ /opt/hostedtoolcache/go/1.26.9/x64`），因此两组唯一的差别是是否恢复 setup-go 缓存。

| 组 | 运行 | 开始（+08:00） | ci job 时长（秒） | Set up Go | Go vet | Test | 排队 |
|---|---|---|---|---|---|---|---|
| 有缓存 | [ci #22 第 1 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627414974/attempts/1) | 23:37:56 | 13 | 2 | 3 | 2 | 1 |
| 有缓存 | [ci #22 第 2 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627414974/attempts/2) | 23:41:52 | 20 | 3 | 7 | 3 | 3 |
| 有缓存 | [ci #22 第 3 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627414974/attempts/3) | 23:43:00 | 22 | 4 | 6 | 4 | 2 |
| 有缓存 | [ci #22 第 4 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627414974/attempts/4) | 23:44:17 | 16 | 3 | 5 | 4 | 2 |
| 有缓存 | [ci #22 第 5 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627414974/attempts/5) | 23:45:35 | 15 | 2 | 5 | 4 | 3 |
| 有缓存 | [ci #22 第 6 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627414974/attempts/6) | 23:46:51 | 10 | 1 | 1 | 2 | 2 |
| 无缓存 | [ci #23 第 1 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627904510/attempts/1) | 23:41:08 | 29 | 2 | 18 | 4 | 2 |
| 无缓存 | [ci #23 第 2 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627904510/attempts/2) | 23:42:26 | 26 | 0 | 17 | 4 | 2 |
| 无缓存 | [ci #23 第 3 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627904510/attempts/3) | 23:43:36 | 29 | 1 | 19 | 4 | 2 |
| 无缓存 | [ci #23 第 4 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627904510/attempts/4) | 23:44:42 | 29 | 1 | 21 | 3 | 2 |
| 无缓存 | [ci #23 第 5 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627904510/attempts/5) | 23:46:09 | 28 | 1 | 17 | 3 | 3 |
| 无缓存 | [ci #23 第 6 次](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39627904510/attempts/6) | 23:47:07 | 27 | 1 | 18 | 3 | 2 |

**结论。** 有缓存组的 job 时长为 10～22 秒，中位数 15.5 秒；无缓存组为 26～29 秒，中位数 28.5 秒。两组区间不重叠，中位数相差约 13 秒，与预期的“约快 10 秒”方向一致、幅度略大。差别几乎全部出现在 `Go vet`：中位数分别为 5 秒和 18 秒。`Test` 两组都是 2～4 秒，因为同一个 job 中 `go vet` 已把标准库的编译结果写进 `GOCACHE`，`go test` 直接复用。`Set up Go` 在有缓存时因下载约 43 MB 的缓存略慢，中位数为 2.5 秒对 1 秒。所以对本示例来说，缓存节省的不是依赖下载，而是标准库的编译。

**波动来源与局限。**

- 有缓存组 6 次都命中同一个缓存，`Go vet` 仍在 1～7 秒之间波动，job 时长在 10～22 秒之间。这部分波动来自共享 runner 每次分配到的机器的 CPU 和磁盘性能；缓存下载速度在 12～129 MB/s 之间，也受网络影响。
- 时间戳精度为 1 秒，单个步骤只有 0～4 秒时相对误差很大；排队时间（1～3 秒）不计入 job 时长，但计入开发者实际等待的时间。
- 每组只有 6 次，集中在同一天深夜的 10 分钟内，只覆盖一份代码。不同时段的 runner 负载不同，结论只说明本示例在 `ubuntu-24.04` runner 上大约能省十几秒，不能推广为固定比例。有第三方依赖的项目还会节省模块下载时间，差距可能更大。
- 缓存键只由 Go 版本和 `go.mod` 的哈希决定，setup-go 只在未命中时保存。本组 `go.mod` 一直未变，命中的始终是 2026-10-16 main 上第一次 push 运行保存的缓存。随着代码变化，其中业务包的编译结果会过期，但标准库部分仍然有效，所以这个结论在 `go.mod` 和 Go 版本不变期间成立。

### 选做 O3：在 PaperMate 上对照

PaperMate（`Shundi-Wu/PaperMate`，主干为 `dev`）是 Shundi-Wu 已有的项目，由 Java 后端、前端和 Python AI 服务三部分组成。它的流水线在 2026 年 7 月搭建，早于本实验，这里按本实验的流程逐项对照。仓库为私有，以下证据在现场登录后演示。

| 本实验的流程 | PaperMate 中的对应做法 |
|---|---|
| 改动经 PR 合入主干，`ci` 通过才能合并 | PR 合入 `dev`。规则集 `protect dev` 为 Active，Bypass list 为空，禁止删除和强制推送，要求三个服务的检查和三个镜像的构建共 6 项必需检查通过 |
| 只在主干上产出制品 | PR 运行只构建镜像、不推送；push 到 `dev` 或 `main` 时才推送到 ghcr |
| 制品携带提交号 | 镜像 tag 为完整的 `github.sha`，并带 `org.opencontainers.image.revision` 和 `source` 标签 |
| 发布与 release gate | `dev` 上 push 触发的 CI 成功后，由 `workflow_run` 触发部署；部署前确认镜像已推送，部署配置也按同一提交读取 |
| 回退 | 部署脚本先备份数据库，健康检查失败时自动回退到上一次部署的提交；也可用 `workflow_dispatch` 指定任一已推送的提交重新部署 |

一次完整的流程（+08:00）：PR #56 的 CI（run `30601296252`）于 2026-07-31 11:19:41～11:24:09 通过，11:24:22 合入 `dev`，得到提交 `232e1d2`；`dev` 上的 CI（run `30601494290`）于 11:24:25～11:28:17 推送三个镜像；随后部署（run `30601656237`）于 11:28:20～11:39:02 成功。

与本实验的差异：

- 制品是三个容器镜像，不是 tar.gz。提交号记在镜像 tag 和标签里，没有 `BUILD_INFO` 和 `.sha256`。
- 没有语义化版本号和 Release，版本就是提交 SHA，发布就是部署到服务器。
- 主干是 `dev`，合并方式为 merge commit，不是 squash；规则集没有要求人工审批，只有自动检查。
- runner 使用 `ubuntu-latest`，Java、Node 只固定到主版本，构建环境不如本实验固定。
- 服务有数据库。回退时只回退应用镜像，数据库迁移镜像保持新版本，因为迁移只能前进。这正是问题 4 中“服务无状态”这一条件不成立时的做法。

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| RyanShen30 | [#2](https://github.com/RyanShen30/ops-lab1-2/pull/2) | [#3](https://github.com/RyanShen30/ops-lab1-2/pull/3#pullrequestreview-5561903762)、[#4](https://github.com/RyanShen30/ops-lab1-2/pull/4#pullrequestreview-5561917120)、[#6](https://github.com/RyanShen30/ops-lab1-2/pull/6#pullrequestreview-5572324223)、[#7](https://github.com/RyanShen30/ops-lab1-2/pull/7#pullrequestreview-5608701517)、[#8](https://github.com/RyanShen30/ops-lab1-2/pull/8#pullrequestreview-5608705542) |
| Liuxiang400 | [#3](https://github.com/RyanShen30/ops-lab1-2/pull/3)、[#4](https://github.com/RyanShen30/ops-lab1-2/pull/4) | [#2](https://github.com/RyanShen30/ops-lab1-2/pull/2#pullrequestreview-5561740627)、[#5](https://github.com/RyanShen30/ops-lab1-2/pull/5#pullrequestreview-5567500720) |
| ashkirakira | [#5](https://github.com/RyanShen30/ops-lab1-2/pull/5)、[#6](https://github.com/RyanShen30/ops-lab1-2/pull/6) | — |
| Shundi-Wu | [#7](https://github.com/RyanShen30/ops-lab1-2/pull/7)、[#8](https://github.com/RyanShen30/ops-lab1-2/pull/8) | [#5](https://github.com/RyanShen30/ops-lab1-2/pull/5#pullrequestreview-5567512409) |
