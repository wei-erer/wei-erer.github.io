---
layout: post
title: 先导杯实训基础语法笔记：Linux、Shell 与 Python 新手速查
date: 2026-10-01
tags: [先导杯, Linux, Shell, Python, 国产算力, DCU, PyTorch]
categories: [竞赛备赛]
description: 一个新手在 SCNet/DCU 实训平台上被命令行卡住之后，把所有踩过的语法坑整理成的一份可查、可读、带真实输出的基础语法笔记。
---

{% raw %}

> **写在前面**
>
> 这篇笔记的起因很具体：参加「先导杯」智能计算创新设计赛（江南大学校内赛）的实训一，面对一个黑框框终端，老师说「执行 `bash run_check.sh`」，我就照抄；老师说「先导出环境变量」，我也照抄。抄完能跑，但完全不知道为什么，换个路径就报错，换个参数就慌乱。
>
> 于是我把自己所有「只会复制、不理解」的地方整理成了这份笔记。它不是 Linux 手册，而是**一个新手在国产算力实训平台上真正用得上的那部分语法**，从「终端到底是什么」讲到「Python 脚本的参数从哪来」。
>
> 全部示例都来自实训平台上的真实场景（SCNet + DCU + PyTorch-DAS + DIBaS 细菌图像分类）。如果你也在做类似的实训或课程，这篇可以直接当速查手册用。

---

## 目录

