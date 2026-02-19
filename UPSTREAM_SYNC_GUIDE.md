# 如何把上游仓库更新合并到你的 Fork（grok2api）

你当前仓库是从 `chenyme/grok2api` Fork 出来的。一个稳妥、可回滚、可持续维护的做法如下。

## 1) 配置上游仓库 remote（只需一次）

```bash
git remote -v
# 如果还没有 upstream，则添加
git remote add upstream https://github.com/chenyme/grok2api.git
# 验证
git remote -v
```

建议保持：
- `origin` = 你的 fork
- `upstream` = 原作者仓库

## 2) 拉取上游最新代码

```bash
git fetch upstream --prune
```

## 3) 从你的主分支切一个“同步分支”

假设主分支是 `main`（如果是 `master` 请替换）：

```bash
git checkout main
git pull origin main
git checkout -b chore/sync-upstream-YYYYMMDD
```

## 4) 合并上游主分支

```bash
git merge upstream/main
# 如果上游是 master：git merge upstream/master
```

如果你希望提交历史更线性，也可以使用 rebase：

```bash
git rebase upstream/main
```

> 建议优先 `merge`，对团队协作和排查问题更直观。

## 5) 解决冲突（重点）

处理冲突时建议遵循以下原则：

1. **模型列表、路由、配置项优先保留上游新增能力**（如新增模型名、参数字段）。
2. **你自己的定制逻辑用“最小补丁”重新叠加**，避免整段覆盖上游文件。
3. 对涉及 `.env.example`、README、API 文档的变更，要同步更新，避免“代码可用但文档落后”。

常用命令：

```bash
git status
# 手动改完冲突文件后
git add .
git commit -m "merge: sync upstream and resolve conflicts"
```

## 6) 本地验证（至少做这些）

```bash
# 依项目而定：
# 1) 依赖安装
# 2) 启动服务
# 3) 调用关键接口
# 4) 验证新增模型可用
```

最低检查清单：
- 服务能正常启动
- 原有模型调用不回归
- 上游新增模型可识别、可转发、可返回
- 错误处理和日志无明显异常

## 7) 推送并通过 PR 合并到你的主分支

```bash
git push origin chore/sync-upstream-YYYYMMDD
```

然后在 GitHub 上从该分支提 PR 到你的 `main`，标题可用：

- `chore: sync upstream chenyme/grok2api`

## 8) 把你的定制做成“可持续维护”结构（让仓库更好）

为了避免每次同步都大冲突，建议：

1. **把你自定义内容集中在独立文件/模块**，尽量不直接改上游核心文件。
2. **用配置驱动差异**（环境变量、配置文件）而不是硬编码。
3. **建立“上游同步节奏”**（例如每周或每两周同步一次），减少一次性大升级痛苦。
4. 在 PR 模板里固定包含：
   - 本次同步到的上游 commit/tag
   - 冲突文件清单
   - 人工验证清单

## 9) 一套可直接执行的完整命令模板

```bash
# 一次性配置
git remote add upstream https://github.com/chenyme/grok2api.git

# 每次同步
git fetch upstream --prune
git checkout main
git pull origin main
git checkout -b chore/sync-upstream-$(date +%Y%m%d)
git merge upstream/main

# 解决冲突后
git add .
git commit -m "merge: sync upstream"
git push origin HEAD
```

---

如果你愿意，我还可以下一步给你一份**针对这个仓库的“冲突高发文件清单 + 合并策略”模板**（按目录和文件类型细分），你以后每次同步直接照表执行即可。
