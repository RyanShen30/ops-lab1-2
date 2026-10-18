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
| 未合入分支上的 tag 被 release gate 拒绝的运行 | <链接> |
| Release v1.0.0 | <链接> |
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
4. 本 PR 同时补齐第一轮审批、合并、main push 运行和制品记录。最终生产逻辑与 main 一致，代码改动仅新增回归测试；实验记录也一并更新。第二轮待另一位组员审批后 squash 合并，最终提交标题使用“test: 补充单节课程合法性边界测试”，正文描述新增测试和记录证据，删除自动列出的缺陷与 revert 等演练提交说明。此处尚未声称第二轮已合入。

ci #9 新增测试的失败输出（服务器本地与 Actions 均发现相同问题）：

```text
--- FAIL: TestValidSinglePeriod (0.00s)
    conflict_test.go:60: single-period section should be valid: {ID: Weekday:1 Start:2 End:2 Weeks:all}
    conflict_test.go:60: single-period section should be valid: {ID: Weekday:1 Start:2 End:2 Weeks:odd}
    conflict_test.go:60: single-period section should be valid: {ID: Weekday:1 Start:2 End:2 Weeks:even}
FAIL
```

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | <填写> | <填写> |
| 构建号 | <填写> | <填写> |
| 提交号（SHA） | <填写> | <填写> |
| 引入该提交的 PR | <编号与链接> | <编号与链接> |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | <链接> | <链接> |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | <填写> | <填写> |

仅凭构建号能否唯一确定是哪次运行？v1.0.0 的构建号在 `ci` 和 `release` 的运行列表中各对应哪次运行？<填写>

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：<填写>。与上表中发布当时的值是否一致？<填写>

`trace.sh` 输出（v1.0.0）：

```text
<粘贴>
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
| O2 缓存对反馈时间的影响 | <未做 / 预期、数据与结论> |
| O3 迁移到小组项目 | <未做 / 仓库链接> |

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| <用户名> | <编号> | <编号> |
