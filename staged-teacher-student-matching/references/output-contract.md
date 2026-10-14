# Output Contract

## Stage 1 Outputs

Create these workbooks in the Stage 1 output folder when their tables are non-empty:

1. `第一阶段_匹配成功师生表.xlsx`
2. `第一阶段_未匹配到学生的老师表.xlsx`
3. `第一阶段_学生未匹配科目表.xlsx`
4. `第一阶段_有余量老师表.xlsx`

Suggested columns:

- Matched table: `序号`, `学生姓名`, `年级`, `学生薄弱科目`, `匹配老师`, `本阶段匹配科目`, `匹配类型`.
- Unmatched teachers: `序号`, `老师姓名`, `可教年级`, `可教科目`, `最多可接收人数`, `说明`.
- Student missing subjects: `序号`, `学生姓名`, `年级`, `未匹配科目`, `说明`.
- Remaining teachers: `序号`, `老师姓名`, `可教年级`, `可教科目`, `最多可接收人数`, `已带学生数`, `剩余余量`.

## Stage 2 Outputs

Create these workbooks in the Stage 2 output folder when their tables are non-empty:

1. `第二阶段_匹配成功师生表.xlsx`
2. `第二阶段_学生未匹配科目表.xlsx`
3. `第二阶段_有余量老师表.xlsx`

Use the same matching columns as Stage 1. In the remaining-teacher table, include only teachers with `remaining > 0`, where remaining is the declared maximum minus the final unique student count.

If a Stage 1 or Stage 2 output table is empty, do not create that workbook. Report the empty table in the final response instead.

## Final Core Workbook

Create `最终_核心结果.xlsx` with exactly two worksheets.

### `学生安排`

Columns:

1. `学生名称`
2. `年级`
3. `匹配到的老师`

Format multiple teachers with a semicolon:

```text
Teacher A（Mathematics）；Teacher B（English）
```

### `老师安排`

Columns:

1. `老师姓名`
2. `对应学生姓名`

Format multiple students with a semicolon. Do not repeat a student for the same teacher.

## Final Shortage Workbook

Create `最终_缺量表.xlsx` with exactly four worksheets, even if a table is empty:

1. `未匹配到老师的学生`
2. `未匹配到学生的老师`
3. `老师余量表`
4. `学生未匹配到的科目`

### Empty Sheets

Keep the headers and put `当前无记录` in the first data cell.

### Teacher Remaining Sheet

Columns:

1. `老师姓名`
2. `最多可接收学生数`
3. `已带学生数`
4. `剩余余量`

Use `declared maximum - final unique student count`. This sheet is the handoff for later manual adjustment; the skill does not perform the adjustment.

## Formatting and Hygiene

- Use clear header styling, legible column widths, and frozen header rows.
- Do not add extra explanation sheets unless the user asks.
- Remove temporary scripts, preview images, PDFs, and inspection files before final delivery.
- Never overwrite original raw workbooks.
