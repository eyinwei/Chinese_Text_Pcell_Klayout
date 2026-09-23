# Chinese_Text_Pcell — KLayout 中文文字 PCell

一个单文件 KLayout 宏（`Chinese_Text_Pcell.lym`），提供把**中文文本转成多边形**的 PCell（`ChineseText` 库 / `ChineseText` PCell）。字形轮廓由 KLayout 自带的 Qt 绑定（`QPainterPath.addText`）从系统字体实时提取，无需任何外部字库文件。

> 背景：KLayout 内置的 TEXT PCell 基于编译进程序的 GDS 字形库（`db::TextGenerator`），只支持 ASCII 32–127，无法直接排中文。本宏沿用其 PCell 接口设计，把字形来源换成系统 TrueType/OpenType 字体。

## 安装

把 `Chinese_Text_Pcell.lym` 放入 KLayout 宏目录（Windows: `%USERPROFILE%\KLayout\pymacros`），重启 KLayout（或在宏编辑器中 Reload）。

## 用法

- **PCell**：`Edit > Instance`（快捷键 `I`）→ 库选 `ChineseText` → Cell 选 `ChineseText`；或脚本 `layout.create_cell("ChineseText", "ChineseText", {"text": "中文", "char_height": 10.0})`。
- 参数对话框里直接输入或粘贴中文，`\n` 换行。

## 主要参数

| 参数 | 说明 |
|---|---|
| `text` | 文本内容（`\n` 换行） |
| `font_name` | 系统字体名，如 `SimHei` / `SimSun` / `Microsoft YaHei` |
| `font_size` | 字号（µm，以 1 em 计） |
| `char_spacing` / `line_spacing` | 字间距（µm）/ 行距（µm，附加量） |
| `bold` / `italic` | 粗体 / 斜体 |
| `layer` | 输出图层 |

**原点**：第一行文字基线的左端（x=0 为首字符起点，y=0 为基线，字形主体在 +y）。

## 实现要点

- `QPainterPath.addText` 逐字符提取轮廓 → `toSubpathPolygons` 取子路径 → `pya.Region` 逐环 XOR（偶奇）合成，消解重叠并把内孔正确识别为孔 → 带孔多边形沿孔的 y 中线递归切分为**无孔简单多边形**（GDS 不支持孔，这样导出才干净）。
- 直接使用 `toFillPolygons` 会产生外轮廓与内孔以零宽缝相连的"钥匙孔"轮廓，渲染出多余斜线——这是本实现改走布尔合成的原因。
- 按字符缓存字形（字体+粗斜体+字符为键），重复字符不重复计算。
- 字号用 `QFont.setPixelSize` 定义 em，使路径坐标与系统 DPI 无关。

## 限制

- 仅 GUI 模式可用（依赖 Qt 绑定），`klayout -z` 纯命令行下不可用。
- 字形随系统字体而变；用于流片交付前建议展平（flatten）固化几何，并确认目标机器字体一致。
- 参数对话框如无法调出输入法打中文，可在系统其他地方打好后 Ctrl+V 粘贴。

## 环境

KLayout 0.28+（pymod Python 绑定），Windows / macOS / Linux 均可（只要有 Qt 与系统中文字体）。

## 致谢

字形提取思路参考 KLayout 作者 Matthias Koefferlein 在官方论坛关于 `QPainterPath::addText` + 多边形合成的答复，以及 KLayout 内置 Basic TEXT PCell（`libBasicText.cc`）的接口设计。
