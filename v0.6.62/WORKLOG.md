# WORKLOG — v0.6.62（融合 trim：robust + intersection 合并，少一次整云写盘）

- 版本：v0.6.62（链 0.6.61 → 0.6.62）。本地热更新包，未发布，未推中转站，未碰桌面。
- 时间：2026-11-09 12:40–13:30（CST）。
- 改了什么：`app/echo_scene/fuse.py` + `app/installer/version.inc`（改号）。
  - 融合 trim：robust（1–99%）trim 不再先落成 trim1 spill，keep 掩码直接在 intersection（K=2）trim 里按掩码流式消费；lin 键、unique、逆标签、pair 码、keep 掩码、dropped/kept 都在掩码流上算；dropped 与 kept 合并成一遍（每 chunk 的行/颜色/帧号只读一次）。掩码流与 trim1 spill 的行完全相同、顺序相同。keep<1000 时走 identity 分支（全云），与旧代码行为一致。新增 `_read_int32_col`（掩码帧号临时文件用，跑完即删）。
  - 顺带修（同轮发现的真毛病）：trim 掏空云时 stage_fuse 干净失败（ok False，"no points after trim"），不再在 voxel_pick 百分位里 IndexError 崩掉。退化小任务实测：旧版崩溃，新版干净返回。
  - 没动：collect 循环、voxel_pick、百分位、参考实现（`_corroboration_trim`、`_packed_unique_inverse`、`_yardstick_voxel`、`_yardstick_voxel_spill`、`_chunked_unique`、`_spill_inverse`、`_percentile_1_99_1d`、`_percentile_1_99_cols`、`_Spill`、`_ColSpill` 源码与基底逐项相同）。
- 为什么：先量后做（量测笔记 `exp/trim_fuse62.md`）。stage_fuse 120 帧 32.5 秒里 trim 段占 7.27 秒，其中 trim1 写盘 ~3.5 秒、trim2 写盘 ~3.03 秒——两次整云写盘是 trim 段的大头。v0.6.54 轮次留的候选（只合 keep/dropped/kept 三遍）量完否决了：keep 重读只有 0.07 秒/120 帧，合并不值 1%。
- 实测（真任务，真代码）：
  - 120 帧：8 个产物文件全部逐字节相同（两组独立对照 + 暖缓存对照 + A/B 三轮交替）；新版平均 24.1 秒 vs 旧版 25.3 秒（A/B），约快 1.2 秒。页缓存抖动可达 ±13 秒，120 帧量级的省时在噪声边缘，REPORT 里如实写了。
  - 638 帧：8 个产物文件全部逐字节相同；276.5 → 175.8 秒（正向），反向重跑 202.9 vs 240.8 秒（新版先跑、冷缓存仍快 37.9 秒）。保守口径：长视频省约 38 秒（16%）。点数 2,679,654，两版一致。
  - 峰值内存：638 帧 1150 vs 1149 MB（正向）、1153 vs 1128 MB（反向），基本不变；120 帧新版峰值约高 80 MB（合并遍里同一 chunk 的掩码拷贝多握了一份，瞬时量），在限额内。
  - 小任务（2 帧 64×48，384 点，keep<1000 的 identity 分支）：两版逐字节相同。
- 验过：`tests/test_v62.py` 65/65（基底血统 + 补丁复现 + 参考实现未动 + `_read_int32_col` 往返 + 三轮量测等价性 + A/B 时序 + guard 实活 + 热更新模拟应用/中途回滚/中断回滚）；`verify_pack.py` 0 失败。包 508,899 B，sha256 `5e402021…2f54`。
- 没动：pipeline.py、depth_batch.py、voxel_export.py、web_app.py、voxels.py（冻结）、depth_video.py（冻结）、updater/hotupdate、web_ui。前端未动。
- 待用户拍板：v0.6.48–v0.6.62 发布、最大化验证、采集器 v2.1.2 重测、走路视频。
