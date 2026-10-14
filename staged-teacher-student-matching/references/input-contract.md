# Input Contract

## Teacher Table

The teacher table is a seven-column worksheet.

| Column | Required Field | Meaning |
|---|---|---|
| A | `1、姓名` | Teacher name; unique teacher identity for deduplication and matching. |
| B | `2、学号` | Teacher student ID; used to detect and correct swapped name and ID fields. |
| C | `3、可教科目（可多选）` | Subjects the teacher can teach. |
| D | `4、可教年级（可多选）` | Grades or grade bands the teacher can teach. |
| E | `5、上期所教学生姓名及科目（之前没带过就写无）` | Previous students and the single subject taught to each. |
| F | `6、是否愿意继续匹配该学生` | `是` permits strict continuation matching; all other values normalize to `否`. |
| G | `7、最多愿意接受学生数量` | Declared maximum unique students, no more than 3. Stage 2 still caps assignments at 2. |

### Teacher Column E Format

Each semicolon-separated item contains exactly one student and one subject:

```text
Student（Subject）
```

Multiple students:

```text
Student A（Mathematics）；Student B（English）
```

Rules:

- No item may contain two subjects for the same student.
- A student with two subjects must appear under two different teacher records.
- `无` means the teacher did not teach anyone in the previous period.
- Do not use `无科目记录`.
- Do not create or require a separate history-subject column.

## Student Table

The student table is a five-column worksheet.

| Column | Required Field | Meaning |
|---|---|---|
| A | `1、孩子姓名` | Student name; unique student identity for deduplication and matching. |
| B | `2、孩子目前年级` | Current grade. |
| C | `3、孩子薄弱科目（最多选两门）` | Subject needs, at most two subjects. |
| D | `4、之前给孩子上课的老师是谁（填写老师姓名全名）` | Previous teacher information, normalized to `姓氏＋老师`; this field does not identify the previous subject. |
| E | `5、是否愿意该老师继续教孩子` | `是` permits strict continuation matching; all other values normalize to `否`. |

## Grade Mapping

- Primary grades one through six match `小学`.
- `初一`, `初二`, and `初三` match `初中`.
- `高一`, `高二`, and `高三` match `高中`.

## Input Gate

If the workbook structure, required field names, teacher column E format, or student subject count does not match this contract, stop and report the exact problem. Do not silently infer a legacy structure.
