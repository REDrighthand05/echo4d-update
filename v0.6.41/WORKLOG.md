# v0.6.41 工作记录

## 这一版改了什么（给用户看的）

- 修了一个启动时弹 .NET 报错、界面压成一长列的毛病：软件打开时不再
  报 .NET 错，界面恢复正常排版。
- 最大化不受影响：点最大化还是铺满工作区（任务栏留着），还原也正常。

## 技术记录

- 根因（2026-11-07 远程查桌面）：.NET `System.ArgumentOutOfRangeException`，
  事件 1026，2026-11-06 17:08:34；栈：`StringBuilder..ctor` ←
  `Control.get_WindowText()` ← `Control.set_CacheTextInternal` ←
  `Control.PerformLayout` ← `ControlCollection.Add` ←
  `Form.ControlCollection.Add`，经 `Python.Runtime.Dispatcher`。
  pywebview 6.2.1 的 WinForms 后端往窗体里加浏览器控件时，WinForms
  在布局里读半成品控件的窗口文字，StringBuilder 越界。2026-11-07
  09:37 又挂一次（AppHangB1， reopen 时的 .NET 框）。控件没建好，
  页面拿不到样式，于是所有元素堆成一列（不是 CSS 文件坏了，
  style.css / index.html / app.js 都完好）。
- 触发点：v0.6.36 的 `_ensure_maximize_styles` 在 `_polish_and_hook`
  里被急着调（显示、还原、定时器 1.5/4/10/30 秒），改窗口样式
  （SetWindowLongW + SWP_FRAMECHANGED）正撞上 WinForms 加控件做布局。
  （17:08 那次是 v0.6.31 的子窗口钩子，v0.6.33 已拆掉；09:37 这次是
  v0.6.36 新加的急着改样式。）
- 改动（只动 web_app.py + version.inc）：
  1. `_polish_and_hook` 不再急着设最大化样式；样式只在
     `_maximize_work_area` 里、调 ShowWindow(SW_MAXIMIZE) 之前设，
     窗口已经稳了才改。最大化行为不变。
  2. `_fill_maxinfo_from_work_area` 加防卫：工作区退化（宽/高 ≤ 0，
     虚拟/副屏可能报这种）就原样不动；最小跟踪尺寸钳住，永不大于
     最大尺寸（WinForms 的 MinimumSize 大于最大化尺寸时会抛
     ArgumentOutOfRange）。
- 没动：resize 钩子、DWM 修饰、桥、深度、体素、更新器、web_ui。
  没升 pywebview：它冻在 exe 里，热更新动不了；改触发点就够了。

## 验证

- 包：EchoScene-Update.zip 521,699 字节，
  sha256 3fc7b1a313406ecac9b4bffd05b06ee733ed64240695aa55d51d3d6b528e1bb3。
  链 0.6.36 → 0.6.41（from-manifest 是中转站 0.6.36 的目标清单；
  基底哈希已核对一致）。
- verify_pack.py：29 项全过，0 失败。
- tests/test_v41.py：打补丁后 10/10 过；同一套在 0.6.36 基底上
  正好失败 4 项（小屏上最小尺寸没钳住、退化工作区写了 0 宽/0 高、
  急着改样式还在）。基底复现毛病，补丁修掉毛病。
- web_app.py 能编译，纯 CRLF。

## 桌面应用结果

- （待填：中转站提交 + 桌面扳手应用 + 重启后版本号/界面确认）

## 状态

- 本地已验证；链：0.6.36 → 0.6.41。本地待命的 v0.6.37/0.6.38/
  0.6.39/0.6.40（冗余 + 回放）未发布，需先 rebase 到 0.6.41 上才能发。
