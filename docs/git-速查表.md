# Git 命令速查表

> 面向嵌入式开发的 Git 速查。按"我要做 X"组织，不做命令罗列。
> 完整讲解见 [STM32 学习手册 · 第 06 章](STM32学习手册.md#06-git-与版本管理)。

---

## 一、首次配置（只做一次）

```bash
# 身份（必须与 GitHub 账号一致，否则提交不归到你名下）
git config --global user.name  "你的姓名"
git config --global user.email "GitHub 账号邮箱"

# 其他建议项
git config --global init.defaultBranch main   # 默认分支名与 GitHub 一致
git config --global core.autocrlf true        # Windows 换行符处理，跨平台必需
git config --global core.quotepath false      # 中文文件名正常显示

# 自检
git config --global --list
```

### SSH key（推荐，替代 PAT）

> ⚠️ GitHub 的 **classic PAT 自 2026 年 3 月 1 日起被逐步淘汰**。网上大量教程还在教"生成 token 当密码用"，**已过时**。直接用 SSH key。

```bash
ssh-keygen -t ed25519 -C "GitHub 账号邮箱"    # 一路回车
cat ~/.ssh/id_ed25519.pub                    # 复制全部内容
# → GitHub → Settings → SSH and GPG keys → New SSH key → 粘贴 → 保存
ssh -T git@github.com                        # 验证：应输出 "Hi <用户名>!"
```

`id_ed25519` 是**私钥，绝不外传**；只有 `.pub` 结尾的公钥可以粘贴出去。

---

## 二、日常 15 条

| 我想做 | 命令 |
| --- | --- |
| 看当前状态 | `git status` |
| 看具体改了什么 | `git diff` |
| 看已暂存的改动 | `git diff --staged` |
| 加入暂存区 | `git add <文件>` |
| 提交 | `git commit -m "feat(x): 描述"` |
| 推送到 GitHub | `git push` |
| 拉取最新 | `git pull` |
| 看历史（含分支图） | `git log --oneline --graph --all` |
| 看某次提交内容 | `git show <commit-id>` |
| 比较两次提交 | `git diff <c1> <c2>` |
| 丢弃工作区改动 | `git restore <文件>` |
| 从暂存区撤出 | `git restore --staged <文件>` |
| 改最后一次提交信息 | `git commit --amend -m "新信息"` |
| 安全回退（生成反向提交） | `git revert <commit-id>` |
| **误操作救命** | `git reflog` |

### 分支

```bash
git switch -c feature/lvgl-port   # 开分支
git switch main                   # 切回
git merge feature/lvgl-port       # 合并
git branch -d feature/lvgl-port   # 删除已合并的分支
```

> 个人项目用 `main` + `feature/xxx` 两条规则就够。**不要引入 Git Flow**（多分支模型是为多人协作和定期发布设计的，个人项目用它是纯粹负担）。

---

## 三、场景化用法

### 把已有工程第一次推上去

先在 GitHub 网页建**空仓库**（不要勾选 "Add a README"），然后：

```bash
cd /d/STM32/stm32-iot-node
git init
git add .
git status                       # ← 先看！确认没混进 build/ 或大文件
git commit -m "chore: 初始提交"
git remote add origin git@github.com:<用户名>/<仓库名>.git
git push -u origin main
```

### 撤销与回退

```bash
git restore <文件>                # 丢弃工作区修改（不可恢复）
git restore --staged <文件>       # 只从暂存区撤出，保留文件内容
git reset --soft <commit-id>      # 退回提交前，改动仍在暂存区
git reset --hard <commit-id>      # 危险：改动全部丢失
git revert <commit-id>            # 安全：生成一个反向提交，历史保留
```

### 误操作恢复

```bash
git reflog
```

记录**所有 HEAD 的移动**（包括 reset、rebase、checkout）。几乎任何"我把东西搞丢了"都能靠它找回。

```bash
git reset --hard <reflog 里找到的 commit-id>
```

### 忽略已经提交过的文件

`.gitignore` 对**已被跟踪的文件无效**：

```bash
git rm --cached <文件>            # 从索引移除，保留本地文件
# 然后把该文件加进 .gitignore
git commit -m "chore: 停止跟踪 <文件>"
```

### 清理误提交的大文件

```bash
git rm --cached <大文件>
git commit --amend --no-edit      # 若只错在最后一次提交
# 若已在历史深处，需要 git filter-repo 重写历史
```

### 处理冲突

```bash
git pull                          # 冲突会写进文件
# 手动编辑，处理 <<<<<<< ======= >>>>>>> 三标记
git add <冲突文件>
git commit
```

---

## 四、`.gitignore` 模板（嵌入式）

```gitignore
# ===== 构建产物 =====
build/
Build/
cmake-build-*/
*.o
*.obj
*.elf
*.bin
*.hex
*.map
*.lst
*.d
*.su

# ===== CubeMX =====
mx.scratch
Backup/
/Core/Backup/

# ===== Python 上位机 =====
__pycache__/
*.py[cod]
.venv/
.venv-*/
data/*.csv
data/*.log

# ===== 凭据（重要）=====
wifi_credentials.h
secrets.h
.env

# ===== 系统 =====
.DS_Store
Thumbs.db
desktop.ini
```

### 不要忽略这些（常见错误）

| 文件 | 为什么必须提交 |
| --- | --- |
| `*.ioc` | CubeMX 工程源文件。**没有它工程无法复现** |
| `.mxproject` | CubeMX 元数据 |
| `CMakeLists.txt` / `CMakePresets.json` / `cmake/` | 构建定义 |
| `.vscode/settings.json`、`launch.json`、`c_cpp_properties.json` | 让别人 clone 下来就能编译调试 |
| `.clangd` | 代码跳转的编译数据库指向 |
| `.settings/` | IDE 工程设置 |

**两个最常见的错误**：

1. **把 `build/` 提交进去** → 仓库迅速膨胀到几十 MB，每次构建产生海量无意义 diff
2. **把 `.ioc` 排除掉** → 别人 clone 下来只有一堆 C 文件，无法重新配置外设

---

## 五、commit message 规范

格式：`<类型>(<范围>): <描述>`

| 类型 | 用于 |
| --- | --- |
| `feat` | 新功能 |
| `fix` | 修 bug |
| `docs` | 只改文档 |
| `refactor` | 重构，不改变行为 |
| `chore` | 构建、依赖、杂项 |
| `test` | 测试相关 |
| `perf` | 性能优化 |

### 对照示例

| ❌ 坏 | ✅ 好 |
| --- | --- |
| `update` | `feat(i2c): 接入 SHT30 温湿度采集` |
| `改了点东西` | `fix(uart): 修复 DMA 接收丢帧，改用空闲中断 + 环形缓冲` |
| `1111` | `fix(i2c): 修正 7 位地址未左移导致读不到器件` |
| `完成` | `refactor(bsp): 传感器驱动按 BSP 分层抽出，App 层不再直接碰 HAL` |
| `调试` | `perf(adc): 改用定时器触发 + DMA，采样抖动从 ±200µs 降到 ±2µs` |

**为什么值得花这个时间**：`git log` 是面试官翻仓库时第一个看的地方。一个信息清晰的提交历史，能直接证明工程习惯，而且它顺便帮你准备好了"踩过什么坑"这个必问题的答案。

---

## 六、三条最容易忘的规则

1. **`.gitignore` 对已跟踪文件无效** → 先 `git rm --cached <文件>`
2. **`git add .` 之前先 `git status`** → 确认没混进 `build/`、大文件、敏感信息
3. **`git reflog` 几乎能救回一切** → 别慌，先跑它
