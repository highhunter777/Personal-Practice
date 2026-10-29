# 操作系统应用层实践 · Lab 仓库

> **AI 会话开场**：请先通读本文件再开工。让 AI「读 ~/os-lab/README.md，按第 7 节约定协作」即可。
> 本文件是唯一事实源：进度、状态、约定都以这里为准。

## 1. 目标

应用层路线学操作系统（不做内核开发）：C 语言 + Linux 系统编程，三大经典项目——
迷你 Unix Shell → Malloc Lab → 并发服务器（最终成小 HTTP 服务器）。
教材主线：CSAPP（实验载体）+ OSTEP（概念）+ TLPI（字典）。
追求真正理解并写出可运行的代码，不背概念。

## 2. 环境（2026-10-30 就绪）

| 项 | 配置 |
|---|---|
| 宿主机 | Windows 11 |
| Linux | WSL2 发行版 `Ubuntu-24.04`（官方 rootfs + `wsl --import`，VHDX 在 `E:\WSL\Ubuntu-24.04`，**勿迁回 C 盘**，C 盘仅余 ~33G） |
| 用户 | `hunter`（免密 sudo，见 `/etc/sudoers.d/hunter`；默认用户由 `/etc/wsl.conf` 指定） |
| apt 源 | 中科大 `mirrors.ustc.edu.cn` |
| 工具链 | gcc 13.3 / make / gdb / strace / ltrace / valgrind / /usr/bin/time / git / lsof / tree / ab / netcat-openbsd |
| git | WSL 内已配 `core.autocrlf=input` |
| 工作目录 | `~/os-lab/`，一切代码放这里，**禁止 /mnt/c/** |
| 编辑器 | VS Code + WSL 扩展（远程模式）；终端 Windows Terminal |

## 3. 环境验收（2026-10-30 六项全绿）

| # | 命令 | 结果 |
|---|---|---|
| ① | `gcc -g -Wall -Wextra hello.c -o hello && ./hello` | `Hello, OS!` |
| ② | `strace ./hello` | `write(1, "Hello, OS!\n", 11)` |
| ③ | `cat /proc/self/maps` | r--p / r-xp 段布局正常 |
| ④ | `/usr/bin/time -v ./hello` | Major 0 / Minor 70 |
| ⑤ | `gdb ./hello`：break main → run → bt | 正常 |
| ⑥ | `valgrind ./hello` | All heap blocks were freed, 0 errors |

## 4. 路线图（8 周）

| 周次 | 任务 | 交付物 | 状态 |
|---|---|---|---|
| 第 1 周 | strace 观察 + 复刻 cat/wc/cp（各 ~50 行） | 行为与真品一致 | ← 当前 |
| 第 2–3 周 | Shell Lab | trace01–16 全过 | 待开始 |
| 第 4–5 周 | Malloc Lab | mdriver 达标，ASan/valgrind 干净 | 待开始 |
| 第 6–8 周 | echo → 线程池 → epoll → mini HTTP | ab 压 500 并发稳定 | 待开始 |
| 之后（可选） | mini-docker 或 CS144 | 按兴趣 | — |

## 5. Lab 获取与验收

**Shell Lab**：`cd ~/os-lab && wget https://csapp.cs.cmu.edu/3e/shlab-handout.tar && tar xvf shlab-handout.tar && cd shlab-handout && make`
单条验证：`./sdriver -t trace01.txt -s ./tsh -a "-p"`，输出须与 `./tshref` 一致；快捷 `make test01 / make rtest01`。
功能点：fork/exec/waitpid、SIGINT、SIGCHLD 回收、内置命令(quit/jobs/bg/fg)、`>` `<` 重定向(dup2)、`|` 管道、`&` 后台。

**Malloc Lab**：`cd ~/os-lab && wget https://csapp.cs.cmu.edu/3e/malloclab-handout.tar && tar xvf malloclab-handout.tar && cd malloclab-handout`
开发期 `gcc -g -fsanitize=address`；阶段性 valgrind；最终 `./mdriver` 打分（Correct/Util/Thru）。
观察内核交互：`strace -e trace=brk,mmap ./mdriver -V`。路线：隐式 → 显式 → segregated free list。

**并发服务器**：`ab -n 10000 -c 500 http://localhost:8080/`；`lsof -p <pid>`、`ls /proc/<pid>/task`、`/usr/bin/time -v ./httpd`。
递进：多线程 echo → 加锁 → 生产者消费者(mutex+cond) → select → epoll → 小 HTTP 服务器。
WSL2 localhost 自动转发到 Windows，浏览器直接访问。

## 6. 坑位清单（遇到先对照）

| 坑 | 症状 | 处理 |
|---|---|---|
| 代码放 `/mnt/c/` | 编译慢一个数量级 | 全程 `~/os-lab` |
| CRLF | `\r: command not found`、Makefile 报错 | `core.autocrlf=input`；VS Code 右下角确认为 LF |
| perf | WSL2 定制内核版本不匹配 | 不用 perf，用 strace + valgrind + `time -v` |
| WSL 是虚拟机 | `wsl --shutdown`/重置丢文件 | 代码每日 push GitHub |
| gcc 警告被忽略 | 埋难查 bug | 强制 `-Wall -Wextra`，警告清零 |
| `wsl.exe` 从 Windows 侧传参 | 引号被破坏：管道符被解析、`$var` 被提前展开 | 复杂命令走 stdin：`wsl.exe -- bash -s <<'EOF' ... EOF` |
| 24.04 无 `nc` 包 | `no installation candidate` | 装 `netcat-openbsd` |
| 清华 apt 源 | HTTP/HTTPS 都 403 | 用中科大/阿里云/腾讯源 |
| C 盘空间 | 93% 占用 | WSL VHDX、下载缓存一律放 E 盘 |

## 7. AI 协作约定

1. **角色**：结对程序员 + 导师。用户写代码，AI 审代码、答疑、指出问题，**不替写完整实现**——先给思路/伪代码/关键片段，用户明确说「卡住了」再给更完整参考。
2. **先检查环境再动手**：涉及命令先确认 WSL2（`wsl -l -v` 显示 VERSION=2）、在 `~/os-lab` 下工作。
3. **调试优先用工具说话**：gdb / strace / valgrind / `time -v` 定位，不凭空猜。
4. **代码规范**：统一 `gcc -g -Wall -Wextra`；Makefile 用 tab；所有源文件 LF。
5. **改动范围**：只在对应 lab 目录内工作（shlab-handout / malloclab-handout / httpd 目录）。
6. **讲解深度**：答「为什么」结合内核机制（fork 写时复制、sbrk/mmap 边界等），不做内核实验。
7. **每次实验后**：提醒用观测工具回看（strace 看进程树、`time -v` 对比线程版 vs epoll 版上下文切换等）。

## 8. 参考资源

- CS:APP 3e Labs: https://csapp.cs.cmu.edu/3e/labs.html （shlab 说明：https://csapp.cs.cmu.edu/3e/shlab.pdf）
- OSTEP 免费在线: https://pages.cs.wisc.edu/~remzi/OSTEP/
- TLPI《Linux/UNIX 系统编程手册》当字典查
- CS144（可选）: https://cs144.github.io/

## 9. 当前进度

- [x] 环境搭建完成（2026-10-30）
- [x] 六项验收全绿
- [ ] 第 1 周小工具 cat / wc / cp 　**← 当前任务**
- [ ] Shell Lab
- [ ] Malloc Lab
- [ ] 并发服务器
- 当前卡点：无
- 下一步：写 `mycat.c`（先想：read 返回值的三种情况与循环条件；strace 对比真品 cat）

## 10. 进度日志

- **2026-10-30**：环境搭建完成。官方 rootfs 经 `wsl --import` 装至 E 盘（清华源 403 改中科大；`nc` 包名修正为 `netcat-openbsd`）。六项验收全绿，工具链齐。实验笔记随各 lab 目录沉淀。
