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
| 规则集里启用的规则 | <列出规则名、审批人数和必需检查的名称> |
| 组长直接 `git push origin main` 被拒的终端输出 | <粘贴文本> |
| 第一轮：失败的 PR 运行 | <链接> |
| 第一轮：修复后通过的 PR 运行 | <链接> |
| 第一轮：合入 main 后的运行（含制品） | <链接> |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | <一两句话> |
| 第二轮：仅含缺陷时仍然通过的运行 | <链接> |
| 第二轮：补测试后失败的运行 | <链接> |
| 第二轮：撤销缺陷后通过的运行 | <链接> |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | <链接> |
| Release v1.0.0 | <链接> |
| Release v1.0.1 | <链接> |

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

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | <填写> | <PR 链接> |
| 第一次 `ci` 失败（运行结束） | <填写> | <链接> |
| 推送修复提交 | <填写> | <链接> |
| `ci` 通过（运行结束） | <填写> | <链接> |
| 合入 main | <填写> | <链接> |

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
