# Database Design and Management

## Đề 1

![DE1](%C4%90%E1%BB%81%201.png)

### Solve

a. Create table Student with the constraint that student age should be over 17.

```sql
CREATE TABLE Student (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    date_of_birth DATE,
    city VARCHAR(20),
    CONSTRAINT chk_age CHECK (DATEDIFF(YEAR, date_of_birth, GETDATE()) > 17)
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

or

```sql
SELECT c.name
FROM Course c
JOIN Enroll e ON e.cid = c.id
GROUP BY c.name
HAVING COUNT(e.sid) = (
    SELECT TOP 1 COUNT(e.sid)
                      FROM Course c
                      JOIN Enroll e ON e.cid = c.id
                      GROUP BY c.name
                      ORDER BY COUNT(e.sid) DESC
);
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

or

```sql
SELECT s.name
FROM Student s
JOIN Enroll e ON s.id = e.sid
JOIN Course c ON e.cid = c.id
WHERE c.name = 'Database' AND s.id IN (
    SELECT s.id
    FROM Student s
    JOIN Enroll e ON s.id = e.sid
    JOIN Course c ON e.cid = c.id
    WHERE c.name = 'Networking'
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

## Đề 2

![Đề 2](%C4%90%E1%BB%81%202.png)

### Giải

a. Liệt kê tên các nhân viên vừa tham gia dự án “Falcon-9” vừa tham gia dự án “Tesla”

```sql
SELECT e.Name
FROM Employee e
JOIN Enroll er ON e.ID = er.EID
JOIN Project p ON p.ID = er.PID
WHERE p.Name = 'Falcon-9' AND e.ID IN (
    SELECT er.EID
    FROM Enroll er 
    JOIN Project p ON p.ID = er.PID
    WHERE p.Name = 'Tesla'
);
```

b. Liệt kê tên các nhân không tham gia dự án “Tesla

```sql
SELECT e.Name
FROM Employee e
WHERE e.ID NOT IN (
    SELECT er.EID
    FROM Enroll er
    JOIN Project p ON er.PID = p.ID AND p.Name = 'Tesla'
);
```

c. Liệt kê ID nhân viên tham gia đồng thời tất cả các dự án còn lại ngoài dự án “Tesla”

d. Liệt kê ID một nhân viên tham gia nhiều dự án nhất

```sql
SELECT er.EID
FROM Enroll er
GROUP BY er.EID
HAVING COUNT(er.EID) = (
    SELECT MAX(project_count)
    FROM (
        SELECT COUNT(PID) as project_count
        FROM Enroll
        GROUP BY EID
    ) AS Subquery
)
```

e. Liệt kê tất tất cả các nhân viên tham gia ít nhất 3 dự án, và tổng mức lương khi tham gia các dự án của họ là ít nhất

```sql
SELECT e.*
FROM Employee e
JOIN Enroll er ON e.ID = er.EID
GROUP BY e.ID
HAVING COUNT(er.PID) >= 3
ORDER BY SUM(er.salary) ASC;
```

f. Liệt kê các nhân viên mà họ luôn có mức lương cao nhất trong tất cả các dự án mà họ tham gia

```sql

```

g. Với mỗi dự án, liệt kê tên dự án, số người tham gia, số nam, số nữ trong dự án

h. Liệt kê tên dự án mà budget đang nhỏ hơn lương của tất cả các nhân viên trong dự án đó cộng lại

i. Liệt kê tên các dự án mà không có dự án khác có cùng số lượng nhân viên tham gia

j. Liệt kê tên nhân viên, tên dự án mà lương của họ trong dự án đó bằng lương của tất cả các nhân viên trong nhóm đó cộng lại

k. Liệt kê các dự án có tỉ lệ số nhân viên lữ tham gia lớn hơn 30%
