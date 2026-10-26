# 厚生书屋排课 Skill

`staged-teacher-student-matching` 是一个面向 Codex 的排课 Skill，用于完成厚生书屋项目的老师、学生数据清洗，严格执行续教优先规则，并按照两个阶段输出师生匹配表和缺量表。

这个 Skill 只负责自动计算并生成要求的工作簿，不负责后续人工调整。老师报名表总上限与阶段二实际上限分开保留，剩余的灵活额度会输出到老师余量表，供人工后续处理。

## 功能范围

- 读取并校验老师和学生报名表。
- 按姓名去重，保留最新记录。
- 检查老师姓名和学号是否颠倒。
- 清洗老师上期学生及科目。
- 清洗学生薄弱科目和上期老师信息。
- 严格续教优先匹配：
  - 同一名学生；
  - 同一位老师；
  - 与上期相同的科目；
  - 双方均愿意继续匹配。
- 第一阶段先完成全部严格续教匹配；同一学生可在第一阶段同时匹配两位历史老师。
- 尚未获得续教匹配的学生，再在阶段一补一门普通匹配，保证每人至少一位老师。
- 第二阶段优先补足完全未匹配老师，再使用剩余容量。
- 同一名老师不能负责同一名学生的两门科目。
- 同一名老师在本期只能负责一个具体年级和一个科目；带两名学生时也必须同年级、同科目。
- 同一名学生的不同科目必须由不同老师负责。
- 到第二阶段结束，每位老师最多匹配 2 名学生。
- 输出核心结果和缺量表。

## 不包含的内容

- 不自动使用老师剩余容量进行人工调整。
- 不自动决定最后一轮人工分配。
- 不修改原始报名表。

## 仓库结构

```text
staged-teacher-student-matching/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── input-contract.md
    ├── matching-rules.md
    └── output-contract.md
```

- `SKILL.md`：Skill 主流程和核心约束。
- `references/input-contract.md`：老师表、学生表字段和格式要求。
- `references/matching-rules.md`：数据清洗、续教规则、两阶段匹配和容量规则。
- `references/output-contract.md`：工作簿名称、工作表和输出格式要求。

## 安装方法

### Windows PowerShell

先关闭正在运行的 Codex，然后执行：

```powershell
$repo = "$env:USERPROFILE\.codex\skill-repos\staged-teacher-student-matching-skill"
$dest = "$env:USERPROFILE\.codex\skills\staged-teacher-student-matching"
git clone https://github.com/sushu349/staged-teacher-student-matching-skill.git $repo
New-Item -ItemType Directory -Path $dest -Force | Out-Null
Copy-Item -LiteralPath "$repo\staged-teacher-student-matching\*" -Destination $dest -Recurse -Force
```

安装后目录应类似：

```text
C:\Users\<用户名>\.codex\skills\staged-teacher-student-matching\
```

### macOS 或 Linux

```bash
git clone https://github.com/sushu349/staged-teacher-student-matching-skill.git ~/.codex/skill-repos/staged-teacher-student-matching-skill
mkdir -p ~/.codex/skills
cp -R ~/.codex/skill-repos/staged-teacher-student-matching-skill/staged-teacher-student-matching ~/.codex/skills/
```

### 手动安装

下载本仓库，并将整个文件夹放入：

```text
~/.codex/skills/staged-teacher-student-matching
```

Windows 对应路径为：

```text
C:\Users\<用户名>\.codex\skills\staged-teacher-student-matching
```

确认目录中直接存在 `SKILL.md`。不要在 `skills` 目录下多套一层文件夹。

## 调用指令

在 Codex 中使用：

```text
$staged-teacher-student-matching
```

也可以直接用自然语言说明任务，例如：

```text
使用 $staged-teacher-student-matching，读取老师表和学生表，完成数据清洗、第一阶段匹配和第二阶段匹配，并输出核心结果表和缺量表。
```

```text
使用 $staged-teacher-student-matching，按照当前规则重新排课。到第二阶段结束时，每位老师最多带 2 名学生，剩余名额输出到老师余量表。
```

## 必需输入

### 老师表

老师表输入时固定为 7 列：

