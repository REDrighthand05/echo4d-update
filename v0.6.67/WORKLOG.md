# WORKLOG — v0.6.67（体素 GLB 改用内置极简写码器）

- 版本：v0.6.67（链 0.6.66 → 0.6.67）。本地热更新包，未发布，未推中转站，未碰桌面。
- 时间：2026-11-10 12:40–13:30（CST）。
- 改了什么：`app/echo_scene/voxels.py` + `app/installer/version.inc`（改号）。
  - 体素段写 voxel_model.glb 不再走 open3d：懒导入 `from .voxel_export import write_glb`（voxel_export 在顶层 import 本模块，顶层 import 会循环），再 `write_glb(candidate, verts, vcols, tris)`。同一批顶点/颜色/三角面，写成 POSITION float32 + COLOR_0 uint8 normalized + uint32 索引、真正的 BIN 块——就是导出按钮写 voxel_color.glb 用的那个写码器。
  - 失败仍走原有的 except → glb_path=None，任务不死。独立加载（无包）时相对导入失败，同样退化到 glb_path=None、任务照常完成，测试里钉死了这个行为。
  - 没动：冻结的体素模型（build_voxel_mesh、write_ply、FROZEN docstring）源码逐项相同；diff 恰好一个 hunk；深度、融合、导出、前端、更新器都没动。
- 为什么：先量后做（0.6.63 留的数：GLB 分支 120 帧 12.16/13.77 秒、725 MB、峰值 5.08 GB；638 帧本机 OOM）。量测还看清了旧文件的真结构：open3d 的 GLB 没有 BIN 块，buffer 以 base64 嵌在 JSON 里（文件因此大 4/3），颜色是 float32，还带算出来的平滑法线。
- 实测（真任务，真代码）：
  - 小云旧 vs 新（1,000 个真实点）：位置逐字节相同、颜色逐值相同、索引逐值相同、包围盒相同；trimesh 与 Blender 4.2.3 两边都认（8,000 顶点 / 12,000 面、带顶点色）。文件 481,204 → 264,996 字节。唯一差别是旧文件有平滑法线（新文件没有，和 PLY 一样，读者自己算）。
  - 120 帧：写 GLB 12.16/13.77 秒 → 2.24 秒（暖轮；首轮冷 7.93 秒），峰值 5.08 GB → 1.19 GB，文件 725 MB → 332 MB，两轮 sha 相同。
  - 638 帧：旧分支本机从未完成（OOM）；新分支 6.87–9.75 秒、峰值 2.45 GB、文件 707 MB，两轮 sha 相同——638 帧的体素 GLB 第一次在本机产出来了。
  - 发版文件活验：src 的 stage_voxel_model 在 120 帧与 638 帧新任务副本上完整跑一遍，PLY 与 0.6.63 基线逐字节相同，GLB 与 A/B 新分支逐字节相同；整段 120 帧 5.18 秒、638 帧 10.73 秒。
- 验过：`tests/test_v67.py` 69/69（基底血统 + 补丁复现 + 冻结函数源码相同 + 单 hunk + 小云旧新语义相等 + 全尺寸两轮确定性 + 计时/峰值/文件大小对旧基线 + 全尺寸解析验证 + Blender + 发版文件活验 + 边缘（缺 points.bin、空云、1 点云、独立加载退化）+ 热更新模拟应用/中途回滚/中断回滚）；`verify_pack.py` 0 失败。包 493,853 B，sha256 `b8f2154c…4464de9`。
- 没动：pipeline.py、depth_batch.py、fuse.py、voxel_export.py（write_glb 原样复用）、web_app.py、depth_video.py（冻结）、updater/hotupdate、web_ui。前端未动。深度未动（已量到顶）。
- 待用户拍板：v0.6.48–v0.6.67 发布、最大化验证、采集器 v2.1.2 重测、走路视频；体素 GLB（voxel_model.glb）留不留；深度预览 PNG 删不留。本包只让这个文件更便宜，不改变“留不留”的问题——如果不要，整段 GLB 工作可以整段省掉。
- 量测里的自我纠正：比较脚本第一版把归一化后的颜色直接当 uint8 比，报最大差 204（脚本的毛病，不是文件的问题）；解析器第一版假设 GLB 一定有 BIN 块，open3d 的文件没有，补了 base64 分支才看清结构；638 帧全量验证第一版全量握持被 OOM 杀掉，改成分块 memmap 比对；本机 open3d 开局缺 libEGL.so.1（VM 重启后系统包丢了），.deb 直装 libegl1 + libegl-mesa0 修好，apt-get update 跑了 16 分钟无果且被 OOM 杀过一次，不走 apt。
