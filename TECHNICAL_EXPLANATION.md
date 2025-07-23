# 技术性解释/说明

[TOC]

## 免责声明
本文仅描述思路与大体做法，仅作学习与交流。**我们不鼓励解包，以下仅是做必要的技术披露。**

如果需要具体的解析方法和代码，请自行摸索或直接使用本项目网站上的现成、处理后的数据。

本文档**不是教程**，但描述的内容可证明项目数据处理流程的真实性与有效性。

## 准备事宜
- [AssetRipper](https://github.com/AssetRipper/AssetRipper)
  - Asset Studio GUI 大概也行，不过问题是它已经很久没更新了，所以ummmm可能解不出来一些东西
- [TypeScript](https://www.typescriptlang.org/)
  - 或者随便一个你喜欢的编程语言都行；虽然我个人确实是 Python 比较熟，但是果然还是太馋 TS 的强类型检查了 ~~（Python的类型注解还是太吃操作了）~~

## 一、解包游戏
AssetRipper 启动！然后以“Export as a Unity project”形式导出。

**仅此而已。**

## 二、分析数据
实际在摸索文件的时候其实分了好几次去摸（因为不知道自己要摸什么，只有在遇到疑难杂症的时候才会想着再去翻一下文件碰碰运气），这里为求行文简单只写最后摸出来的东西和一点心得。

### 本地化文件
位于 `AssetRipper_export_20250723_031405\ExportedProject\Assets\TextAsset` 下

目前的最新版本（[Build ID](https://steamdb.info/patchnotes/19035344/)、[Manifest ID](https://steamdb.info/depot/2704111/history/?changeid=M:6695761162722337488)）文件名为 `localization_2025_4_23.txt` ，使用 CSV 格式，是典型的键值对查询。

本站在处理为前端数据时将其转为 JSON Record/Dict/Map[^1] 进行分发。

[^1]: 取决于你学的编程语言，这种 key-value 形式可能叫以上的任何名字，或都不是。

### 流程图 prefab 文件
位于 `AssetRipper_export_20250723_031405\ExportedProject\Assets\Resources\myprefabs\flowchart` 下

目前本站只涵盖了已发布的主线剧情，对未发布的剧情未进行公开。

Asset Ripper 的解包结果中包含 `.meta` 和 `.prefab` 文件，其中 `.prefab` 文件为一多 YAML 文件（一个文件中包含多个 YAML 文档），使用如 `--- !u!1 &1145141919810000` 的方式分隔；诸如 `1145141919810000` 的 ID 为 Fungus 内部追踪各“组件”（命令块）时使用的全局 ID ，而每一个 YAML 文档都代表着一个 Fungus 的“命令块”。

### Fungus 命令
位于 `AssetRipper_export_20250723_031405\ExportedProject\Assets\Scripts\Assembly-CSharp` 下，但主要的自定义命令都在该目录的 `CustomCommand` 文件夹下。

主要是 `.cs` 及其相关的 `.meta` 文件。

在分析这些自定义命令的代码与各个“命令块”之间的关系时，我们注意到如：

```yaml
--- !u!114 &1145141919810000
MonoBehaviour:
......
  m_Script: {fileID: 11500000, guid: 791d1b40be4a7bc888a1e77697a710c2, type: 3}
......
```

其中的

```yaml
  m_Script: {fileID: 11500000, guid: 791d1b40be4a7bc888a1e77697a710c2, type: 3}
```

之中的“guid”与这些自定义命令的 `.meta` 文件：

```yaml
......
guid: 791d1b40be4a7bc888a1e77697a710c2
......
```

中的“guid”一致。即我们发现这些命令块都与这些自定义命令有以上关联。通过这种关系，我们建立了 `guid -> .cs文件` 关系表并将其应用于脚本处理中。

### 图片
位于 `AssetRipper_export_20250723_031405\ExportedProject\Assets\Resources\images` 下，全部为 `.png` 格式。

### BGM（不计音效）
位于 `AssetRipper_export_20250723_031405\ExportedProject\Assets\Resources\music` 下，全部为 `.ogg` 格式。

## 三、处理数据
主要涉及以下流程：

- YAML 预处理
  - 主要是将所有数字都转为 string （使用正则匹配所有数字并将其用双引号包裹起来），因为 JS 数字过大损失后几位精度（恼，差点栽这了）
- TS 读取所有 YAML 文档、去除与规范化数据（主要是类型转换），将其保存为 JSON 备用
- TS 读取上一步的 JSON 文档，处理流程图数据
  - 预处理部分命令块数据
  - Per `GroupBlock` basis 把各个命令块给连接起来（建立边缘（edge）关系）
  - 把流程图的开头和第一个 `GroupBlock` 连起来
- 导出为 Vue Flow 专用数据格式

本地化数据较为简单不做叙述。图片和BGM因未做额外处理不另行叙述。

## 四、其它
除了网站之外，其它所有数据处理流程都是在我自己的个人计算机上完成。


