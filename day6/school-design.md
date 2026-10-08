# School Database Design

## Students Table

The `students` table stores information about each student.

It contains:

- `id` as the primary key
- `name`
- `email`

The email column is marked `UNIQUE` so two students cannot use the same email address.

## Courses Table

The `courses` table stores information about each course.

It contains:

- `id` as the primary key
- `title`

Each course has its own unique id.

## Enrolments Table

The `enrolments` table connects students to courses.

It contains:

- `id` as the primary key
- `student_id` as a foreign key
- `course_id` as a foreign key
- `grade`

The combination of `student_id` and `course_id` is unique so the same student cannot enrol in the same course twice.

## Relationships

A student can have many enrolments, so the relationship between `students` and `enrolments` is one-to-many.

A course can also have many enrolments, so the relationship between `courses` and `enrolments` is one-to-many.

Students and courses have a many-to-many relationship because one student can take many courses, and one course can have many students.

The `enrolments` table is needed as a join table because it connects students and courses. It also stores information about the relationship, such as the student's grade.

## Index

I would add an index on `enrolments.student_id` because the system will often need to find all the courses taken by one student.

The SQL for the index would be:

    CREATE INDEX idx_enrolments_student_id
    ON enrolments(student_id);

This index would make these searches faster because the database would not need to scan every enrolment row.

## SQL or NoSQL

I would choose SQL for this school system because the data is structured and has clear relationships between students, courses and enrolments. SQL databases are well suited to this type of data because they support primary keys, foreign keys, joins and rules such as unique student emails and preventing duplicate enrolments. These features help keep the school data accurate and consistent, so SQL is a better choice than NoSQL for this system.