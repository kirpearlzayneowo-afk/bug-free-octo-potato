# AI Handoff：小动物画画监督日历

这是一份给 AI / Claude / 工程师看的项目说明，用于后续维护、改版、重构或扩展。

## 项目定位

这是一个本地优先的绘画复健打卡工具。

核心目标不是记录产量，而是帮助用户记录“重新靠近创作”的动作。

产品原则：

- 打开绘画软件就算一次有效开始。
- 不以完成作品为唯一目标。
- 降低开始的心理阻力。
- 允许低完成度也被记录和肯定。
- 数据默认只保存在浏览器本地。
- 不依赖后端。
- 不上传私人记录。
- 适合部署在 Netlify / GitHub Pages / 任意静态托管平台。

## 当前技术形态

当前项目是单文件 HTML：
txt
index.html

内部包含：

- HTML
- CSS
- 原生 JavaScript
- localStorage 数据持久化

没有使用框架。  
没有构建步骤。  
没有后端。  
没有数据库。

## 当前 localStorage key
js
animalDrawingCalendar.v3

旧版本兼容：
js
foxDrawingCalendar.v1
foxDrawingCalendar.v2

当前代码中包含 v1 / v2 到 v3 的迁移逻辑。

## 核心数据结构
ts
type AppData = {
version: 3;settings: Settings;days: Record<DateKey, DayEntry>;
};

type Settings = {
userName: string;charName: string;theme: string;animalIcon: string;animals: Animal[];moods: string[];
};

type Animal = {
icon: string;name: string;
};

type DateKey = string; // YYYY-MM-DD

type DayEntry = {types: string[];customWork: string;mood: string;reward: string;note: string;updatedAt: string;
};
## 完成度等级

当前完成度等级：
js
const levels = [
{ value: 0, label: "还没开始", done: false },{ value: 1, label: "只是打开", done: true },{ value: 2, label: "画了几笔", done: true },{ value: 3, label: "完成了简单涂鸦", done: true },{ value: 4, label: "细化了作品", done: true },{ value: 5, label: "完成了大作！", done: true }
];
判断是否打卡完成：
js
function isDone(entry) {
return Number(entry.level || 0) > 0;
}
也就是说，只要 level > 0，就算当天完成。

## 可替换称呼

用户可以在页面里设置：

- `userName`
- `charName`

默认值：
js
userName: "User"
charName: "Char"
这些值会用于“复制今日汇报”。

当前汇报模板逻辑在：
js
function makeShortReport(key = selectedDate)
当前输出格式类似：
txt
Char，User今天画画打卡完成🐾
6月23日｜完成度：只是打开
类型：爽画草稿
心情：平稳
想要：小青蛙夸夸
如果要国际化或自定义模板，可以优先改这里。

## 主题系统

主题定义在：
js
const themes = [...]
主题通过给 `body` 添加 class 实现。

例如：
js
body.theme-cream
body.theme-mint
body.theme-blue
body.theme-sakura
默认主题 `berry` 不添加额外 class，直接使用 `:root` 变量。

如果要新增主题：

1. 在 CSS 中新增 `body.theme-xxx`。
2. 在 `themes` 数组中添加：
js
{ id: "xxx", label: "主题名", className: "theme-xxx" }

## 监督员系统

默认监督员：
js
const defaultAnimals = [
{ icon:"🦊", name:"小狐狸" },{ icon:"🐱", name:"小猫" },{ icon:"🐰", name:"小兔" },{ icon:"🐟", name:"小鱼" },{ icon:"🐧", name:"小企鹅" },{ icon:"🐣", name:"小鸡" },{ icon:"🐸", name:"小青蛙" }
];
用户可以在页面中添加自定义监督员：

- emoji
- 名字

保存到：
js
data.settings.animals
data.settings.animalIcon
注意：有些 emoji 在不同系统显示不稳定。  
例如 `🐻‍❄️` 在部分 Windows 环境可能显示成熊 + 雪花，因此默认列表中不使用它。

## 心情系统

默认心情：
js
const defaultMoods = [
"平稳","开心","有点怕","焦虑","手感回来","累","想逃","被夸了","只想打开"
];
用户可以：

- 临时使用自定义心情
- 把自定义心情加入常用心情

常用心情保存在：
js
data.settings.moods

## 今日类型

当前今日类型：
js
const workTypes = [
"爽画草稿","练习","摸鱼","线稿","上色","临摹","OC","同人","设计","复健"
];
类型是多选，保存在：
js
entry.types: string[]

## 日历逻辑

日期 key 格式：
txt
YYYY-MM-DD

相关函数：
js
toKey(date)
parseKey(key)
dateLabel(key)
shortDateLabel(key)


日历按周一开始显示。

当前每月视图固定渲染 42 个格子。

## 统计逻辑

统计包括：

- 连续打卡天数
- 本月打卡数
- 总打卡数

连续打卡从今天开始往前数，遇到未完成日期停止。

相关函数：
js
countStats()

## 备份与导入

导出：
js
exportData

会下载完整 JSON 文件。

导入：
js
importFile.onchange
会读取 JSON，并通过 `normalize()` 规整后合并：
js
data = {
version: 3,settings: { ...data.settings, ...imported.settings },days: { ...data.days, ...imported.days }
};
注意：当前导入策略是“导入数据覆盖同日期旧数据”。  
如果未来需要冲突处理，可以在这里改。

## 重要产品文案

底部说明当前为：txt
这是一个本地优先的绘画复健打卡工具。
它记录的不是产量，而是你是否重新靠近创作。

数据默认保存在浏览器 localStorage 中，不会上传服务器。
你可以复制今日汇报分享给陪伴者，也可以导出 JSON 作为备份。
建议保留这段，因为它准确表达了产品定位。

## 后续可扩展方向

可选增强：

- 自定义完成度等级
- 自定义今日类型
- 自定义汇报模板
- 多语言支持
- PWA 安装
- 导出 PNG / 分享卡片
- GitHub Pages 部署说明
- 更完整的设置面板
- 数据加密备份
- WebDAV / iCloud Drive / GitHub Gist 同步
- Notion / Google Sheet 同步
- 多设备同步
- 移动端布局进一步优化

## 修改建议

如果后续由 AI 继续修改，请注意：

1. 不要移除 localStorage 本地优先原则。
2. 不要默认上传用户数据。
3. 不要把低完成度视为失败。
4. “只是打开”也必须算完成。
5. 保留 JSON 导入 / 导出。
6. 保留 User / Char 称呼可配置。
7. 保持单文件可直接打开，除非用户明确要求框架化。
8. 修改代码时优先给完整 `index.html`，方便非技术用户直接替换。