# WORKLOG — v0.6.57（窗口过程加固）

- 版本：v0.6.57（链 0.6.56 → 0.6.57）。本地热更新包，未发布，未推中转站，未碰桌面。
- 时间：2026-11-09 00:40–01:00（CST）。
- 改了什么：`app/web_app.py`（窗口过程加固）+ `app/installer/version.inc`（改号）。
  - WM_SIZE 分支不再同步调子窗口调整，改投递私有消息 `_WM_RESIZE_CHILD_DEFERRED`（WM_APP+1），消息循环里再调，带合并（拖动时最多挂一个）。
  - `_resize_wndproc` 改成薄壳兜底：inner 抛任何异常都记日志、链到原过程；没原过程回 HTCLIENT。任何 Python 异常逃不出窗口回调。
  - `win_toggle_max` 删掉直接调子窗口那次（它的最大化/还原本来就会产生 WM_SIZE，延后调整已覆盖）。
- 为什么：2026-11-08 20:59 桌面 EchoScene.exe 最大化→还原崩溃，WER 异常码 c000041d（STATUS_FATAL_USER_CALLBACK_EXCEPTION）——异常从窗口回调里逃出去了。调用栈和 app.log 拿不到（桌面 SSH 认证还等用户跑管理员那两行），确切抛出处未证明；本包关掉的是我们自己代码里全部已知的逃逸口。
- 验过：`tests/test_v57.py` 47/47（基底复现两处毛病 + 新版延后/合并/吞消息/兜底/失败路径 + 热更新模拟应用/中途回滚/中断回滚）；`verify_pack.py` 50/50。包 527,778 B，sha256 `1d9b88ec…e5a84235574c20be3a3`。
- 没动：fuse.py、pipeline.py、voxels.py（冻结）、depth_video.py（冻结）、voxel_export.py、updater/hotupdate、web_ui。前端未动。
- 待用户拍板：v0.6.48–v0.6.57 发布、最大化验证、采集器 v2.1.2 重测、走路视频。仍等 rebase：v0.6.48 / v0.6.49 / v0.6.50（0.6.48 的 web_app 部分下轮要搬到本包的 web_app 上，与本包 WM_SIZE 分支同一处，注意合并）。
