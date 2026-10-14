---
name: staged-teacher-student-matching
description: Execute or reproduce the 厚生书屋 Excel data-cleaning and two-stage teacher-student scheduling workflow, including exact subject continuation, capacity limits, and final workbook outputs. Use when the user asks to clean, match, schedule, or output teacher-student assignments from the project's teacher and student workbooks; do not use for generic calendar or room scheduling.
metadata:
  short-description: Clean and match teachers and students in two stages
---

# Staged Teacher Student Matching

## Outcome

Produce the requested cleaned and staged scheduling workbooks from the project's teacher and student tables. The workflow stops at workbook output. It does not perform manual adjustment of unused capacity.

## Required Reading

Read [references/input-contract.md](references/input-contract.md) before reading or modifying the source workbooks.

Read [references/matching-rules.md](references/matching-rules.md) before calculating any match.

Read [references/output-contract.md](references/output-contract.md) before creating or overwriting deliverables.

## Non-Negotiable Rules

- Treat the teacher table as a seven-column input.
- Teacher column E is `5、上期所教学生姓名及科目（之前没带过就写无）` and already follows `Student（Subject）` format. Do not create a separate history column or describe the process as merging columns.
- Each semicolon-separated item represents exactly one student and one subject.
- A teacher cannot teach the same student two subjects. A student's two subjects must be assigned to two different teachers.
- Continue-teaching priority requires the same student, the same teacher, and the same historical subject, with both willingness fields equal to `是`.
- Teacher capacity counts unique students, not subjects.
- By the end of Stage 2, assign at most two students to each teacher even when the teacher's declared maximum is three. Output the unused part of the declared maximum as manual adjustment capacity.
- Do not add historical teachers or historical students that are absent from the required input contract.
- Do not perform manual reassignment. Generate the matching, shortage, and remaining-capacity tables only.

## Workflow

1. Validate both input tables against the input contract. Stop and report the exact mismatch if the contract is not met.
2. Clean duplicate names, name and student-ID issues, willingness values, grades, subjects, and teacher history strings according to the matching rules.
3. Build eligible student-subject-to-teacher candidates.
4. Solve Stage 1 with exact continuation priority and the hard constraints.
5. Solve Stage 2 in two passes: first give every zero-match teacher at most one student, then use remaining allowed capacity.
6. Validate that no student lacks a teacher, no teacher exceeds two matched students by the end of Stage 2, and no teacher repeats for the same student's subjects.
7. Export only the output tables specified by the output contract. Intermediate empty tables are omitted; the final shortage workbook keeps its four sheets.

## Verification

- Check exact continuation triples against the teacher table history field.
- Check unique-student capacity, not subject-count capacity.
- Check for teacher-subject duplication within each student.
- Check the declared teacher maximum and the Stage 2 cap separately.
- Check that the final remaining amount equals declared maximum minus unique matched students.
- Remove temporary scripts, previews, and inspection files after delivery.

## References

- Input fields and formatting: [references/input-contract.md](references/input-contract.md)
- Matching, continuation, and capacity rules: [references/matching-rules.md](references/matching-rules.md)
- Required workbooks and worksheets: [references/output-contract.md](references/output-contract.md)
