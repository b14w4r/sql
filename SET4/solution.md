### Задача 1
**Найти всех сотрудников, подчиняющихся Ивану Иванову (с EmployeeID = 1), включая их подчиненных и подчиненных подчиненных. Для каждого сотрудника выводится информация, включая их имя, ID менеджера, название отдела и роли, а также проекты и задачи, если они есть.**
```sql
WITH RECURSIVE Subordinates AS (
    SELECT EmployeeID, Name, ManagerID, DepartmentID
    FROM Employees
    WHERE ManagerID = 1
    UNION ALL
    SELECT e.EmployeeID, e.Name, e.ManagerID, e.DepartmentID
    FROM Employees e
    INNER JOIN Subordinates s ON e.ManagerID = s.EmployeeID
)
SELECT 
    e.EmployeeID,
    e.Name AS EmployeeName,
    e.ManagerID,
    d.DepartmentName,
    r.RoleName,
    GROUP_CONCAT(p.ProjectName) AS ProjectNames,
    GROUP_CONCAT(t.TaskName) AS TaskNames
FROM Subordinates s
JOIN Employees e ON e.EmployeeID = s.EmployeeID
LEFT JOIN Departments d ON e.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON e.RoleID = r.RoleID
LEFT JOIN Projects p ON e.EmployeeID = p.ProjectID
LEFT JOIN Tasks t ON e.EmployeeID = t.AssignedTo
GROUP BY e.EmployeeID
ORDER BY e.Name;

```
---
### Задача 2
**Найти всех сотрудников, подчиняющихся Ивану Иванову, включая их подчиненных и подчиненных подчиненных. Для каждого сотрудника вывести информацию о проектах и задачах, а также количество задач и количество непосредственных подчиненных (не включая подчиненных их подчиненных).**
```sql
WITH RECURSIVE Subordinates AS (
    SELECT EmployeeID, Name, ManagerID, DepartmentID
    FROM Employees
    WHERE ManagerID = 1
    UNION ALL
    SELECT e.EmployeeID, e.Name, e.ManagerID, e.DepartmentID
    FROM Employees e
    INNER JOIN Subordinates s ON e.ManagerID = s.EmployeeID
)
SELECT 
    e.EmployeeID,
    e.Name AS EmployeeName,
    e.ManagerID,
    d.DepartmentName,
    r.RoleName,
    GROUP_CONCAT(p.ProjectName) AS ProjectNames,
    GROUP_CONCAT(t.TaskName) AS TaskNames,
    COUNT(t.TaskID) AS TotalTasks,
    (SELECT COUNT(*) FROM Employees WHERE ManagerID = e.EmployeeID) AS TotalSubordinates
FROM Subordinates s
JOIN Employees e ON e.EmployeeID = s.EmployeeID
LEFT JOIN Departments d ON e.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON e.RoleID = r.RoleID
LEFT JOIN Projects p ON e.EmployeeID = p.ProjectID
LEFT JOIN Tasks t ON e.EmployeeID = t.AssignedTo
GROUP BY e.EmployeeID
ORDER BY e.Name;

```
---
### Задача 3
**Найти всех сотрудников, которые занимают роль менеджера и имеют хотя бы одного подчиненного. Для каждого такого сотрудника нужно вывести информацию о проекте, задачах и количестве всех подчиненных (включая их подчиненных).**
```sql
WITH RECURSIVE Subordinates AS (
    SELECT EmployeeID, Name, ManagerID, DepartmentID
    FROM Employees
    WHERE ManagerID IN (SELECT EmployeeID FROM Employees WHERE RoleID = 1)
    UNION ALL
    SELECT e.EmployeeID, e.Name, e.ManagerID, e.DepartmentID
    FROM Employees e
    INNER JOIN Subordinates s ON e.ManagerID = s.EmployeeID
)
SELECT 
    e.EmployeeID,
    e.Name AS EmployeeName,
    e.ManagerID,
    d.DepartmentName,
    r.RoleName,
    GROUP_CONCAT(p.ProjectName) AS ProjectNames,
    GROUP_CONCAT(t.TaskName) AS TaskNames,
    (SELECT COUNT(*) FROM Employees WHERE ManagerID = e.EmployeeID) AS TotalSubordinates
FROM Subordinates s
JOIN Employees e ON e.EmployeeID = s.EmployeeID
LEFT JOIN Departments d ON e.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON e.RoleID = r.RoleID
LEFT JOIN Projects p ON e.EmployeeID = p.ProjectID
LEFT JOIN Tasks t ON e.EmployeeID = t.AssignedTo
GROUP BY e.EmployeeID
HAVING TotalSubordinates > 0
ORDER BY e.Name;

```