- [第 0 章 先搞清楚：你面对的这个黑框框是什么](#第-0-章-先搞清楚你面对的这个黑框框是什么)
- [第 1 章 文件与目录：先学会「我在哪、有什么」](#第-1-章-文件与目录先学会我在哪有什么)
- [第 2 章 Shell 语法：命令行的「语法规则」](#第-2-章-shell-语法命令行的语法规则)
- [第 3 章 环境变量与 export：实训里最容易翻车的一节](#第-3-章-环境变量与-export实训里最容易翻车的一节)
- [第 4 章 管道、重定向与筛选三件套](#第-4-章-管道重定向与筛选三件套)
- [第 5 章 bash 脚本：run_train.sh 里到底写了什么](#第-5-章-bash-脚本run_trainsh-里到底写了什么)
- [第 6 章 实训的标准运行流程（逐条解释）](#第-6-章-实训的标准运行流程逐条解释)
- [第 7 章 Python 基础：脚本、参数与 import](#第-7-章-python-基础脚本参数与-import)
- [第 8 章 报错信息怎么读：从一头雾水到知道去哪查](#第-8-章-报错信息怎么读从一头雾水到知道去哪查)
- [第 9 章 自测清单：这 20 条能默写，实训就不慌了](#第-9-章-自测清单这-20-条能默写实训就不慌了)
- [附录 A 常用命令速查表](#附录-a-常用命令速查表)
- [附录 B 术语中英对照](#附录-b-术语中英对照)

---

## 第 0 章 先搞清楚：你面对的这个黑框框是什么

### 0.1 它不是 Windows 的 CMD，也不是 PowerShell

实训平台的终端是一个 **Linux 云服务器上的 Web 终端**：

![实训平台界面：左侧是实训文档，右侧是 DCU 实例的 E-Shell 命令行](平台界面.png)

图里右侧那个 `ghdevadmin@worker-0:~/learning/Instances_Course_...` 就是终端提示符。拆开看：

```
ghdevadmin @ worker-0 : ~/learning/Instances_Course_...$
└────┬───┘   └──┬───┘   └──────────┬─────────────┘
   用户名      主机名          当前所在目录
```

`~` 是「当前用户的家目录」的简写。最末尾那个 `$` 表示「我是普通用户，等你输入」（如果是 `#`，说明你成了 root，权限很大，操作要更小心）。

**关键差别**（这是新手最容易踩的坑）：

| | Windows CMD / PowerShell | Linux Bash（实训平台） |
|---|---|---|
| 路径分隔符 | `\` | `/` |
| 大小写 | 不敏感 | **敏感**，`Data` ≠ `data` |
| 盘符 | `C:\ D:\` | 没有盘符，一切从 `/` 开始 |
| 命令后面加参数 | `/h` 或 `-h` | 一般是 `-h` 或 `--help` |
| 复制粘贴 | Ctrl+C / Ctrl+V | 复制 `Ctrl+Shift+C`，粘贴 `Ctrl+Shift+V`（或右键） |

> ⚠️ **实训平台特有警告**：Web 终端是「远程会话」。**关掉网页、或者新开一个终端窗口，之前 `export` 设置的环境变量会失效**，有些临时文件也可能找不到。做完实验，第一件事是把结果文件下载到本地。

### 0.2 一条命令的解剖结构

```
bash    run_train.sh    cuda
└─┬─┘   └─────┬─────┘   └─┬─┘
命令名     参数1        参数2
```

再复杂一点：

```
python   -u   scripts/train_resnet18.py   --epochs   30
└─┬─┘   └┬┘   └──────────┬──────────┘   └───┬───┘  └┬┘
命令名  选项      脚本路径（也是参数）      长选项名   选项的值
```

三个必须建立的直觉：

1. **空格是分隔符**。`bash run_train.sh` 是两个词；`bashrun_train.sh` 是一个不存在的命令。多敲、少敲一个空格，结果完全不同。
2. **命令名和参数之间必须用空格**。这一点和自然语言不同，也和 Windows 的「命令 /参数」不同。
3. **短选项和长选项**：`-u` 是短选项（一个字母），`--epochs` 是长选项。大多数命令都支持 `--help`：

```bash
python --help
bash --help
ls --help
```

**`--help` 是你的第一救援手段**，比问人快。

### 0.3 提示符里的 `$` 是「提示」，不是「要你输入」

很多教程写：

```bash
$ ls
file1.txt  file2.txt
```

这里的 `$` 是提示符，**不要复制**。你只需要敲 `ls`。所以看文档时，遇到行首的 `$` 或 `#` 要自动忽略。

---

## 第 1 章 文件与目录：先学会「我在哪、有什么」

我实训时第一个恐慌瞬间是：「文件到底跑哪儿去了？」解决这个问题只需要三条命令。

### 1.1 `pwd`：我在哪

```bash
pwd
```

输出示例：

```
/public/SothisAI/learning_center
```

`pwd` = **p**rint **w**orking **d**irectory。永远先 `pwd`，再执行任何「找不到文件」的操作。

### 1.2 `ls`：这层有什么

```bash
ls            # 只列名字
ls -l         # 详细列表（权限、大小、时间）
ls -lh        # 详细列表 + 人类可读的大小（K/M/G）
ls -a         # 连隐藏文件（以 . 开头的）一起列
ls -lht       # 详细 + 大小可读 + 按时间倒序（找刚生成的文件神器）
```

输出示例：

```bash
$ ls -lh
total 24K
drwxr-xr-x 3 ghdevadmin ghdevadmin 4.0K Oct  1 10:12 scripts
drwxr-xr-x 4 ghdevadmin ghdevadmin 4.0K Oct  1 10:10 data
drwxr-xr-x 2 ghdevadmin ghdevadmin 4.0K Oct  1 10:12 checkpoints
-rwxr-xr-x 1 ghdevadmin ghdevadmin  412 Oct  1 10:12 run_check.sh
-rw-r--r-- 1 ghdevadmin ghdevadmin  598 Oct  1 10:12 run_train.sh
```

第一列读法（后面 3.5 节详细讲）：

```
d rwx r-x r-x
│  │   │   └── 其他人：读+执行
│  │   └────── 同组人：读+执行
│  └────────── 自己：读+写+执行
└───────────── d=目录，-=普通文件
```

### 1.3 `cd`：换目录

```bash
cd scripts               # 进入 scripts 目录（相对路径）
cd /public/SothisAI/learning_center   # 进入绝对路径
cd ..                    # 上一级
cd ../..                 # 上两级
cd ~                     # 回到家目录
cd -                     # 回到上一次待过的目录（来回切换很方便）
cd                       # 什么都不跟，也是回家目录
```

**绝对路径 vs 相对路径**——这是新手第二大坑：

| | 写法 | 含义 |
|---|---|---|
| 绝对路径 | `/public/SothisAI/learning_center` | 以 `/` 开头，从根目录算起，**在哪执行都一样** |
| 相对路径 | `scripts/train_resnet18.py` | 不带 `/` 开头，**相对当前目录**算起 |

> 💡 **实训踩坑实例**：文档里写 `python -u scripts/train_resnet18.py`。如果你当前 `pwd` 是 `/public/SothisAI/learning_center/scripts`，那么这条命令会去找 `/public/SothisAI/learning_center/scripts/scripts/train_resnet18.py`，报错：
>
> ```
> python: can't open file 'scripts/train_resnet18.py': [Errno 2] No such file or directory
> ```
>
> **看到 `No such file or directory`，第一反应就是 `pwd` + `ls`。**

### 1.4 建、删、移、看

```bash
mkdir my_results                 # 建目录
mkdir -p a/b/c                   # 建多层目录（-p = parents，已存在也不报错）
touch test.txt                   # 建空文件（或更新时间戳）

cp a.txt b.txt                   # 复制文件
cp -r figures figures_bak        # 复制整个目录（-r = recursive）
mv old.txt new.txt               # 改名 / 移动
rm test.txt                      # 删除文件
rm -r my_results                 # 删除目录及其内容
```

> ☠️ **危险警告**：Linux 的 `rm` **没有回收站**，删了就没了。永！远！不要在不确定路径的情况下执行：
>
> ```bash
> rm -rf /            # 这是灾难，绝对不要碰
> rm -rf $WPATH/*     # 如果 $WPATH 是空的……你自己想想
> ```
>
> 演练建议：**先 `echo` 出你要删的路径，确认无误，再删。**

```bash
echo "即将删除：$WPATH/old_results"    # 先打印确认
rm -rf "$WPATH/old_results"            # 确认后再删
```

### 1.5 看文件内容

```bash
cat results/inference/inference_results.csv    # 一次性全打印（短文件用）
head -n 20 results/logs/train_resnet18_latest.log   # 看前 20 行
tail -n 50 results/logs/train_resnet18_latest.log   # 看后 50 行
tail -f results/logs/train_resnet18_latest.log      # 实时跟踪（训练时开另一个窗口看日志）
less results/logs/train_resnet18_latest.log         # 分页浏览（q 退出，/ 搜索）
```

`tail -f` 在实训里特别有用：训练要跑 13 分钟，你可以开一个窗口 `tail -f` 看日志实时滚。

### 1.6 找文件

```bash
find . -name "inference_results.csv"            # 在当前目录下按名字找
find . -name "*.pth"                            # 找所有模型权重
find . -name "*.log" -newer run_check.sh        # 找比某文件更新的日志
find . -type d -name "checkpoints"              # 只找目录（-type d）
```

```bash
$ find . -name "resnet18_best_*.pth"
./checkpoints/resnet18_best_dcu_hip_33class.pth
```

`find` 会递归往下找，所以「文件去哪了」这个问题，`find` 基本都能回答。

---

## 第 2 章 Shell 语法：命令行的「语法规则」

### 2.1 Shell 和终端不是一回事

- **终端（Terminal）**：你看到的那个窗口，负责显示和输入。
- **Shell**：窗口里真正解析你输入的程序。实训平台上默认是 **bash**（Bourne Again SHell）。

你敲 `ls -lh`，不是系统「认识」这句话，而是 bash 把这一行拆成「命令名 `ls` + 参数 `-lh`」，找到 `ls` 这个可执行文件去执行。

```bash
echo $BASH_VERSION     # 看当前 bash 版本
echo $SHELL            # 看当前默认 shell
```

### 2.2 注释：`#`

```bash
# 这一整行都是注释，bash 不会执行
export WPATH="$PWD"    # 行内也可以跟在命令后面
```

实训脚本 `run_train.sh` 里大量用注释说明用途，读脚本时**先读注释**。

### 2.3 变量：赋值、引用、花括号

```bash
# 赋值：等号两边不能有空格！
NAME="resnet18"
EPOCHS=30

# 引用：用 $ 取值
echo "$NAME"           # 输出 resnet18
echo "$EPOCHS"         # 输出 30
```

**错误示范**（新手 100% 会犯）：

```bash
NAME = "resnet18"
# bash: NAME: command not found
```

原因：bash 把 `NAME` 当成了命令名。**赋值时等号两边绝对不能有空格。**

**花括号 `${}` 的作用**——限定变量名的边界：

```bash
NAME="resnet18"
echo "$NAME_best.pth"     # ✗ bash 会去找变量 NAME_best，结果为空 -> "_best.pth"
echo "${NAME}_best.pth"   # ✓ 输出 resnet18_best.pth
```

这个坑在实训里很常见，因为权重文件名就是 `resnet18_best_dcu_hip_33class.pth` 这种带下划线的形式。

**命令替换 `$()`**：把一个命令的输出当作值。

```bash
WPATH="$(pwd)"             # 等价于 export WPATH=$PWD
TODAY="$(date +%Y%m%d)"
COUNT="$(ls *.pth | wc -l)"
echo "今天 $TODAY，有 $COUNT 个权重文件"
```

老写法是反引号 `` `pwd` ``，效果一样，但 `$()` 更清晰、还能嵌套：

```bash
echo "$(basename "$(pwd)")"
```

### 2.4 引号：单引号、双引号、不加引号

这是 Shell 里最反直觉、也最容易出错的部分。

| 写法 | 变量的处理 | 通配符的处理 | 空格的处理 |
|---|---|---|---|
| `"双引号"` | **会展开** `$VAR` | 不展开 `*` | 整块当一个参数 |
| `'单引号'` | **原样输出** | 不展开 `*` | 整块当一个参数 |
| 不加引号 | 会展开 | **会展开** | **按空格拆成多个参数** |

```bash
V="hello"
echo "双引号会展开：$V"        # 双引号会展开：hello
echo '单引号原样输出：$V'      # 单引号原样输出：$V
echo 反斜杠转义：\$V           # 反斜杠转义：$V
```

**为什么路径几乎总要加引号？**因为路径里可能有空格：

```bash
DIR="my results"
ls $DIR          # ✗ 变成 ls my results —— 找两个东西
ls "$DIR"        # ✓ 找 "my results" 这一个目录
```

所以请养成习惯：**变量引用一律加双引号**，写成 `"$VAR"`。

### 2.5 参数与特殊变量

写脚本或函数时，这些符号直接可用：

| 符号 | 含义 |
|---|---|
| `$0` | 脚本自己的名字（或函数名） |
| `$1` `$2` `$3` | 第 1、2、3 个参数 |
| `$#` | 参数个数 |
| `$@` | 全部参数（推荐，`"$@"` 会保留每个参数的边界） |
| `$*` | 全部参数（合并成一串） |
| `$?` | **上一条命令的退出码**（0 = 成功，非 0 = 失败） |
| `$$` | 当前进程号 |
| `${1:-默认值}` | 第 1 个参数为空时用默认值 |

这就是 `bash run_train.sh cuda` 里 `cuda` 的去处——脚本内部的 `$1` 拿到了它。

### 2.6 退出码 `$?`：判断成功与否的唯一标准

```bash
bash run_check.sh
echo $?          # 0 表示成功
```

实训文档里每个任务后面都跟着 `EXIT_CODE: 0`，这就是退出码。**看到非 0，说明那一步失败了，后面别往下跑。**

```bash
python scripts/bench_resnet18.py
echo $?          # 0 成功；1 一般异常；2 参数错误；127 命令不存在；130 被 Ctrl+C 中断
```

### 2.7 `&&`、`||`、`;`：命令之间的逻辑

```bash
mkdir -p results && cd results        # 前面成功才执行后面（&& 是「与」）
cd nonexist || echo "目录不存在"       # 前面失败才执行后面（|| 是「或」）
echo A; echo B                        # 分号：不管前面成不成，都继续
```

实训脚本里常见这种稳健写法：

```bash
cd /public/SothisAI/learning_center || exit 1
```

意思是：切不过去就直接退出（退出码 1），不要让后面的命令在错误目录里瞎跑。

### 2.8 命令替换再进阶：把多行输出当值

```bash
CKPT="$(ls checkpoints/resnet18_best_*.pth | head -n 1)"
echo "找到权重：$CKPT"
```

### 2.9 通配符（Glob）

| 符号 | 匹配 |
|---|---|
| `*` | 任意长度任意字符 |
| `?` | 任意**一个**字符 |
| `[abc]` | a、b、c 中任意一个 |
| `{a,b}` | 大括号展开，变成两个词 |

```bash
ls checkpoints/*.pth
ls checkpoints/resnet18_best_*.pth
ls figures/training/*.png
cp data/splits/split_indices_*.csv /tmp/
```

**重要区别**：通配符是 **shell 展开**的，不是命令自己处理的：

```bash
ls *.pth        # shell 先展开成 ls a.pth b.pth，再执行
ls "*.pth"      # 引号阻止展开，ls 收到字面量 "*.pth"，报 No such file
```

### 2.10 Tab 补全与历史：省一半打字量

| 操作 | 效果 |
|---|---|
| `Tab` | **自动补全**。`cd scr` + Tab → `cd scripts/` |
| `Tab` `Tab` | 列出所有候选 |
| `↑` / `↓` | 翻历史命令 |
| `Ctrl + R` | 反向搜索历史（输入关键词，回车执行） |
| `Ctrl + C` | 强制终止当前正在跑的命令 |
| `Ctrl + L` | 清屏（等于 `clear`） |
| `Ctrl + A` / `Ctrl + E` | 光标跳到行首 / 行尾 |
| `Ctrl + U` / `Ctrl + K` | 删掉光标前 / 后的内容 |
| `Ctrl + W` | 删掉光标前一个词 |

> **强烈建议把 Tab 补全练成肌肉记忆**：路径又长又容易敲错（`DIBaS_33class_official660` 这种），全靠 Tab 才不会打错大小写。打错了只会得到 `No such file or directory`。

### 2.11 权限位与 `chmod`

`ls -l` 第一列的 10 个字符：

```
- rwx r-x r-x
│  │   │   └── 其他用户 (other)
│  │   └────── 同组用户 (group)
│  └────────── 文件属主 (user)
└───────────── 类型：- 文件, d 目录, l 软链接
```

`r` = 读（4）、`w` = 写（2）、`x` = 执行（1）。

```bash
chmod +x run_check.sh        # 给脚本加「可执行」权限
chmod 755 run_check.sh       # owner=7(rwx) group=5(r-x) other=5(r-x)
ls -l run_check.sh
# -rwxr-xr-x 1 ghdevadmin ghdevadmin 412 Oct 1 10:12 run_check.sh
```

**关键区别**（很多新手不知道 `bash xxx.sh` 为什么能跑）：

```bash
./run_check.sh        # 直接执行：需要这个文件有 x 权限
bash run_check.sh     # 用 bash 去「读」这个文件并解释执行：不需要 x 权限，只需要 r
```

所以实训文档统一写 `bash run_check.sh`——**这样不依赖文件权限，更稳**。如果报 `Permission denied`，要么改用 `bash xxx.sh`，要么 `chmod +x xxx.sh`。

---

## 第 3 章 环境变量与 `export`：实训里最容易翻车的一节

实训第一步永远是这几行，我一开始完全不懂为什么要背下来：

```bash
export WPATH="$PWD"
export HOME="$WPATH/.home"
export XDG_CONFIG_HOME="$HOME/.config"
export XDG_CACHE_HOME="$WPATH/.cache"
export MIOPEN_USER_DB_PATH="$XDG_CONFIG_HOME/miopen"
cd /public/SothisAI/learning_center
```

现在逐行解释。

### 3.1 环境变量是什么

**环境变量 = 贴在整个会话上的便签**。任何程序启动时都能读到这些便签，从而知道「该把文件写到哪」「去哪找配置」。

```bash
env | head -n 20          # 列出当前所有环境变量
echo "$HOME"              # 看某一个
echo "$PATH"              # PATH 是最重要的之一：命令的搜索路径
```

`PATH` 是个用冒号隔开的目录列表。你敲 `python`，bash 就是沿着 `PATH` 一个个目录找有没有叫 `python` 的可执行文件。找到了就执行，全找完还没有，就报：

```
bash: python: command not found
```

### 3.2 不带 `export` 的变量，只是「局部便签」

```bash
MYVAR="只在当前 shell"      # 局部变量
export MYVAR2="会传给子进程"  # 环境变量
```

验证一下（这是本节的实验，建议自己敲一遍）：

```bash
A="local"
export B="exported"
bash -c 'echo "A=$A"'
bash -c 'echo "B=$B"'
```

输出：

```
A=          <- 子进程看不到 A
B=exported  <- 子进程看得到 B
```

**为什么实训要用 `export`？** 因为你敲的 `bash run_train.sh` 会**启动一个新的 bash 子进程**。只有 `export` 过的变量，才会被这个子进程继承。

### 3.3 为什么必须把 `HOME` 改掉

实训文档里这句话是官方解释：

> 以上设置用于将运行过程中产生的用户配置、缓存及 MIOpen 相关文件写入当前课程实例的可写目录，避免因系统用户目录或公共课程目录只读导致权限错误。

翻译成人话：

- 平台上的 `/public/SothisAI/learning_center` 是**公共课程目录，只读**，谁都不能往里写。
- 默认的 `HOME`（比如 `/root` 或 `/home/xxx`）在容器里可能也**不可写**。
- PyTorch / MIOpen（DCU 的算子库）运行时**必须往用户目录写缓存**（MIOpen 会做算子自动调优，把结果存到数据库文件里）。
- 一旦写不进去，就是各种 `Permission denied`、`Read-only file system`。

于是把 `HOME`、`XDG_CONFIG_HOME`、`XDG_CACHE_HOME`、`MIOPEN_USER_DB_PATH` 全部指到你自己的可写目录（`$WPATH` 下面），问题就消失了。

```bash
export WPATH="$PWD"                     # 把当前目录记下来当「我的工作区根」
export HOME="$WPATH/.home"              # 家目录也指到我的工作区
export XDG_CONFIG_HOME="$HOME/.config"  # 配置目录
export XDG_CACHE_HOME="$WPATH/.cache"   # 缓存目录
export MIOPEN_USER_DB_PATH="$XDG_CONFIG_HOME/miopen"  # MIOpen 算子缓存
```

注意 `export HOME="$WPATH/.home"` 和 `export XDG_CONFIG_HOME="$HOME/.config"` 的**顺序不能换**——后面引用了前面刚设好的值。这就是「逐层派生」的写法。

> ⚠️ **再次强调**：这些 `export` 只对**当前这个终端会话**有效。**关掉网页 / 新开窗口 / 重连实例之后，必须重新执行一遍。**
>
> 如果你发现「刚刚还在的文件找不到了」或者「又报权限错误了」，八成就是这个原因。这也是我实训时最崩溃的一次。

### 3.4 环境变量常用操作

```bash
echo "$WPATH"                        # 检查有没有设上
env | grep MIOPEN                    # 筛选相关变量
export OMP_NUM_THREADS=8             # 临时限制线程数
unset WPATH                          # 删掉一个变量
export CUDA_VISIBLE_DEVICES=0        # 只用第 0 号加速卡（多卡时用）
VAR=value command                    # 只对这一条命令生效，不改当前会话
```

### 3.5 配置持久化：不想每次重敲怎么办

两种办法：

```bash
# 办法 1：把 export 写进 ~/.bashrc（每次开新 shell 自动执行）
cat >> "$HOME/.bashrc" <<'EOF'
export WPATH="$PWD"
export HOME="$WPATH/.home"
export XDG_CONFIG_HOME="$HOME/.config"
export XDG_CACHE_HOME="$WPATH/.cache"
export MIOPEN_USER_DB_PATH="$XDG_CONFIG_HOME/miopen"
EOF

# 办法 2：写成一个 env.sh，每次手动 source
source ./env.sh      # 或 . ./env.sh
```

这里用到了 **here-document（`<<'EOF'`）**：把两个 `EOF` 之间的内容当作输入喂给命令。加引号的 `'EOF'` 表示**内部不做变量展开**，原样写入——本意就是要把 `$PWD` 这行字写进文件，而不是现在就展开。

顺带复习 **`source` 与 `bash` 的区别**（很重要）：

```bash
bash env.sh      # 开子进程执行 -> 里面的 export 出了子进程就没了
source env.sh    # 在当前 shell 里执行 -> export 生效在当前会话
```

**所以「配置类脚本」要用 `source`，「任务类脚本」用 `bash`。**

### 3.6 DCU 环境自检（顺手记下来）

```bash
python -c "import torch; print(torch.__version__)"
python -c "import torch; print(torch.cuda.is_available())"
python -c "import torch; print(torch.cuda.get_device_name(0))"
python -c "import torch; print('hip:', torch.version.hip, 'cuda:', torch.version.cuda)"
```

这里的 `-c` 就是「把后面这串字符串当成一段 Python 代码执行」。在 DCU 上，`device` 依然写 `"cuda"`（PyTorch-DAS 复用了 CUDA 的设备语义），但设备名会显示 **BW**，`torch.version.hip` 有值而 `torch.version.cuda` 为空——这是区分 DCU 和 NVIDIA GPU 的最快方法。

---

## 第 4 章 管道、重定向与筛选三件套

### 4.1 三个标准流

每个 Linux 程序天生带三个通道：

| 编号 | 名字 | 用途 |
|---|---|---|
| 0 | stdin | 标准输入（你敲的、别的程序喂的） |
| 1 | stdout | 标准输出（正常结果） |
| 2 | stderr | 标准错误（报错信息） |

**关键认知：正常输出和报错是两条不同的通道。** 这解释了为什么有的报错文件里没有、屏幕上却有。

### 4.2 重定向

```bash
python train.py > train.log          # stdout 覆盖写入文件（屏幕上看不到了）
python train.py >> train.log         # stdout 追加写入文件
python train.py 2> err.log           # 只把 stderr 写入文件
python train.py > all.log 2>&1       # stdout 和 stderr 都写进去（顺序很重要）
python train.py > /dev/null 2>&1     # 全部丢弃（/dev/null 是「黑洞」）
command < input.txt                  # 把文件当 stdin
```

`2>&1` 的读法是：**「把 2 号（stderr）指向 1 号（stdout）现在所在的地方」**。因为重定向是从左往右生效的，所以：

```bash
> all.log 2>&1      # ✓ 先让 stdout 进文件，再让 stderr 跟着 stdout 进文件
2>&1 > all.log      # ✗ 先让 stderr 指向屏幕（当时的 stdout），再把 stdout 进文件
```

这也是实训文档里 `find . -name "*.pth" 2>/dev/null` 的含义：**「权限不足」这类噪音我不想看，丢掉。**

```bash
find / -name "run_train.sh" 2>/dev/null      # 只看结果，不看满屏的 Permission denied
```

### 4.3 管道 `|`：把上一个的输出当下一个的输入

```bash
ls results/logs/ | grep train
cat train.log | grep "Epoch" | tail -n 5
```

管道是 Linux 最优雅的设计：每个命令只干一件小事，用 `|` 串起来完成复杂任务。

```bash
# 实训实战：从日志里捞出最后 3 个 epoch 的记录
grep "Epoch" results/logs/train_resnet18_latest.log | tail -n 3
```

### 4.4 筛选三件套：`grep`、`head/tail`、`wc`

```bash
grep "Error" results/logs/train_resnet18_latest.log        # 找含 Error 的行
grep -i "error" train.log                                   # 忽略大小写
grep -n "Epoch" train.log                                   # 显示行号
grep -c "Epoch" train.log                                   # 只输出匹配行数
grep -r "num_classes" scripts/                              # 递归搜索整个目录
grep -v "warmup" bench.log                                  # 反选：不包含 warmup 的行
```

```bash
head -n 20 file.log      # 前 20 行
tail -n 20 file.log      # 后 20 行
tail -f file.log         # 实时跟随
wc -l file.log           # 数行数
wc -c file.log           # 数字节
```

组合起来的经典排查套路：

```bash
# 1. 训练卡住了？看日志最后几行
tail -n 30 results/logs/train_resnet18_latest.log

# 2. 报错了？把错误上下文抓出来
grep -n -A 5 -B 2 "Traceback" results/logs/train_resnet18_latest.log
#        └─┬─┘ └───┬───┘
#      显示行号  匹配行后5行/前2行

# 3. 结果文件生成没？
ls -lht results/inference/ | head
```

### 4.5 `sed`、`awk`、`cut`、`sort`、`uniq`：处理文本的瑞士军刀

```bash
sed -n '1,10p' file.log                    # 打印第 1~10 行
sed 's/dcu/gpu/g' file.log                 # 把 dcu 替换成 gpu（只影响输出）
sed -i 's/lr=0.0001/lr=0.001/' run_train.sh   # -i 直接改文件（谨慎！先备份）
awk '{print $2}' bench.txt                 # 打印第 2 列（默认按空白切分）
awk -F: '{print $1}' file.txt              # 指定分隔符为冒号
cut -d',' -f1,3 results.csv                # 按逗号取第 1、3 列
sort -n numbers.txt                        # 按数值排序
sort file.txt | uniq -c                    # 去重并统计出现次数
```

**实训实战**：从 benchmark 输出里把吞吐量抽出来。

```bash
grep "img/s" bench.log | awk '{print $NF}'      # $NF = 最后一个字段
```

### 4.6 `du` / `df`：磁盘相关（结果文件太多时会用上）

```bash
du -sh figures/                # 这个目录占多大
du -sh */ | sort -h            # 各子目录占用排序
df -h                          # 整个磁盘还剩多少
```

---

## 第 5 章 bash 脚本：run_train.sh 里到底写了什么

### 5.1 脚本就是「把一串命令存进文件」

先看我自编的简化版（只有骨架，方便理解结构）：

```bash
#!/usr/bin/env bash
# run_train.sh —— 简化示意，展示常见结构

set -e                      # 任何命令失败就立即退出，最实用的安全开关

DEVICE="${1:-cuda}"          # 第 1 个参数，缺省用 cuda
DATA_DIR="data/processed/DIBaS_33class_official660"
EPOCHS=30
BATCH=16
LR=1e-4

echo "COMMAND: python -u scripts/train_resnet18.py --device $DEVICE ..."

python -u scripts/train_resnet18.py \
    --device "$DEVICE" \
    --data_dir "$DATA_DIR" \
    --epochs "$EPOCHS" \
    --batch_size "$BATCH" \
    --lr "$LR"

echo "EXIT_CODE: $?"
```

再对照**课件里 `run_train.sh` 的真实写法**（实训一课件节选，逐字抄录）：

```bash
DEVICE=${1:-cuda}
PYTHON=${PYTHON:-python}
mkdir -p results/logs
LOG_FILE="results/logs/train_resnet18_$(date +%Y%m%d_%H%M%S).log"

$PYTHON -u scripts/train_resnet18.py \
  --device "$DEVICE" \
  --data_dir data/processed/DIBaS_33class_official660 \
  --epochs 30 --batch_size 16 --lr 1e-4 2>&1 | tee -a "$LOG_FILE"

elapsed=$((end_ts - start_ts))
echo "ELAPSED_SECONDS: $elapsed"
```

**这一小段里塞进了本笔记第 2~4 章几乎所有的语法点**，逐条对上号：

| 写法 | 语法点 | 在哪一节 |
|---|---|---|
| `${1:-cuda}` | 位置参数 + 默认值展开 | 2.5 |
| `${PYTHON:-python}` | 「允许外部覆盖」的惯用法 | 2.5 |
| `mkdir -p results/logs` | 建多层目录 | 1.4 |
| `$(date +%Y%m%d_%H%M%S)` | 命令替换，给日志文件名加时间戳 | 2.3 |
| `"$DEVICE"` | 变量加双引号 | 2.4 |
| 行尾 `\` | 续行 | 5.2 ④ |
| `2>&1 \| tee -a` | 重定向 + 管道 + 追加写日志 | 4.2 / 8.4 |
| `$((end_ts - start_ts))` | 算术展开 | 5.2 ⑥ |
| `LOG_FILE=...` | 赋值等号两边**不能有空格** | 2.3 |

> 💡 **读懂这段，你就读懂了所有 `run_*.sh`。** 后面实训二到实训五的脚本，无非是把 `train_resnet18.py` 换成 `train_yolov8.py`、`train_unet.py`，把参数换一换，结构完全一样。
>
> 也正因如此，`LOG_FILE` 里的 `$(date +%Y%m%d_%H%M%S)` 意味着**每次运行都会生成一个新日志**（形如 `train_resnet18_20261001_101230.log`），不会互相覆盖——这是做「实验记录」的正确姿势。

### 5.2 逐行拆解

**① `#!/usr/bin/env bash` —— shebang（释伴行）**

放在第一行，告诉系统「这个文件用 bash 解释执行」。有了它 + `chmod +x`，才能 `./run_train.sh`。没有它也能用 `bash run_train.sh` 跑。

**② `set -e` —— 出错就停**

```bash
set -e
```

默认情况下，bash 脚本里某条命令失败了**还会继续往下跑**，这可能让你在错误状态下一路做下去。`set -e` 让任何一条命令返回非 0 就立即退出。

配套的还有：

```bash
set -u      # 引用未定义变量时报错（防手滑打错变量名）
set -x      # 把每条执行的命令打印出来（调试神技）
set -o pipefail   # 管道中任一环失败，整个管道就算失败
```

调试脚本时的黄金组合：

```bash
bash -x run_train.sh cuda        # 打印每一步实际执行的命令
```

**③ `${1:-cuda}` —— 参数默认值**

```bash
DEVICE="${1:-cuda}"
```

含义：如果调用者给了第 1 个参数就用它，没给就用 `cuda`。所以：

```bash
bash run_train.sh          # DEVICE=cuda
bash run_train.sh cuda     # DEVICE=cuda
bash run_train.sh cpu      # DEVICE=cpu
```

**这就是 `bash run_train.sh cuda` 里那个 `cuda` 的意义**——它被脚本内部的 `$1` 接住，最终传给 Python 脚本的 `--device`。

**④ 反斜杠 `\` 续行**

```bash
python -u scripts/train_resnet18.py \
    --device cuda
```

行尾的 `\` 表示「这一行还没完，下一行接着」。纯粹为了可读性。注意 `\` 后面**不能有空格**，否则会报错。

**⑤ 打印 + 执行 + 打印退出码：官方脚本的固定套路**

实训文档里每个任务的输出都是这个形式：

```
COMMAND: python -u scripts/train_resnet18.py --device cuda --data_dir ... 
...（脚本输出）...
EXIT_CODE: 0
ELAPSED_SECONDS: 811
ELAPSED_HHMMSS: 00:13:31
```

也就是 `run_*.sh` 帮你做了三件事：**打印即将执行的命令 → 执行 → 报告耗时和退出码**。这也是为什么文档说「以实训课程运行说明文档中的最新命令为准」——你其实不需要背 Python 参数，只要会看脚本。

顺带注意课件里那句 `echo "COMMAND: ..."` 的价值：**它把「我到底跑了什么」写进了日志**。做实验记录时，最怕的就是「跑出了结果但忘了当时用的什么参数」。

**⑥ `2>&1 | tee -a`：一边看屏幕一边存日志**

```bash
$PYTHON -u scripts/train_resnet18.py \
  --device "$DEVICE" \
  ... 2>&1 | tee -a "$LOG_FILE"
```

拆开念：

```
python -u ... 2>&1   →  把「正常输出 + 报错」合并成一路
              |      →  用管道喂给下一条命令
        tee -a "$LOG_FILE"  →  一份打印到屏幕，一份追加写入日志文件
```

> 💡 为什么官方脚本要这么写？因为**报错默认走 stderr，直接 `> log` 会漏掉报错**。加了 `2>&1`，日志里才会留下完整的现场；加 `tee`，你又能实时看到进度。这是我自己踩过坑之后才真正记住的一条——第 4.2 节讲过原理，这里是它的实战形态。

**⑦ 计时**

```bash
START=$(date +%s)
# ...跑任务...
END=$(date +%s)
echo "ELAPSED_SECONDS: $((END - START))"
```

`$(( ))` 是算术运算（课件里写作 `elapsed=$((end_ts - start_ts))`）。`date +%s` 取 Unix 时间戳（秒）。

把秒数变成 `00:13:31` 这种格式也常见：

```bash
printf "ELAPSED_HHMMSS: %02d:%02d:%02d\n" $((elapsed/3600)) $((elapsed%3600/60)) $((elapsed%60))
```

### 5.3 条件判断与循环（读脚本必备）

```bash
# if：判断文件/目录是否存在
if [ -f "$CKPT" ]; then
    echo "找到权重文件：$CKPT"
else
    echo "权重文件不存在，请先训练"
    exit 1
fi

# if：判断命令是否存在
if ! command -v python >/dev/null 2>&1; then
    echo "python 未安装"
    exit 1
fi

# 常见测试条件
[ -f file ]     # 是普通文件
[ -d dir ]      # 是目录
[ -z "$V" ]     # 字符串为空
[ -n "$V" ]     # 字符串非空
[ "$A" = "$B" ] # 字符串相等
[ "$A" -eq "$B" ]  # 整数相等（-ne -gt -lt -ge -le）
```

> ⚠️ **`[` 的坑**：`[` 其实是个命令，所以**中括号内侧必须有空格**。`[ -f "$F" ]` 对，`[-f "$F"]` 会报 `command not found`。变量一定要加引号，否则空值时 `[ -z ]` 会变成语法错误。

```bash
# for：批量跑不同 batch size
for BS in 1 8 16 32 64; do
    echo "=== batch=$BS ==="
    python -u scripts/bench_resnet18.py --batch_size "$BS"
done

# while：轮询等文件出现
while [ ! -f results/inference/inference_results.csv ]; do
    sleep 5
done
echo "结果文件已生成"
```

**数组**（脚本里偶尔会见到）：

```bash
MODELS=(resnet18 unet yolov8n)
echo "${MODELS[0]}"      # 第一个元素（下标从 0 开始）
echo "${MODELS[@]}"      # 全部元素
echo "${#MODELS[@]}"     # 元素个数
for m in "${MODELS[@]}"; do echo "$m"; done
```

### 5.4 函数

```bash
run_task() {
    local name="$1"        # local 限制作用域在函数内
    echo "--- 开始：$name ---"
    echo "参数个数：$#"
}

run_task "train_resnet18"
```

### 5.5 脚本排查三板斧

```bash
bash -n run_train.sh        # 只做语法检查，不执行（Syntax OK）
bash -x run_train.sh cuda   # 打印每条执行到的命令（调试）
cat -A run_train.sh         # 看隐藏字符（Windows 换行 \r\n 会让脚本报 $'\r': command not found）
```

> 💡 **经典坑：从 Windows 复制粘贴的脚本在 Linux 上跑不起来**，报 `bash: ./run.sh: /bin/bash^M: bad interpreter` 或 `$'\r': command not found`。原因就是 Windows 换行是 `\r\n`，Linux 只认 `\n`。解决办法：
>
> ```bash
> sed -i 's/\r$//' run.sh          # 去掉所有 \r
> # 或者
> dos2unix run.sh
> ```

---

## 第 6 章 实训的标准运行流程（逐条解释）

以实训一（DIBaS 细菌显微图像分类）为例，把整个流程和语法对应起来。

### 6.1 第 1 步：进入目录 + 设置环境变量

```bash
export WPATH="$PWD"
export HOME="$WPATH/.home"
export XDG_CONFIG_HOME="$HOME/.config"
export XDG_CACHE_HOME="$WPATH/.cache"
export MIOPEN_USER_DB_PATH="$XDG_CONFIG_HOME/miopen"
cd /public/SothisAI/learning_center
cd $WPATH/lesson_xjtxfl/
```

- `$PWD`：bash 内置变量，等于当前目录（和 `pwd` 命令的输出一致）。
- 先记住工作区，再把 `HOME`、缓存目录全部搬到可写位置（原因见第 3 章）。
- **为什么 `cd` 了两次？** 第一次进公共课程目录 `/public/SothisAI/learning_center`（**只读**，放的是课程材料），第二次用 `$WPATH` 拐回**你自己的可写目录**去跑实验。**这个顺序不能反**：`WPATH` 必须在你还没 `cd` 走之前就记下来。
- **这五条 `export` 是「会话级」的，重开终端就得重来。**

每个实训的可写目录不一样，命名是拼音首字母：

| 实训 | 目录 | 拼音含义 |
|---|---|---|
| 实训一 · 细菌图像分类 | `$WPATH/lesson_xjtxfl/` | 细菌图像分类 |
| 实训二 · 菌落检测计数 | `$WPATH/lesson_jljcjs/` | 菌落检测计数 |
| 实训三 · 果蔬品质分类 | `$WPATH/lesson_gspzfl/` | 果蔬品质分类 |
| 实训四 · 细胞核分割 | `$WPATH/lesson_xbhfg/` | 细胞核分割 |
| 专项 · 模型优化加速 | `$WPATH/lesson_mxyhjs/` | 模型优化加速 |

> ⚠️ **注意：课件 PPT 里的路径是旧版本，别照搬。** PPT 写的是 `cd lesson_xjtxfl/preset`，而运行说明文档给的是上面这套「先 export 再 `cd /public/...` 再 `cd $WPATH/lesson_xxx/`」的流程——课件里明确写了「PPT 中的运行路径与命令为旧版本，请勿照搬；实际操作请统一以实训课程运行说明文档中的最新命令为准」。
>
> **为什么这件事和语法有关？** 因为 `cd lesson_xjtxfl/preset` 是**相对路径**（从当前位置往下找），而 `cd /public/SothisAI/learning_center` 是**绝对路径**（从根目录开始找）。照抄旧文档最容易得到的报错就是：
>
> ```
> bash: cd: lesson_xjtxfl/preset: No such file or directory
> ```
>
> **这类报错的解药永远是同一套动作：`pwd` 看我在哪，`ls` 看这儿有什么，`find / -name "run_check.sh" 2>/dev/null` 看它到底在哪。**

> 💡 **顺便理解「预置工程 + 可写输出」的目录设计**（这是平台很贴心的安排，也是你要交作业的地方）：
>
> ```
> /public/SothisAI/learning_center/     ← 课程代码：只读，别改（改了也写不进去）
> $WPATH/lesson_xjtxfl/                 ← 你的运行目录：可写
>   └── preset/
>       ├── scripts/      训练/评估/benchmark 脚本
>       ├── data/         数据集与划分文件
>       ├── checkpoints/  训练产出的权重（实训一）
>       ├── results/      日志、指标 CSV/JSON（所有实训）
>       └── figures/      曲线图、混淆矩阵、样例图
> ```
>
> 文档里那句「运行生成的模型权重、日志、评估结果和图片保存在 `$WPATH/lesson_xjtxfl/`」，说的就是这个位置。**交作业前要下载的，就是 `results/` 和 `figures/` 里的东西。**

### 6.2 第 2 步：检查环境与数据

```bash
bash run_check.sh
```

输出（节选）：

```
COMMAND: environment and DIBaS dataset check
torch: 2.4.1
hip: 6.1.25065
torchvision: 0.19.1
numpy: 1.24.3
cuda available: True
device: BW
class dirs: 33
image files: 660
split file exists: True
```

**怎么读这份输出**：

| 行 | 含义 |
|---|---|
| `torch: 2.4.1` | PyTorch 版本正确 |
| `hip: 6.1.25065` | HIP 有值 → 确实是 DCU 环境 |
| `cuda available: True` | 加速卡能被 PyTorch 用上 |
| `device: BW` | 设备名是 BW（DCU 的标识） |
| `class dirs: 33` | 找到 33 个类别目录 |
| `image files: 660` | 找到 660 张图像（33 × 20） |
| `split file exists: True` | 划分文件存在 |

### 6.3 第 3 步：训练

```bash
bash run_train.sh cuda
```

脚本内部实际执行的是（文档里的 `COMMAND:` 行）：

```bash
python -u scripts/train_resnet18.py \
    --device cuda \
    --data_dir data/processed/DIBaS_33class_official660 \
    --epochs 30 \
    --batch_size 16 \
    --lr 1e-4
```

参数逐项理解：

| 参数 | 含义 |
|---|---|
| `-u` | Python 的 unbuffered 模式：**日志实时输出，不攒着** |
| `--device cuda` | 用加速卡（DCU 也写 cuda） |
| `--data_dir ...` | 数据目录 |
| `--epochs 30` | 训练 30 轮 |
| `--batch_size 16` | 每批 16 张图 |
| `--lr 1e-4` | 学习率 0.0001（`1e-4` 是科学计数法） |

训练输出长这样：

```
Epoch  1/30 | loss 2.6591 | acc 0.337 | f1 0.339 | 27.3s
Epoch 10/30 | loss 0.8552 | acc 0.964 | f1 0.965 | 26.6s
Epoch 30/30 | loss 0.7171 | acc 0.996 | f1 0.996 | 26.5s

Training complete!
  Best train accuracy: 0.9962
  Total time:          802.2s
EXIT_CODE: 0
```

训练产物：

```
checkpoints/resnet18_best_dcu_hip_33class.pth                 # 权重
results/logs/train_resnet18_latest.log                        # 日志
results/training/training_summary_dcu_hip_33class.json        # 汇总
figures/training/training_curves_dcu_hip_33class.png          # 曲线图
```

**训练要跑 13 分钟左右**，这时候 `tail -f` 就派上用场了。

### 6.4 第 4 步：评估

```bash
bash run_eval.sh
```

实际执行：

```bash
python -u scripts/eval_resnet18.py \
    --data_dir data/processed/DIBaS_33class_official660 \
    --checkpoint checkpoints/resnet18_best_*.pth \
    --split_file data/splits/split_indices_33class_official660_seed42.csv
```

**注意 `checkpoints/resnet18_best_*.pth` 里的 `*`**：这是通配符，shell 会把它展开成实际的文件名。这就是为什么要用第 2.9 节的通配符知识——脚本里写 `*` 是有意的，为了「不管时间戳怎么变都能找到最新权重」。

评估结果：

```
Test samples: 132 from 33 classes
Accuracy: 0.9697
Macro F1: 0.9684
Macro Precision: 0.9702
Macro Recall: 0.9697
```

### 6.5 第 5 步：推理性能测试

```bash
bash run_benchmark.sh
```

实际执行：

```bash
python -u scripts/bench_resnet18.py
```

输出：

```
COMMAND: python -u scripts/bench_resnet18.py
batch=1: 3.7ms, 270 img/s
batch=8: 3.7ms, 2187 img/s
batch=16: 4.0ms, 3989 img/s
batch=32: 7.1ms, 4536 img/s
batch=64: 12.7ms, 5025 img/s
Saved: results/inference/inference_results.csv
```

### 6.6 第 6 步：核对产物

```bash
ls -lht checkpoints/ results/inference/ results/training/ figures/training/
cat results/inference/inference_results.csv
```

### 6.7 提交物清单（实训一要交什么）

按实训要求，最终提交三样：

1. **细菌图像分类训练、评估和推理性能测试的实验记录**
2. **训练过程、测试集评估和 benchmark 的运行结果文件**（就是上面那些 `.log` / `.csv` / `.json` / `.png`）
3. **训练结果、测试指标与推理性能的简要分析说明**

第 3 条是很多人丢分的地方。写分析时至少要包含：

- 训练收敛情况：30 个 epoch 的 loss / acc / F1 变化（`2.6591 → 0.7171`，`0.337 → 0.996`），总耗时 802.2s。
- 测试集指标：Accuracy 0.9697、Macro F1 0.9684，**并解释为什么 33 类小样本要看 Macro F1 而不是只看 Accuracy**（类别样本极少，Macro 对每一类平等对待）。
- 推理性能：batch 越大吞吐越高（270 → 5025 img/s），但延迟也在涨（3.7ms → 12.7ms），**说明吞吐和延迟是一对权衡**，要结合部署场景选 batch。
- 环境确认：device 是 BW（DCU），HIP 6.1.25065。

> 📌 这些结论就是「分析说明」该有的样子——**用你跑出来的数字说话**，而不是复述文档。

### 6.8 后面几个实训的脚本规律（同一套语法）

实训一跑通之后，二、三、四、五其实是同一套东西换个模型。列出来你就可以「见招拆招」：

```bash
# 实训二 · 菌落检测计数（YOLOv8n）
cd $WPATH/lesson_jljcjs/
bash run_check.sh
bash run_train.sh
bash run_eval.sh
bash run_infer.sh
bash run_benchmark.sh

# 实训三 · 果蔬品质分类（双模型对比）
cd $WPATH/lesson_gspzfl/
bash run_train_mv3.sh        # MobileNetV3-Small
bash run_train_effb0.sh      # EfficientNet-B0
bash run_compare.sh          # 汇总成对比表

# 实训四 · 细胞核分割（U-Net）
cd $WPATH/lesson_xbhfg/
bash run_check.sh
bash run_train.sh            # 50 轮
bash run_eval.sh             # Dice / IoU / Pixel Acc / Precision / Recall
bash run_infer.sh            # 输出 原图/真实mask/预测mask 三列对照
bash run_benchmark.sh        # batch=1/2/4/8/16

# 专项 · 模型优化加速
cd $WPATH/lesson_mxyhjs/
bash run_check.sh                # 检查设备、权重、评估数据
bash run_resnet18_opt.sh         # FP32 / AMP / channels_last / 剪枝
bash run_unet_opt.sh
bash run_yolo_opt.sh
bash run_profile.sh              # Profiler 找耗时算子
bash run_high_resolution_stress.sh  # 640 / 2000 / 3000 分辨率压力测试
```

**脚本命名规律**（看名字就知道干什么）：

```
run_<动作>[_<模型>].sh
     │        └─ mv3 / effb0 / resnet18 / unet / yolo
     └─ check / train / eval / infer / bench(benchmark) / compare / opt / profile
```

**五处需要留心的语法差异**：

1. **`--device cuda` 与 `--device 0` 是两种约定**。实训一写 `cuda`（字符串，表示用加速卡），实训二/三/四/专项写 `0`（整数，表示第 0 号卡）。在 DCU 上两者都指向同一块 `BW` 卡。**照抄各自的文档，不要混用。**

2. **下划线与短横线两套参数风格并存，不能互换**：

   | 实训 | 风格 | 例子 |
   |---|---|---|
   | 实训一 | 下划线 | `--data_dir`、`--batch_size`、`--split_file` |
   | 实训三 / 四 / 专项 | 连字符 | `--batch-size`、`--img-size`、`--results-dir`、`--figures-dir` |

   `argparse` 定义的是 `--batch_size`，你写 `--batch-size` 会直接报 `unrecognized arguments`。**看到这种报错，先 `--help` 看一眼真实参数名。**

3. **`--batch-size` 和 `--batch-sizes` 是两个不同的参数**：前者是「一个值」（训练时的批大小），后者是「一串值」（benchmark 要测的多个批大小）：

   ```bash
   python scripts/bench_mv3.py --batch-sizes 1 8 16 32 64 --img-size 224 --device 0
   ```

   注意 `1 8 16 32 64` 后面**没有逗号**——在 shell 里它们就是 5 个独立的参数，由 `argparse` 的 `nargs="+"` 收集成列表。**这是命令行和 Python 列表字面量最大的视觉差异**（Python 里你得写 `[1, 8, 16, 32, 64]`）。

4. **`$PYTHON` 变量**（实训一、三的脚本里出现）：

   ```bash
   PYTHON=${PYTHON:-python}
   $PYTHON -u scripts/train_resnet18.py ...
   ```

   意思是「如果你在环境里设过 `PYTHON` 就用你的，否则用默认的 `python`」。所以你可以：

   ```bash
   PYTHON=python3 bash run_train.sh cuda     # 临时换解释器，只对这一条命令生效
   ```

5. **`DATA_ROOT="$(pwd)/..."` 解决绝对路径问题**（实训二）：

   ```bash
   DATA_YAML="data/AGAR_subset_yolo/agar.yaml"
   DATA_ROOT="$(pwd)/data/AGAR_subset_yolo"
   ```

   而 Python 侧读的是 `cfg["path"] = os.environ["DATA_ROOT"]`——**Shell 里设出来的变量，Python 用 `os.environ` 读**（第 7.6 节）。这也是实训二里唯一一处「Shell 与 Python 通过环境变量握手」的地方：因为 YOLO 的 `agar.yaml` 里 `path:` 必须写绝对路径，脚本就用 `$(pwd)` 现场拼一个。

---

## 第 7 章 Python 基础：脚本、参数与 import

### 7.1 怎么运行一个 Python 脚本

```bash
python scripts/train_resnet18.py              # 最基本
python3 scripts/train_resnet18.py             # 明确指定 Python 3
python -u scripts/train_resnet18.py           # 不缓冲输出（日志实时可见，实训常用）
python -m pip install something               # 以模块方式运行（-m）
```

**`-u` 到底解决什么问题**：Python 默认会对输出做缓冲，可能你盯着屏幕十几分钟什么都没看到，以为卡死了。`-u` 强制实时输出。**所有长时间跑的任务都建议加 `-u`。**

### 7.2 一个 Python 脚本的骨架

```python
#!/usr/bin/env python3
"""ImageNet 预训练 ResNet18 在 DIBaS 上微调。"""
import argparse
import os
import sys
from pathlib import Path

import torch
import torch.nn as nn
import torchvision


def main():
    parser = argparse.ArgumentParser(description="Fine-tune ResNet18 on DIBaS")
    parser.add_argument("--data_dir", type=str,
                        default="data/processed/DIBaS_33class_official660")
    parser.add_argument("--device", type=str, default="cuda")
    parser.add_argument("--epochs", type=int, default=30)
    parser.add_argument("--batch_size", type=int, default=16)
    parser.add_argument("--lr", type=float, default=1e-4)
    parser.add_argument("--amp", action="store_true",
                        help="启用自动混合精度")
    args = parser.parse_args()

    print(f"Device: {args.device}")
    print(f"Data: {args.data_dir}")
    # ... 训练逻辑 ...
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

三个固定套路，看懂了就能读任何实训脚本：

**① `if __name__ == "__main__":`**

意思是「只有这个文件被**直接运行**时才执行下面的代码；被 `import` 时不执行」。它是「入口函数」的分界线。

**② `argparse`：命令行参数从哪来**

```python
parser.add_argument("--epochs", type=int, default=30)
```

| 部分 | 含义 |
|---|---|
| `--epochs` | 命令行里写的名字（长选项用两个减号） |
| `type=int` | 自动把字符串转成整数（命令行传来的永远是字符串！） |
| `default=30` | 不传就用 30 |
| `action="store_true"` | 开关型参数，写了就是 `True`，不写就是 `False` |

于是这些写法都合法：

```bash
python train.py --epochs 30              # 空格分隔
python train.py --epochs=30              # 等号分隔（效果完全一样）
python train.py --amp                    # 开关参数
python train.py --epochs 5 --batch_size 32
python train.py --help                   # 自动生成的帮助
```

**③ `sys.exit(main())`：把返回值变成退出码**

`main()` 返回 0 → 进程退出码 0 → 脚本外层 `echo $?` 得到 0 → 实训文档里 `EXIT_CODE: 0`。这就是前面第 2.6 节退出码的来源。

### 7.3 命令行字符串 → Python 类型

一个必须建立的认知：

```bash
python train.py --epochs 30
```

在这条命令里，`30` 传进 Python 时**是字符串 `"30"`**，是 `argparse` 根据 `type=int` 帮你转成了整数 `30`。

这也是为什么**漏写值会直接报错退出**：

```bash
$ python train.py --epochs
usage: train.py [-h] [--epochs EPOCHS] ...
train.py: error: argument --epochs: expected one argument
$ echo $?
2
```

**退出码 2 = 参数用法错误**，不是你的代码有 bug，而是命令没敲对。

### 7.4 `import` 与路径

```python
import torch                                    # 第三方库
import torch.nn as nn                           # 子模块起别名
from pathlib import Path                        # 只导入一个名字
from torch.utils.data import DataLoader
```

**`ModuleNotFoundError: No module named 'torch'`** 是最常见的报错之一，说明当前 Python 环境里没装这个包，或者你不在正确的环境里：

```bash
which python              # 我现在用的 python 是哪个？
python -c "import sys; print(sys.executable)"   # 看完整路径
python -m pip list | grep torch                  # 装了哪些包
```

### 7.5 路径拼接：为什么用 `pathlib` 而不是字符串相加

```python
from pathlib import Path

root = Path("data/processed/DIBaS_33class_official660")
class_dir = root / "Acinetobacter_baumannii"     # / 号拼接，跨平台安全
print(class_dir)                                  # data/.../Acinetobacter_baumannii

for f in sorted(class_dir.iterdir()):
    if f.suffix.lower() in {".jpg", ".png", ".tif"}:
        print(f.name)
```

**不要用 `root + "/" + name`**：容易多一个或少一个斜杠，Windows 上还会踩反斜杠的坑。

### 7.6 环境变量：Python 侧怎么读

```python
import os

wpath = os.environ.get("WPATH", ".")
print(wpath)
```

对应关系：

| Shell | Python |
|---|---|
| `export WPATH="$PWD"` | `os.environ.get("WPATH")` |
| `echo "$WPATH"` | `print(os.environ["WPATH"])` |
| `export CUDA_VISIBLE_DEVICES=0` | 影响 `torch.cuda.device_count()` |

### 7.7 DCU 相关的最小代码模板（实训必背）

```python
import torch
import torch.nn as nn
import torchvision

# 1. 设备：DCU 上也写 "cuda"
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print("device:", device)
if device.type == "cuda":
    print("name  :", torch.cuda.get_device_name(0))       # BW = DCU
    print("hip   :", torch.version.hip)                    # 有值 = DCU

# 2. 迁移学习：换掉分类头
model = torchvision.models.resnet18(weights="IMAGENET1K_V1")
in_features = model.fc.in_features
model.fc = nn.Linear(in_features, 33)      # DIBaS 是 33 类
model = model.to(device)

# 3. 推理计时：必须 synchronize
import time
model.eval()
x = torch.randn(16, 3, 224, 224, device=device)

with torch.inference_mode():               # 比 no_grad 更快，推理专用
    for _ in range(10):                    # warmup：跑掉首次编译/缓存开销
        _ = model(x)
    torch.cuda.synchronize()

    t0 = time.perf_counter()
    _ = model(x)
    torch.cuda.synchronize()               # 等加速卡真正算完，否则测的是「派发时间」
    latency_ms = (time.perf_counter() - t0) * 1000

print(f"latency={latency_ms:.2f}ms, throughput={16 * 1000 / latency_ms:.0f} img/s")
```

**两个容易被忽略但决定结果可信度的细节**：

1. **`torch.cuda.synchronize()`**：CUDA/HIP 的调用是**异步**的。不同步的话，`time.perf_counter()` 量到的只是「把任务丢给加速卡」的时间，不是真正算完的时间——benchmark 会假快。
2. **warmup**：第一次前向要建缓存、编译算子，特别慢。不预热就计时，会把这一次的异常值算进去。

> 📐 **各实训的 benchmark 参数并不相同，抄的时候要看清**：
>
> | 实训 | batch 取值 | warmup | 计时次数 |
> |---|---|---|---|
> | 实训一（ResNet18） | 1 / 8 / 16 / 32 / 64 | 10 | 100 |
> | 实训二（YOLOv8n） | 1 / 2 / 4 / 8（显存 ≥8GB 时加 16） | 3 | 10 |
> | 实训三（MobileNetV3 / EfficientNet） | 1 / 8 / 16 / 32 / 64 | 由 `--warmup` 给出 | 由 `--iters` 给出 |
> | 实训四（U-Net） | 1 / 2 / 4 / 8 / 16 | 10 | 50 |
>
> 实训二那个「显存 ≥8GB 才追加 batch=16」就是一段值得学的条件逻辑：
>
> ```python
> if device_str != "cpu" and torch.cuda.get_device_properties(0).total_memory >= 8 * 1024**3:
>     batch_sizes.append(16)
> ```
>
> 注意 `8 * 1024**3`：`**` 是 Python 的幂运算，`1024**3` 就是 1GB 的字节数。**读代码时遇到这种写法，先在脑子里代成「8GB」。**

> 🔎 **顺带一个很实用的「读数字」技能**：不同实训的 benchmark 输出口径不一样，看的时候要分清「**一次推理的延迟**」和「**每张图的延迟**」。
>
> ```text
> 实训一输出：batch=64: 12.7ms, 5025 img/s        ← 12.7ms 是「整个 batch」的延迟
> 实训二输出：4 → latency_per_image_ms=338.40      ← 338.40ms 是「换算到每张图」的
> ```
>
> 混用这两个口径会算出荒谬的结论。**自检办法：`吞吐 × 延迟 ≈ batch_size`。**
>
> 顺便，这套换算还能让你「读」出模型的真实瓶颈。用每张图的延迟对比（数据来自课件实测）：
>
> | batch | YOLOv8n 每张图 | MobileNetV3 每张图 | EfficientNet-B0 每张图 | U-Net 每张图 |
> |---|---|---|---|---|
> | 1 | 378.96 ms | 6.42 ms | 10.08 ms | 4.88 ms |
> | 8 | 338.47 ms | 6.44 ms | 10.16 ms | 3.55 ms |
> | 16 | 338.10 ms | 6.44 ms | 10.13 ms | 3.41 ms |
> | 64 | — | 6.44 ms | **18.15 ms** ⚠️ | — |
>
> EfficientNet-B0 在 batch=64 时每张图突然变慢——**这就是「batch 不是越大越好」的直接证据**（显存压力或算子切换导致效率反转）。这类现象，只有把「延迟」和「吞吐」两个数字一起看才能发现。

### 7.8 用 pip 装包（以及平台上的注意事项）

```bash
python -m pip install numpy                # 推荐写法：python -m pip
python -m pip install --user numpy         # 装到用户目录（无 root 权限时用）
python -m pip list                         # 已装的包
python -m pip show torch                   # 某个包的详情
python -m pip install -r requirements.txt  # 按清单装
```

> 💡 用 `python -m pip` 而不是直接 `pip`，能保证「装到当前正在用的那个 Python 里」，避免多环境时装错地方。
>
> 在只读的公共目录/系统目录上安装会失败（`Permission denied`）。实训平台请优先用环境里已经装好的 PyTorch-DAS，不要随便升级 torch——**换版本极易把 DCU 适配搞坏**。

---

## 第 8 章 报错信息怎么读：从一头雾水到知道去哪查

### 8.1 读报错的固定顺序

```
Traceback (most recent call last):                 ← ① 从下往上读！
  File "scripts/train_resnet18.py", line 87, in main
    train_one_epoch(...)
  File "scripts/train_resnet18.py", line 42, in train_one_epoch
    out = model(images)
  File "/opt/.../torch/nn/modules/module.py", line 1553, in _call_impl
    return forward_call(*args, **kwargs)
RuntimeError: CUDA out of memory. Tried to allocate ...   ← ② 最后一行才是根因
```

**① Python 的 traceback 要「从下往上」读**：最后一行是真正的错误，上面是调用栈（谁调用了谁）。

**② 最后一行里，冒号前面是错误类型**（`RuntimeError`、`FileNotFoundError`、`ModuleNotFoundError`），冒号后面是描述。

**③ 中间只要有你自己写的文件路径（`scripts/train_resnet18.py`），就从那里开始看**——那才是你的代码。

### 8.2 高频报错对照表（实训版）

| 报错 / 现象 | 真正原因 | 怎么办 |
|---|---|---|
| `bash: xxx: command not found` | 命令名打错，或没这个软件 | `which xxx`；检查拼写；换用完整路径 |
| `python: can't open file 'xxx.py': [Errno 2] No such file or directory` | 路径不对 / 不在正确的目录 | `pwd` + `ls`；用 Tab 补全 |
| `No such file or directory`（打开数据/权重时） | 相对路径的基准目录不对 | 先 `cd` 到文档规定目录；或改用绝对路径 |
| `Permission denied` / `Read-only file system` | 往只读的公共/系统目录写 | 检查那 5 条 `export` 有没有执行；确认 `HOME` 指向可写目录 |
| `ModuleNotFoundError: No module named 'torch'` | 当前 Python 环境没这个包 / 不在 DCU 环境 | `which python`；`python -m pip list` |
| `CUDA out of memory` | batch 或输入尺寸太大 | 减小 `--batch_size`；开 AMP；用 `torch.inference_mode()` |
| `RuntimeError: Expected all tensors to be on the same device` | 模型和数据没搬到同一设备 | 确认 `model.to(device)` 且 `images.to(device)` |
| `AttributeError: 'Namespace' object has no attribute 'xxx'` | 参数名写错 / `add_argument` 没定义 | 对照 `--help` 输出 |
| `error: argument --epochs: expected one argument` | 长选项后面漏了值（退出码 2） | 补上值：`--epochs 30` |
| `$'\r': command not found` / `bad interpreter` | 脚本是 Windows 换行 | `sed -i 's/\r$//' xxx.sh` |
| `Argument list too long` | 通配符展开出了太多文件 | 改用 `find -exec` 或分批处理 |
| 结果波动很大 / 第一次特别慢 | 没 warmup、没 synchronize | 按 7.7 的模板写计时 |
| 训练 loss 不下降 | 学习率太大 / 数据路径错了 | 迁移学习用 `1e-4` 这种小学习率；先 `run_check.sh` |
| 训练 loss 降但测试差 | 过拟合（小样本典型问题） | 数据增强、`weight_decay`、用预训练权重 |
| 优化了但**没有变快** | 优化手段与后端/算子实现不匹配 | 见下面「优化为什么没生效」 |

**关于「优化了但没变快」**——这是模型优化专项的实测结论，很反直觉但非常重要：

| 优化手段 | DCU 实测加速比 | 质量变化（Accuracy / Macro F1 / Dice） | 结论 |
|---|---|---|---|
| AMP（自动混合精度） | ResNet18 4462 → **8244 img/s（1.85×）**；U-Net 28.7 → **16.2 ms（1.78×）** | ResNet18 1.0000 → 1.0000；U-Net Dice 0.9126 → 0.9125 | ✅ **有效且几乎无损，首选** |
| `channels_last` | ResNet18 7.17 → 7.40 ms（**0.97×**） | — | ❌ 无收益 |
| 非结构化剪枝 30% | ≈1.00×（7.1709 → 7.1715 ms） | ResNet18 1.0000 → 0.9924；U-Net Dice 0.9126 → **0.8922**、Recall 0.9070 → 0.8532 | ❌ 不变快，还掉精度 |
| YOLOv8n FP16 | 9.34 → 9.43 ms（**0.99×**） | mAP50 0.6622 → 0.6614（基本不变） | ❌ 该后端上没加速 |

> 💡 **记住这句话：优化不是「一定更快」，而是「在特定硬件 + 特定算子上更快」。** 剪枝去掉的是「权重个数」，dense 算子并不利用稀疏性（虽然稀疏率确实到了 0.2997），所以速度不变、精度反而下降——这就是为什么优化专项要求「剪枝后必须做质量验证」，也是为什么任何优化都必须**先测基线、再改一个变量、再测**。
>
> 顺带看一个「参数调小」的反例：YOLO 把输入从 640 降到 416，mAP50 从 0.6622 掉到 0.5618、mAP50-95 从 0.4106 掉到 0.3156——**速度确实会快，但精度掉得更多**。所以这也解释了为什么另一个纯语法性质的坑值得记下：**YOLO 的输入尺寸会按 stride=32 自动对齐**——你传 2000，模型内部按 2016 算；传 3000，按 3008 算。压力测试里「我明明传了 2000 为什么更慢」这类疑问，答案往往在模型的对齐逻辑里，而不在命令行上。

**测显存峰值的正确写法**（模型优化专项用到的，值得背）：

```python
torch.cuda.reset_peak_memory_stats()      # 先清零
with torch.inference_mode():
    _ = model(x)
allocated = torch.cuda.max_memory_allocated() / 1024**2   # MB，实际分配
reserved  = torch.cuda.max_memory_reserved()  / 1024**2   # MB，缓存池保留
```

> `allocated` 和 `reserved` 不一样：前者是张量真正占用的，后者是 PyTorch 缓存池向驱动要来的。**报 OOM 时看 allocated，判断「还能不能再塞大一点」时看 reserved。**
>
> 实测例子：U-Net 的 peak memory 从 FP32 的 **1269.03 MB** 降到 AMP 的 **1053.09 MB**——**精度省显存，降的是「缓存池」而不是「张量本身」**，这正是 `reserved` 那个数字的意义。
>
> 高分辨率压力测试的实测数据（DCU，`latency / throughput / peak_allocated`）：
>
> | 输入分辨率 | ResNet18 | YOLOv8n |
> |---|---|---|
> | 640×640 | 3.11 ms / 321 img/s / 99 MB | 9.34 ms / 107 img/s / 32 MB |
> | 2000×2000 | 15.58 ms / 64 img/s / 577 MB | 32.79 ms / 30 img/s / 228 MB |
> | 3000×3000 | 31.54 ms / 32 img/s / 1246 MB | 66.16 ms / 15 img/s / 477 MB |
>
> **延迟和显存是同步涨的**，这条曲线就是「为什么输入尺寸要权衡」的证据。

### 8.3 优化之前先「体检」：Profiler 的最小用法

模型优化专项的核心方法论是「**先 Profile，再优化**」，对应的语法其实很短：

```python
import torch
from torch.profiler import profile, ProfilerActivity

activities = [ProfilerActivity.CPU, ProfilerActivity.CUDA]   # DCU 上也写 CUDA

with profile(activities=activities, record_shapes=True, profile_memory=True) as prof:
    with torch.inference_mode():
        for _ in range(10):
            _ = model(x)

print(prof.key_averages().table(sort_by="cuda_time_total", row_limit=20))
```

**输出怎么读**：这是一张按「加速卡上的总耗时」排序的表，前几行就是真正的瓶颈。

真实数据（课件实测）：ResNet18 在 DCU 上**卷积约占 75%**（由 MIOpen 调用 Winograd / implicit GEMM 卷积内核）；U-Net 在 DCU 上**卷积约 68% + 双线性上采样约 11.83%**。

> 💡 这张表的意义在于**它把「优化谁」变成了一个可以用数字回答的问题**。卷积占 75%，就去优化卷积（AMP 有效）；如果数据加载占大头，那就该去调 `DataLoader` 的 `num_workers`，而不是折腾算子。
>
> 顺带认识两个名字：**MIOpen** 是 DCU 上的深度学习算子库（对应 NVIDIA 的 cuDNN）；**`cat`** 在 Profiler 里指的是 `torch.cat` 特征拼接（U-Net 的跳跃连接用的就是它）。

### 8.4 四条排查纪律

1. **先 `pwd` + `ls`**：90% 的「文件找不到」都死在这里。
2. **一次只改一个变量**：先原样跑通，再改参数；改了之后报错，就知道是谁的锅。
3. **保留完整日志**：`python -u train.py 2>&1 | tee train.log`（`tee` 同时输出到屏幕和文件）。
4. **报错原文直接搜**：把最后一行的错误类型和描述粘进搜索引擎，通常第一条就是答案。

### 8.5 `tee`：既要看屏幕，又要存文件

```bash
bash run_train.sh cuda 2>&1 | tee results/logs/my_train_run.log
```

`tee` 像水管三通：一份流向屏幕，一份写进文件。**做实验记录必用**——提交物要求「实验记录」，`tee` 就是最省事的留痕方式。

---

## 第 9 章 自测清单：这 20 条能默写，实训就不慌了

不看答案，能在终端里敲出来并解释含义，就算过关：

**导航与文件**

1. 显示当前所在目录，并列出这一层所有文件（含隐藏文件、大小可读、按时间倒序）
2. 进入 `/public/SothisAI/learning_center`，再回到上一级，再回到上一次待过的目录
3. 在当前目录下找所有 `.pth` 文件
4. 把 `figures/training/` 整个目录复制成 `figures/training_bak/`
5. 看一个日志文件的最后 30 行，并实时跟踪它

**Shell 语法**

6. 定义一个变量 `NAME=resnet18`，然后输出 `resnet18_best.pth`（正确使用花括号）
7. 说明 `"$VAR"`、`'$VAR'`、`$VAR` 三种写法的区别
8. 把 `pwd` 的输出存进变量 `WPATH`
9. 解释 `bash run_train.sh cuda` 里 `cuda` 去哪了，脚本里用什么接住
10. 判断上一条命令是否成功，并说明 0 代表什么
11. 写一条命令：先建目录 `results`，成功后再进入它
12. 把某条命令的正常输出和报错都写进 `all.log`
13. 从日志里筛出含 `Epoch` 的行，只取最后 3 行
14. 说明 `source env.sh` 和 `bash env.sh` 的区别
15. 写一个 for 循环，对 1、8、16、32、64 依次执行同一个命令

**环境与脚本**

16. 完整写出实训开头那 5 条 `export`，并解释为什么必须改 `HOME`
17. 说明为什么重开终端后要重新 `export`
18. 写一个带 shebang、带 `set -e`、能接收第 1 个参数（缺省 `cuda`）的 `run.sh`
19. 用 `bash -n` 检查脚本语法，用 `bash -x` 调试执行

**Python**

20. 写一个用 `argparse` 接收 `--epochs`（整数，默认 30）和 `--amp`（开关）的脚本，并解释 `type=int` 为什么必要

<details>
<summary>点开看部分参考答案要点</summary>

1. `ls -lhta`
2. `cd /public/SothisAI/learning_center` → `cd ..` → `cd -`
3. `find . -name "*.pth"`
4. `cp -r figures/training figures/training_bak`
5. `tail -n 30 xxx.log` / `tail -f xxx.log`
6. `echo "${NAME}_best.pth"`
7. 双引号展开变量、单引号原样、不加引号还会按空格拆分且会展开通配符
8. `WPATH="$(pwd)"`
9. `cuda` 作为第 1 个位置参数传入，脚本里用 `$1`（常写成 `${1:-cuda}`）接住
10. `echo $?`；0 = 成功，非 0 = 失败
11. `mkdir -p results && cd results`
12. `command > all.log 2>&1`
13. `grep "Epoch" xxx.log | tail -n 3`
14. `source` 在当前 shell 执行（变量/export 生效并保留）；`bash` 开子进程执行（执行完就没了）
15. `for b in 1 8 16 32 64; do python bench.py --batch_size "$b"; done`
16. 见 3.3 节；改 `HOME` 是为了让配置、缓存、MIOpen 数据库写进可写目录，避开只读的公共目录
17. `export` 只作用于当前 shell 会话；新终端是新会话
18. 见 5.1 节示例
19. `bash -n run.sh` / `bash -x run.sh cuda`
20. 命令行参数传进来永远是字符串，`type=int` 负责转换；`action="store_true"` 表示开关

</details>

---

## 附录 A 常用命令速查表

### 导航

| 命令 | 作用 |
|---|---|
| `pwd` | 我在哪 |
| `ls -lht` | 详细列表，按时间倒序 |
| `cd path` / `cd ..` / `cd ~` / `cd -` | 切目录 / 上级 / 家目录 / 上一次 |
| `tree -L 2` | 树形看两层（不一定预装） |

### 文件

| 命令 | 作用 |
|---|---|
| `mkdir -p a/b` | 建多层目录 |
| `cp -r src dst` | 递归复制 |
| `mv a b` | 移动 / 改名 |
| `rm file` / `rm -r dir` | 删除（无回收站！） |
| `cat` / `head -n` / `tail -n` / `tail -f` / `less` | 看文件 |
| `find . -name "*.pth"` | 按名查找 |
| `du -sh dir` / `df -h` | 目录体积 / 磁盘余量 |
| `tar -xzvf x.tar.gz` / `unzip x.zip` | 解压 |

### 文本处理

| 命令 | 作用 |
|---|---|
| `grep -rn "kw" dir/` | 递归搜索关键词并显示行号 |
| `sed 's/a/b/g'` / `sed -n '1,10p'` | 替换 / 打印指定行 |
| `awk '{print $2}'` / `awk -F: '{print $1}'` | 取列 |
| `cut -d, -f1,3` | 按分隔符取列 |
| `sort -n` / `sort -h` | 数值 / 体积排序 |
| `uniq -c` | 去重计数（先 `sort`） |
| `wc -l` | 数行数 |
| `tee file` | 同时输出到屏幕和文件 |

### 进程与资源

| 命令 | 作用 |
|---|---|
| `top` / `htop` | 看 CPU / 内存（`q` 退出） |
| `ps aux \| grep python` | 找 Python 进程 |
| `kill PID` / `kill -9 PID` | 结束进程 |
| `Ctrl + C` | 终止前台任务 |
| `nvidia-smi` / `rocm-smi` | 看 GPU / DCU 状态（平台可能没装） |

### 效率

| 操作 | 作用 |
|---|---|
| `Tab` | 补全 |
| `↑` / `Ctrl + R` | 历史 / 搜索历史 |
| `Ctrl + A` / `Ctrl + E` | 行首 / 行尾 |
| `Ctrl + U` / `Ctrl + K` / `Ctrl + W` | 删前 / 删后 / 删一个词 |
| `Ctrl + L` | 清屏 |
| `history` / `!!` / `!$` | 历史列表 / 重跑上一条 / 上一条的最后一个参数 |

---

## 附录 B 术语中英对照

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Terminal | 终端 | 你输入命令的窗口 |
| Shell / Bash | 命令解释器 | 真正解析你输入的程序 |
| Environment Variable | 环境变量 | 贴在会话上的「便签」，程序都能读 |
| export | 导出 | 让变量能被子进程继承 |
| Pipe | 管道 | 把上一条命令的输出喂给下一条 |
| Redirection | 重定向 | 把输出写进文件而不是屏幕 |
| Exit Code | 退出码 | 0 = 成功，非 0 = 失败 |
| Wildcard / Glob | 通配符 | `*` `?` `[]`，由 shell 展开 |
| Argument / Option | 参数 / 选项 | 命令后面跟的东西 |
| Shebang | 释伴行 | `#!/usr/bin/env bash` |
| Permission | 权限 | r / w / x，读 / 写 / 执行 |
| Symlink | 软链接 | 类似 Windows 快捷方式 |
| Latency | 延迟 | 一次推理要多久 |
| Throughput | 吞吐量 | 每秒能处理多少 |
| Warmup | 预热 | 正式计时前先跑几次 |
| Benchmark | 基准测试 | 标准化地测性能 |
| Checkpoint | 权重文件 | 保存下来的模型参数 |
| Epoch / Batch | 轮次 / 批 | 训练遍历一遍数据 / 一次喂多少样本 |
| Accuracy / Macro F1 | 准确率 / 宏平均 F1 | 分类指标；小样本优先看 Macro F1 |
| AMP | 自动混合精度 | FP16 + FP32 混用，提速省显存 |
| DCU / HIP | 国产加速卡 / 其编程接口 | 代码里仍写 `cuda`，设备名显示 `BW` |

---

## 最后：写给同样被命令行卡住的你

我在实训里的真实感受是：**卡住我的从来不是深度学习，而是「路径、引号、环境变量」这些看起来最没技术含量的东西。**

但换个角度想，这也是一件好事——它们的规则是**有限且确定的**。命令只有那么多，语法只有那么多，踩过一次的坑不会踩第二次（大概吧）。

如果你也在备赛，我给三个具体建议：

1. **不要复制粘贴完就走**。每次复制一条命令，花 10 秒问自己：命令名是什么、参数去哪了、输出到哪。这三问能解决大部分困惑。
2. **把 `pwd`、`ls`、`echo $?` 练成本能**。它们是你和机器之间最基本的对话。
3. **留痕**。用 `tee` 保存每一条命令和输出。比赛最后要交「工程文档」（占 10%，是所有维度里最容易被忽视、也最容易拿满的一项），到时候你会感谢现在留痕的自己。

最后一个提醒：**命令行的熟练度是有复利的。** 今天为了跑通实训一学的这几十条语法，在实训二到实训五会一遍又一遍地用上——脚本结构没变，只是模型换了；`run_check.sh` → `run_train.sh` → `run_eval.sh` → `run_benchmark.sh` 这套节奏也没变。所以第一次一定要慢一点、理解透一点，后面会越来越快。

祝我们都顺利跑通，也祝国产算力越来越好用。

---

*本文基于 SCNet/DCU 实训平台的实训一（DIBaS 细菌显微图像分类）实际操作整理。平台命令以「实训课程运行说明文档」的最新版本为准——PPT 中的旧路径与旧命令请勿照搬。*

{% endraw %}
