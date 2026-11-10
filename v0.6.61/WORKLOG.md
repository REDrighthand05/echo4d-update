# WORKLOG — v0.6.61（rebase：0.6.50 搬到 0.6.60 之上）

- 版本：v0.6.61（链 0.6.60 → 0.6.61）。本地热更新包，未发布，未推中转站，未碰桌面。
- 时间：2026-11-09 09:40–10:30（CST）。
- 改了什么：`app/echo_scene/depth_batch.py`（0.6.50 深度线程并行，原样搬运）+ `app/echo_scene/pipeline.py`（0.6.50 深度门 hunk，贴到 0.6.60 基底上）+ `app/installer/version.inc`（改号）。
  - 并行：每工作线程一个钉死单线程的 ORT 会话，16 帧一块流式出结果，顺序与输入一致，坏帧出 None；门：CPU 会话、≥2 核、≥8 帧才开，Base 模型上限 2 个会话，Small 开 4 个，`ECHO_DEPTH_WORKERS` 可改。
  - 兜底：并行半路出任何错都落回串行逐帧，只重做没存好的帧。
  - 合并证明：diff(本包 pipeline, 0.6.50/src) 恰好是 0.6.56 的 stage_present hunk；diff(本包 pipeline, 0.6.60/src) 恰好是 0.6.50 的深度门 hunk；depth_batch 与 0.6.50 src 逐字节相同；四个关键函数源码逐项相同。
  - 重跑实测（真 40 帧，DAV2 Small，发包的迭代器本身）：串行逐帧 57.4 秒；2 工作者 47.3 秒（1.21 倍，桌面 Base 模型的门）；4 工作者 39.6 秒（1.45 倍）；并行与逐帧逐值相同（差 0.0）。绝对耗时比 0.6.50 那轮浮动（2 工作者 38.0 → 47.3 秒），方向不变，REPORT 里如实写了。
  - stage_present：迷你任务 CLI 验过，模板两处都有时 scene.html 与修前旧代码逐字节相同。
- 为什么：0.6.50 建在 0.6.49 链上，0.6.54-ui～0.6.60 改号后它落在链外；深度线程并行是已量过的真改进（深度占整条链约 85% 的时间），搬回来不丢。这是最后一块等 rebase 的旧包——0.6.48/0.6.49/0.6.50 至此全部回到主链上。
- 验过：`tests/test_v61.py` 66/66（合并证明 + 门逻辑 + 12 真帧并行/串行对照 + 端到端 stage_depth + stage_present 行为 + 热更新模拟应用/中途回滚/中断回滚）；`verify_pack.py` 0 失败。包 510,870 B，sha256 `7dc8bf77…2334`。
- 没动：fuse.py、voxel_export.py、web_app.py、voxels.py（冻结）、depth_video.py（冻结）、updater/hotupdate、web_ui。前端未动。
- 待用户拍板：v0.6.48–v0.6.61 发布、最大化验证、采集器 v2.1.2 重测、走路视频。