| 列 | 字段 |
|---|---|
| A | `1、姓名` |
| B | `2、学号` |
| C | `3、可教科目（可多选）` |
| D | `4、可教年级（可多选）` |
| E | `5、上期所教学生姓名及科目（之前没带过就写无）` |
| F | `6、是否愿意继续匹配该学生` |
| G | `7、最多愿意接受学生数量` |

老师表第 E 列格式：

```text
学生姓名（一个科目）
```

多名学生示例：

```text
学生甲（数学）；学生乙（英语）
```

约束：

- 一个分号项只能对应一名学生和一个科目。
- 同一名老师、同一名学生只能有一个科目。
- 不允许写成 `学生甲（数学、英语）`。
- 同一名学生有两门科目时，必须分别出现在两位不同老师的记录中。
- 不出现“无科目记录”。
- 没有带过学生时填写“无”。

### 学生表

学生表输入时固定为 5 列：

| 列 | 字段 |
|---|---|
| A | `1、孩子姓名` |
| B | `2、孩子目前年级` |
| C | `3、孩子薄弱科目（最多选两门）` |
| D | `4、之前给孩子上课的老师是谁（填写老师姓名全名）` |
| E | `5、是否愿意该老师继续教孩子` |

学生表不增加其他历史科目列。学生需要匹配的科目以第 C 列为准。

## 输出内容

### 第一阶段输出

根据实际结果生成非空表格：

- `第一阶段_匹配成功师生表.xlsx`
- `第一阶段_未匹配到学生的老师表.xlsx`
- `第一阶段_学生未匹配科目表.xlsx`
- `第一阶段_有余量老师表.xlsx`

### 第二阶段输出

根据实际结果生成非空表格：

- `第二阶段_匹配成功师生表.xlsx`
- `第二阶段_学生未匹配科目表.xlsx`
- `第二阶段_有余量老师表.xlsx`

### 最终输出

1. `最终_核心结果.xlsx`

包含两张工作表：

- `学生安排`
  - 学生名称
  - 年级
  - 匹配到的老师
  - 多名老师之间使用分号，老师后用括号标注科目
- `老师安排`
  - 老师姓名
  - 对应学生姓名
  - 多名学生之间使用分号

2. `最终_缺量表.xlsx`

固定包含四张工作表：

- `未匹配到老师的学生`
- `未匹配到学生的老师`
- `老师余量表`
- `学生未匹配到的科目`

老师余量计算公式：

```text
剩余余量 = 老师报名表总上限 - 最终实际已带学生数
```

## 核心硬约束

1. 老师可教年级必须包含学生所在学段。
2. 老师可教科目必须包含当前要匹配的科目。
3. 老师容量按唯一学生人数计算，不按科目数量计算。
4. 同一老师不能负责同一名学生的两门科目。
5. 同一名学生的不同科目必须由不同老师负责。
6. 每名学生至少匹配到一位老师。
7. 严格续教优先必须满足同一学生、同一老师、同一科目。
8. 到第二阶段结束，每位老师最多带 2 名学生。
9. 老师总上限最高为 3，超过 2 人的部分保留为人工调整额度。

## 年级顺序

```text
高三
初三
六年级
高二
高一
初二
初一
五年级
四年级
三年级
二年级
一年级
```

## 使用注意

- 输入模板不符合规范时，Skill 应停止并报告具体错误，不应自行猜测。
- 处理完成后，应删除临时脚本、预览图和检查文件，只保留正式输出。
- 如果老师表第 E 列存在同一老师对应同一学生多个科目，属于无效历史记录，应先修正源数据。
- 如果当前环境中安装了 `openpyxl`、`scipy`、`pandas`、`python-docx` 等依赖，可以更顺畅地完成 Excel 和 Word 产物处理。

## 适用场景

适用于：

- 厚生书屋数据清洗；
- 师生续教优先匹配；
- 两阶段排课；
- 缺量统计；
- 老师剩余容量整理；
- 按同一规则重复执行下一期排课。

不适用于：

- 普通日历排班；
- 教室安排；
- 与老师、学生、科目无关的匹配问题。
