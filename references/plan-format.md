# plan.js 数据格式契约

`plan.js` 是整个应用唯一需要个人定制的文件。应用壳 `index.html` 在主脚本之前加载它，从全局常量读取所有计划内容。本文档是编写/生成 plan.js 的完整契约——**AI 生成计划时必须遵守本契约，生成后必须通过 `node scripts/validate-plan.js` 校验**。

## 必须定义的全局常量

| 常量 | 类型 | 用途 |
|------|------|------|
| `PLAN_DATA` | object | 每日模板：`morningTemplate`（晨间矫正，每天）+ `warmupTemplate`（训练前激活，训练日） |
| `WEEKS` | array | 周计划主体，每元素一周 |
| `HABITS` | object | 微习惯打卡项：`toggle`（勾选）+ `counter`（计数） |
| `METRICS` | array | 体测指标定义 |
| `NOTE_TEMPLATES` | array | 动作笔记的快捷模板 |
| `VIDEO_MAP` | object | 动作名 → 视频搜索词/BV 号映射（可为空 `{}`） |
| `getVideoForExercise` | function | 动作名 → `{q, bv?, note?}`，未匹配返回 `{q: 名字+' 教学'}`（照抄示例实现即可） |
| `EXERCISE_NAME_ALIASES` | array | 动作名别名组（新计划留空 `[]`；换计划时把新旧名字放同组以共享历史记录） |

## 动作对象（exercise）

出现在模板和训练日 groups 里的最小单元：

```js
{ id: 'w1-d1-1', name: '死悬', sets: '3 组 × 15 秒', weight: '自重', tip: '肩胛主动下沉' }
```

| 字段 | 必需 | 说明 |
|------|------|------|
| `id` | ✅ | **全计划唯一**（含模板），推荐 `w{周}-d{天}-{序}` / `mr-{序}` / `wu-{序}`。打卡记录与评论按 id 关联，重复 id 会导致数据错绑 |
| `name` | ✅ | 动作名。同一动作跨周请保持命名一致（评论按名字跨周聚合） |
| `sets` | ✅ | 组次描述字符串，如 `'3 组 × 12 次'`、`'60 秒'`、`'2 组 × 15 次/侧'` |
| `weight` | 可选 | 负荷描述，如 `'40 kg'`、`'自重'`、`'弹力带'` |
| `tip` | 可选 | 一句话动作要点/发力提示，强烈建议提供 |

## PLAN_DATA

```js
const PLAN_DATA = {
  morningTemplate: { name, duration, description, exercises: [/* 动作对象 */] },
  warmupTemplate:  { name, duration, description, exercises: [/* 动作对象 */] }
};
```

两个模板都必需。`morningTemplate` 每天显示（含休息日）；`warmupTemplate` 只在训练日显示。

## WEEKS

```js
{
  weekNum: 1,                     // 整数,必须逐周 +1(可从任意数字开始,不必是 1)
  phase: 'Phase 0 - 基础唤醒期 (Week 1/4 · 🌱 感知)',  // 阶段名,建议带进度和主题
  note: '本周训练要点……',          // 可选但强烈建议:本周重点、警示、目标
  days: [ /* 恰好 7 个,周一到周日顺序固定 */ ]
}
```

### day 对象

| dayType | 必需字段 | 说明 |
|---------|----------|------|
| `'training'` | `dayName, dayType, title, groups` | 训练日。`groups` 为非空数组：`{title, description, exercises: [...]}`，可多组（如「主训练」+「测试」） |
| `'recovery'` | `dayName, dayType, title, description` | 主动恢复日（散步/拉伸等） |
| `'rest'` | `dayName, dayType, title, description` | 完全休息日 |

`dayName` 必须依次为 `'周一'`…`'周日'`。`title` 建议格式 `'Week N 周X · 主题'`。

### 周次与日期的关系

应用以设置里的 `startDate`（计划第一周的周一）为基准：`当前周 = WEEKS[0].weekNum + floor((今天 - startDate) / 7天)`。首次启动自动把本周一设为 startDate。**应用壳内不允许硬编码周次**。

## HABITS

```js
const HABITS = {
  toggle:  [{ id: 'h-morning', name: '每日矫正', icon: '🦴' }, ...],
  counter: [{ id: 'h-water', name: '饮水', target: 8, unit: '杯' }, ...]
};
```

id 全局唯一。**`h-morning` 是应用联动锚点**：晨间矫正动作全部完成时自动勾选，建议始终保留该 id。toggle 建议 4-8 个，counter 0-2 个。

## METRICS

```js
{ id: 'm-weight', name: '体重', unit: 'kg', tip: '晨起空腹便后' }
```

id 唯一，`tip` 描述测量条件。建议 5-10 项，且与计划呼应（练什么测什么，左右分开测能暴露不对称）。

## 硬约束（校验器强制）

1. 所有必需常量存在且类型正确
2. 动作 id 全计划唯一（含模板）
3. 每周恰好 7 天，`dayName` 周一→周日顺序固定
4. `weekNum` 逐周 +1 连续
5. `dayType` ∈ `training` / `recovery` / `rest`
6. 训练日必有非空 `groups`，每组非空 `exercises`
7. 习惯/体测 id 各自唯一

## 软约束（生成质量指南）

- **渐进超负荷**：相邻周同一动作小幅递增（次数 +1~2、时长 +5s、或重量 +2.5~5kg），不做跳跃式增长
- **周期化**：4 周以上的计划建议每 4 周安排 1 个减量周（量降 30-40%）；按阶段划分主题（感知 → 巩固 → 渐进 → 目标）
- **恢复结构**：每周训练日 2-5 天，训练日之间尽量隔开；周日建议完全休息
- **note 要具体**：写本周的训练重点和自查要点，不写空话
- **测试锚点**：定期安排力竭测试日，与 METRICS 项对应，形成可追踪基线
- **安全**：涉及疼痛部位的动作要在 tip 中写明停止条件；宁可保守
