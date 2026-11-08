# v0.6.55 工作记录（本地，未发布）

链：0.6.54（UI 包）→ 0.6.55。执行 2026-11-08 16:47 主对话裁决：分块 unique 包改号 v0.6.55，rebase 到 UI 0.6.54 之上。

## 包

- 工作区 `v0.6.55/`（base + src + patch_v55.py + build_pack.py + verify_pack.py + plan + manifest + REPORT + WORKLOG + tests/test_v55.py + tests/run_fuse_once.py + exp/ + EchoScene-Update.zip）。
- 包 507,888 B，sha256 `0f8d213adf8e600d486ee7272aadf4b221b582a1f707100782bf739e72f2d9a7`。
- 改 2 个文件（fuse.py + version.inc）。fuse.py 与 v0.6.54 包逐字节相同（sha256 `6be89144376ca7aa89848bb49b7a55990bdb801dab1bdf900970464b359faaa1`，60,313 B）；version.inc `d1613650…`。

## rebase 的做法（双清单）

- UI 清单（0.6.54，9,800 文件，托盘已删）是安装基底：目标清单 = UI 清单只换 fuse.py + version.inc 两个条目，仍 9,800 文件。
- 0.6.53 清单（9,801 文件）是 fuse 补丁的血缘基底：基底 fuse.py 哈希与 0.6.53 清单逐字相同（c3ff69ae…）；基底 version.inc 哈希与 UI 清单逐字相同。
- 为什么不能直接贴到 UI 清单的 fuse 上：UI 清单的 fuse 是 0.6.47 版（e4e0af29…），补丁锚点（0.6.52 流式化、0.6.53 yardstick 溢出那几处）不存在。整文件替换是唯一不重写、不重测的做法。
- 安全核查（本轮做过）：fuse.py 只引进 json/math/pathlib/numpy/PIL；调用方只传任务目录（0.6.47 pipeline.py:1269、UI app.py:138），签名 `stage_fuse(job_dir, stride=4)` 对得上；读的任务文件格式在 0.6.47–0.6.53 间没变；更新器要求 plan from_version 等于本机版本（0.6.54），目标清单差分正好是这两个文件。

## 量的结果

- 本轮重跑（v0.6.55 src，新任务副本，120 帧）：28.4 秒、峰值 416 MB、1,259,284 点；6 个产物文件与已记录的分块版哈希逐字节相同；job.json 除路径外相同；无溢出残留（exp/measure_rebase55.json）。
- 16:45 轮次的量测沿用（同一文件）：120 帧产物逐字节相同，峰值 945–980 MB → 408–478 MB；638 帧产物逐字节相同，2,679,654 点，峰值 4,532–4,552 MB → 1,086–1,120 MB。
- 测试：tests/test_v55.py 80/80（原 77 项 + 新增 3 项：UI 清单 9,800 文件、无托盘文件、src fuse 与 v0.6.54 版逐字节相同）。verify_pack.py 45/45（清单差分正好是 2 个文件；托盘仍删；UI web_app 原样；fuse 纯 CRLF、能编译）。

## 后果（记下来，别忘）

- **0.6.55 吃掉了 0.6.51/0.6.52/0.6.53 的 fuse.py 内容**（那三版只改 fuse.py + version.inc），三包不能在 0.6.55 之后再装。
- 仍等 rebase：0.6.48（voxel_export + web_app 日志节流；web_app 部分要搬到 UI 版 web_app 上）、0.6.49、0.6.50。它们的基底文件在 0.6.55 清单里原样，rebase 不受影响。
- fuse.py 里的注释仍写“in v0.6.54”——那是这处改动第一次成包时的号，文件逐字节相同才保得住量测证据，刻意没改。

## 状态

本地包已建好、已验证。等用户拍板后，再走正常发布流程（发中转站、装桌面、重启确认、用户实测）。桌面未碰、中转站未推。

## 桌面应用结果

（待填：发中转站 commit、桌面扳手应用结果、用户实测。）
