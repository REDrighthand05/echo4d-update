# WORKLOG — v0.6.60（rebase：0.6.49 搬到 0.6.59 之上）

- 版本：v0.6.60（链 0.6.59 → 0.6.60）。本地热更新包，未发布，未推中转站，未碰桌面。
- 时间：2026-11-09 06:40–07:05（CST）。
- 改了什么：`app/echo_scene/pipeline.py`（0.6.49 配对提速，原样搬运并与 0.6.56 的 stage_present 修复合并）+ `app/installer/version.inc`（改号）。
  - 配对：复用 stage_extract 已解好的缩略图（不再重解一遍视频）、每帧缩略图 PIL 进程内读（ffmpeg 回退留着）、相关匹配一次批量矩阵乘。真 20261105 采集 307 帧：73.1 → 24.7 秒，配对结果逐字相同（v0.6.49 实测）。
  - 合并证明：diff(本包, 0.6.49/src) 恰好是 0.6.56 的 stage_present hunk；diff(本包, 0.6.56/src) 恰好是 0.6.49 的配对 hunk；两函数源码逐项相同。真采集包已不在盘上，本轮在同规模合成数据上重量的配对循环：40.6 → 12.5 秒（3.25 倍），307/307 配上，映射逐字相同。
  - stage_present：迷你任务 CLI 验过，模板两处都有/只在根目录时 scene.html 与修前旧代码逐字节相同；无模板、接缝缺失都软失败（ok True、present_applied False）；points.bin 缺失仍 ok False。
- 为什么：0.6.49 建在 0.6.48 链上，0.6.54-ui～0.6.59 改号后它落在链外；配对提速是已量过的真改进，搬回来不丢。
- 验过：`tests/test_v60.py` 54/54（合并证明 + 配对行为 + stage_present 行为 + 热更新模拟应用/中途回滚/中断回滚）；`verify_pack.py` 44/44。包 507,505 B，sha256 `147ccf1e…9712ec`。
- 没动：fuse.py、voxel_export.py、web_app.py、voxels.py（冻结）、depth_video.py（冻结）、depth_batch.py、updater/hotupdate、web_ui。前端未动。
- 待用户拍板：v0.6.60 发布、最大化验证、采集器 v2.1.2 重测、走路视频。仍等 rebase：v0.6.50（链 0.6.60 → 0.6.61）。
