# Matching Rules and Algorithm

## Data Cleaning Rules

1. Deduplicate teacher and student records by name, retaining the latest occurrence.
2. Correct swapped teacher name and student ID fields after trimming.
3. Parse teacher column E into `(student, subject)` pairs and validate that each teacher-student pair occurs exactly once.
4. Normalize teacher willingness: retain `是`; convert every other value to `否`.
5. Clamp teacher declared maximum to 3. The declared maximum remains the manual-adjustment ceiling.
6. Normalize student previous-teacher text to `姓氏＋老师`; convert unknown text to `无`.
7. Keep at most two student weak subjects.
8. Sort students by the project's grade order.

## Grade Order

```text
高三 → 初三 → 六年级 → 高二 → 高一 → 初二 → 初一 → 五年级 → 四年级 → 三年级 → 二年级 → 一年级
```

Within a grade, preserve the student table's source order unless a later tie-break rule requires otherwise.

## Hard Constraints

Every assignment must satisfy all of these constraints:

1. The teacher's grade band contains the student's grade band.
2. The teacher can teach the subject being assigned.
3. The teacher's unique-student count does not exceed the Stage 2 cap of 2.
4. The teacher's declared maximum is never exceeded.
5. A teacher cannot be assigned to the same student twice.
6. Different subjects of the same student must use different teachers.
7. Each student-subject need is assigned at most once.

## Strict Continuation Match

A match is strict continuation only when all of the following are true:

- Teacher willingness is `是`.
- Student willingness is `是`.
- Teacher column E contains the student name.
- The subject in teacher column E equals the subject being assigned.
- All hard constraints are satisfied.

Do not use the student's previous-teacher column to infer the previous subject.

## Stage 1

Stage 1 gives strict continuation matching absolute priority. Assign every valid strict continuation triple that can fit the hard constraints, even when this gives one student two Stage 1 matches. A student may have two Stage 1 teachers only when the two subjects are different, the teachers are different, and both matches are strict continuation triples.

Use this lexicographic priority order:

1. Maximize the number of strict continuation matches.
2. Give every student at least one Stage 1 match.
3. Preserve feasible capacity for Stage 2, especially for teachers who are otherwise unmatched.
4. Maximize total subject coverage.
5. Prefer earlier grades using the project grade order.
6. Break remaining ties by student source order, then teacher source order.

After all continuation matches are placed:

- A student with no continuation match receives one Stage 1 normal match.
- A student with one continuation match does not receive a normal second Stage 1 match; the remaining subject goes to Stage 2.
- A student with two continuation matches has both subjects matched in Stage 1.

## Stage 2

Stage 2 starts with the residual subject needs and the remaining Stage 2 capacity.

### Pass 1: Zero-Match Teachers

1. Identify teachers with zero Stage 1 students.
2. Give each such teacher at most one student in this pass.
3. Maximize the number of zero-match teachers who receive a student.
4. Apply the hard constraints and strict continuation when possible.
5. Use the project grade order and source order as tie-breakers.

### Pass 2: Remaining Capacity

1. Use remaining available capacity, never exceeding two unique students per teacher.
2. Maximize residual subject coverage.
3. Apply the same hard constraints, grade order, and deterministic tie-breakers.
4. Stop when no eligible assignment remains.

## Capacity Semantics

- `declared maximum`: value in teacher table column G, capped for cleaning at 3.
- `algorithmic cap`: 2 for every teacher in Stage 2.
- `assigned students`: unique students assigned by the end of Stage 2.
- `remaining capacity`: `declared maximum - assigned students`.

Positive remaining capacity is output for later manual adjustment. It is not automatically assigned by the skill.

## Failure and Warning Conditions

Stop and report rather than guessing when:

- Teacher column E contains one student with multiple subjects for the same teacher.
- A continuation triple cannot be verified.
- A teacher exceeds two unique students.
- A student-subject is assigned to two teachers.
- The workbook structure does not match the input contract.
- The same student would be assigned to two different teachers for the same subject.
