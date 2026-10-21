# 实验 1 记录：第 <组号> 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 <组号> 组 |
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
| 第一轮：修复后通过的 PR 运行 | [ci #4](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100223291) 通过；随后清理临时注释的 [ci #5](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39100712651) 也通过。 |
| 第一轮：合入 main 后的运行（含制品） | [ci #7](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39105597138)，事件为 push，分支 main，结果 success；Package 和 Upload 均成功，产出 build-7-attempt-1 制品。 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | 将 Valid 中拒绝 End < Start 改为拒绝 End <= Start，导致起止节次相同的合法单节课程被拒绝。Section 注释规定起止节次均包含；原有合法用例仅覆盖 Start < End，缺少 Start == End 的边界测试。 |
| 第二轮：仅含缺陷时仍然通过的运行 | [PR #3](https://github.com/RyanShen30/ops-lab1-2/pull/3) 的 [ci #8](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106019814)，success；仅含缺陷提交 d1e21a4，未审批、未合并。 |
| 第二轮：补测试后失败的运行 | [ci #9](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106084105)，Test 步骤失败，新增 TestValidSinglePeriod 在 all、odd、even 三种周次下均发现缺陷。 |
| 第二轮：撤销缺陷后通过的运行 | [ci #10](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106121523)，success；通过 git revert d1e21a4 撤销缺陷，保留新增测试。 |
| 第二轮：审批与合入 main 后的运行 | [PR #3 审批](https://github.com/RyanShen30/ops-lab1-2/pull/3#pullrequestreview-5561903762)；2026-10-18 15:14:54 +08:00 squash 合入；[ci #12](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39106946423) 全部成功并产出制品。 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | [release #1](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39197867811)，测试 tag `v0.0.1` 指向未合入 main 的提交，`Release gate` 步骤失败，后续步骤跳过；测试 tag 和分支已清理，未生成 Release。 |
| Release v1.0.0 | [v1.0.0](https://github.com/RyanShen30/ops-lab1-2/releases/tag/v1.0.0)，由 [release #2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440) 成功发布；两个附件已下载，校验和及追溯检查均通过，实际值见第 4 节。 |
| Release v1.0.1 | <链接> |

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

以上完成任务 5 的发布与校验取证，作为任务 6 的旧版本对照基线；任务 6 的 `v1.0.1` 发布、服务运行及回退尚未执行。

## 4. 追溯与运行

`v1.0.0` 的任务 5 发布基线与实际追溯输出如下；`v1.0.1` 及回退验证由任务 6 补齐。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | `1.0.0` | <填写> |
| 构建号 | `2`（`release` 工作流） | <填写> |
| 提交号（SHA） | `5c68dd213a4389cf51500b43be2729ecc880a22e` | <填写> |
| 引入该提交的 PR | [#5](https://github.com/RyanShen30/ops-lab1-2/pull/5) | <编号与链接> |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | [release #2，run_id=39255772440](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440) | <链接> |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `d2936cc91dc6afbaaf90279f6501aedf7edb64770c7c0e64865c7589d8d3c415` | <填写> |

仅凭构建号不能唯一确定运行。`v1.0.0` 的构建号为 `2`：在 `ci` 中对应 [ci #2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/38902466874)，它是 PR 事件，提交号为 `8ed1465a9357c4817964229622a9ecc849fc42dd`；在 `release` 中对应上述 [release #2](https://github.com/RyanShen30/ops-lab1-2/actions/runs/39255772440)，它才构建了 `v1.0.0`。两条工作流各自编号，`BUILD_INFO` 的 `run_id=39255772440` 才能直接确定本次发布运行。

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：<填写>。与上表中发布当时的值是否一致？<填写>

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

`trace.sh` 输出（v1.0.1）：

```text
<粘贴>
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
<运行 v1.0.1 时>
<换回 v1.0.0 后>
```

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

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| #<编号> | <填写> | <填写> | <填写> | <填写> |
| #<编号> | <填写> | <填写> | <填写> | <填写> |
| #<编号> | <填写> | <填写> | <填写> | <填写> |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| #<编号> | <填写> | <填写> | <填写> |

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

<填写>

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

<填写>

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | <填写> | <填写> |
| ② | 与 ① 完全相同，再构建一次 | <填写> | <填写> |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | <填写> | <填写> |
| ④ | 去掉 `--mtime=@0`，其余同 ① | <填写> | <填写> |
| ⑤ | 同 ④，间隔几秒后再构建一次 | <填写> | <填写> |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | <两次的值> | <两次的值> |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

<填写>

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

<填写>

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | <未做 / 链接、镜像摘要（digest）、运行容器的主机架构> |
| O2 缓存对反馈时间的影响 | 已完成。实验 [PR #9](https://github.com/RyanShen30/ops-lab1-2/pull/9) 不合并；预期、全部 12 次运行与结论见下方“选做 O2”。 |
| O3 迁移到小组项目 | 未做 |

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

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| RyanShen30 | [#2](https://github.com/RyanShen30/ops-lab1-2/pull/2) | [#3](https://github.com/RyanShen30/ops-lab1-2/pull/3#pullrequestreview-5561903762) |
| Liuxiang400 | [#3](https://github.com/RyanShen30/ops-lab1-2/pull/3) | [#2](https://github.com/RyanShen30/ops-lab1-2/pull/2#pullrequestreview-5561740627)、[#5](https://github.com/RyanShen30/ops-lab1-2/pull/5#pullrequestreview-5567500720) |
| ashkirakira | [#5](https://github.com/RyanShen30/ops-lab1-2/pull/5) | — |
