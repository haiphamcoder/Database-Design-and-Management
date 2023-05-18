# Đề 1

Given the following database schema:

> Student(**id**, name, date_of_birth, city)
>
> Course(**id**, name)
>
> Enroll(**sid**, **cid**, grade)

Assume date_of birth has date datatype. Please write SQL statements for the following questions:

a. Create table Student with the constraint that student age should be over 17.

```sql
CREATE TABLE Student (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    date_of_birth DATE,
    city VARCHAR(255),
    CHECK (DATEDIFF(YEAR, date_of_birth, GETDATE()) > 17)
);
```

b. List the name of all students from “Hanoi”, taking the course “Database” with the final grades > 8.

```sql
SELECT s.name
FROM Student s
JOIN Enroll e ON s.id = e.sid
JOIN Course c ON e.cid = c.id
WHERE s.city = 'Hanoi' AND c.name = 'Database' AND e.grade > 8;
```

c. List every city with its associated number of students coming from that city

```sql
SELECT city, COUNT(id) as num_students
FROM Student
GROUP BY city;
```

d. Show the name of one course that has the greatest number of students enrolled

```sql
SELECT TOP 1 c.name
FROM Course c
JOIN Enroll e ON c.id = e.cid
GROUP BY c.name
ORDER BY COUNT(e.sid) DESC;
```

e. Show the name of all the courses that have the greatest number of students enrolled

```sql
WITH CourseEnrollment AS (
    SELECT c.name, COUNT(e.sid) as num_students
    FROM Course c
    JOIN Enroll e ON c.id = e.cid
    GROUP BY c.name
)
SELECT name
FROM CourseEnrollment
WHERE num_students = (SELECT MAX(num_students) FROM CourseEnrollment);
```

f. Show the name of all students that enrolled in “database” and also “networking”.

```sql
SELECT s.name
FROM Student s
WHERE EXISTS (
    SELECT 1
    FROM Enroll e
    JOIN Course c ON e.cid = c.id
    WHERE e.sid = s.id AND c.name = 'Database'
) AND EXISTS (
    SELECT 1
    FROM Enroll e
    JOIN Course c ON e.cid = c.id
    WHERE e.sid = s.id AND c.name = 'Networking'
);
```

g. Show the name of all students that enrolled in “database” or “networking”.

```sql
SELECT s.name
FROM Student s
JOIN Enroll e ON s.id = e.sid
JOIN Course c ON e.cid = c.id
WHERE c.name = 'Database' OR c.name = 'Networking'
GROUP BY s.name;
```

h. Show the course name that has no student enrolled.

```sql
SELECT c.name
FROM Course c
LEFT JOIN Enroll e ON c.id = e.cid
WHERE e.sid IS NULL;
```

Relational algebra expression for the question:
List the name of all students from “Hanoi”, taking the course “Database” with the final grades > 8.

$$ \pi_{name}(\sigma_{city='Hanoi' \land grade>8 \land course.name='Database'}(Student \bowtie Enroll \bowtie Course)) $$
