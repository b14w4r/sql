# I
### Задача 1
Запрос для нахождения производителей и моделей спортивных мотоциклов с мощностью более 150 л.с., ценой менее 20 тыс. долларов, отсортированных по мощности в порядке убывания:

```sql
SELECT v.maker, v.model
FROM Motorcycle m
JOIN Vehicle v ON m.model = v.model
WHERE m.horsepower > 150
  AND m.price < 20000
  AND m.type = 'Sport'
ORDER BY m.horsepower DESC;
```

### Задача 2
Объединенный запрос для поиска автомобилей, мотоциклов и велосипедов, удовлетворяющих заданным критериям, с сортировкой по мощности:

```sql
SELECT 
    v.maker, 
    v.model, 
    CASE 
        WHEN c.vin IS NOT NULL THEN c.horsepower
        WHEN m.vin IS NOT NULL THEN m.horsepower
        ELSE NULL 
    END AS horsepower,
    CASE 
        WHEN c.vin IS NOT NULL THEN c.engine_capacity
        WHEN m.vin IS NOT NULL THEN m.engine_capacity
        ELSE NULL 
    END AS engine_capacity,
    CASE 
        WHEN c.vin IS NOT NULL THEN 'Car'
        WHEN m.vin IS NOT NULL THEN 'Motorcycle'
        WHEN b.serial_number IS NOT NULL THEN 'Bicycle'
    END AS vehicle_type
FROM Vehicle v
LEFT JOIN Car c ON v.model = c.model 
    AND c.horsepower > 150 
    AND c.engine_capacity < 3 
    AND c.price < 35000
LEFT JOIN Motorcycle m ON v.model = m.model 
    AND m.horsepower > 150 
    AND m.engine_capacity < 1.5 
    AND m.price < 20000
LEFT JOIN Bicycle b ON v.model = b.model 
    AND b.gear_count > 18 
    AND b.price < 4000
WHERE c.vin IS NOT NULL OR m.vin IS NOT NULL OR b.serial_number IS NOT NULL
ORDER BY horsepower DESC;


```
# II
### **Задача 1**  
**Найти автомобиль из каждого класса с наименьшей средней позицией в гонках. Вывести информацию об автомобиле, классе, средней позиции и количестве гонок. Отсортировать по средней позиции.**  

```sql
WITH AvgPositions AS (
    SELECT 
        c.name AS car_name, 
        c.class AS car_class, 
        AVG(r.position) AS average_position, 
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    GROUP BY c.name, c.class
),
ClassMinAvg AS (
    SELECT 
        ap.car_class, 
        MIN(ap.average_position) AS min_avg_position
    FROM AvgPositions ap
    GROUP BY ap.car_class
)
SELECT 
    ap.car_name, 
    ap.car_class, 
    ROUND(ap.average_position, 4) AS average_position, 
    ap.race_count
FROM AvgPositions ap
JOIN ClassMinAvg cma ON ap.car_class = cma.car_class
WHERE ap.average_position = cma.min_avg_position
ORDER BY ap.average_position;
```

### **Задача 2**  
**Найти автомобиль с наименьшей средней позицией среди всех, если их несколько — выбрать по алфавиту.**  

```sql
WITH AvgPositions AS (
    SELECT 
        c.name AS car_name, 
        c.class AS car_class, 
        AVG(r.position) AS average_position, 
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    GROUP BY c.name, c.class
)
SELECT 
    ap.car_name, 
    ap.car_class, 
    ROUND(ap.average_position, 4) AS average_position, 
    ap.race_count, 
    cl.country AS car_country
FROM AvgPositions ap
JOIN Classes cl ON ap.car_class = cl.class
WHERE ap.average_position = (SELECT MIN(average_position) FROM AvgPositions)
ORDER BY ap.car_name
LIMIT 1;
```

---

### **Задача 3**  
**Найти классы с наименьшей средней позицией в гонках и вывести автомобили из этих классов.**  

```sql
WITH AvgPositions AS (
    SELECT 
        c.name AS car_name, 
        c.class AS car_class, 
        AVG(r.position) AS average_position, 
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    GROUP BY c.name, c.class
),
ClassAvg AS (
    SELECT 
        ap.car_class, 
        AVG(ap.average_position) AS class_avg
    FROM AvgPositions ap
    GROUP BY ap.car_class
),
MinClassAvg AS (
    SELECT MIN(class_avg) AS min_avg FROM ClassAvg
)
SELECT 
    ap.car_name, 
    ap.car_class, 
    ROUND(ap.average_position, 4) AS average_position, 
    ap.race_count, 
    cl.country AS car_country, 
    (SELECT COUNT(*) FROM Results r JOIN Cars c ON r.car = c.name WHERE c.class = ap.car_class) AS total_races
FROM AvgPositions ap
JOIN Classes cl ON ap.car_class = cl.class
JOIN ClassAvg ca ON ap.car_class = ca.car_class
WHERE ca.class_avg = (SELECT min_avg FROM MinClassAvg);
```

---

### **Задача 4**  
**Найти автомобили, чья средняя позиция лучше средней по их классу (если в классе 2 и более машин).**  

```sql
WITH AvgPositions AS (
    SELECT 
        c.name AS car_name, 
        c.class AS car_class, 
        AVG(r.position) AS average_position, 
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    GROUP BY c.name, c.class
),
ClassAvg AS (
    SELECT 
        ap.car_class, 
        AVG(ap.average_position) AS class_avg, 
        COUNT(ap.car_class) AS car_count
    FROM AvgPositions ap
    GROUP BY ap.car_class
)
SELECT 
    ap.car_name, 
    ap.car_class, 
    ROUND(ap.average_position, 4) AS average_position, 
    ap.race_count, 
    cl.country AS car_country
FROM AvgPositions ap
JOIN Classes cl ON ap.car_class = cl.class
JOIN ClassAvg ca ON ap.car_class = ca.car_class
WHERE ca.car_count > 1 AND ap.average_position < ca.class_avg
ORDER BY ap.car_class, ap.average_position;
```

---

### **Задача 5**  
**Найти классы с наибольшим числом машин, у которых средняя позиция больше 3.0.**  

```sql
WITH AvgPositions AS (
    SELECT 
        c.name AS car_name, 
        c.class AS car_class, 
        AVG(r.position) AS average_position, 
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    GROUP BY c.name, c.class
),
LowPositionCars AS (
    SELECT 
        ap.car_class, 
        COUNT(*) AS low_position_count
    FROM AvgPositions ap
    WHERE ap.average_position > 3.0
    GROUP BY ap.car_class
),
ClassTotalRaces AS (
    SELECT 
        c.class AS car_class, 
        COUNT(r.race) AS total_races
    FROM Results r
    JOIN Cars c ON r.car = c.name
    GROUP BY c.class
)
SELECT 
    ap.car_name, 
    ap.car_class, 
    ROUND(ap.average_position, 4) AS average_position, 
    ap.race_count, 
    cl.country AS car_country, 
    ctr.total_races, 
    lpc.low_position_count
FROM AvgPositions ap
JOIN Classes cl ON ap.car_class = cl.class
JOIN ClassTotalRaces ctr ON ap.car_class = ctr.car_class
JOIN LowPositionCars lpc ON ap.car_class = lpc.car_class
WHERE ap.average_position > 3.0
ORDER BY lpc.low_position_count DESC;
```

# III

### Задача 1  
**Найти клиентов, которые сделали более двух бронирований в разных отелях, вывести их данные, общее количество бронирований, список отелей, в которых они бронировали номера, а также среднюю длительность их пребывания. Результаты должны быть отсортированы по количеству бронирований.**  

```sql
SELECT 
    c.name, 
    c.email, 
    c.phone, 
    COUNT(b.ID_booking) AS total_bookings, 
    GROUP_CONCAT(DISTINCT h.name ORDER BY h.name) AS hotels, 
    ROUND(AVG(DATEDIFF(b.check_out_date, b.check_in_date)), 4) AS avg_duration
FROM Customer c
JOIN Booking b ON c.ID_customer = b.ID_customer
JOIN Room r ON b.ID_room = r.ID_room
JOIN Hotel h ON r.ID_hotel = h.ID_hotel
GROUP BY c.ID_customer
HAVING COUNT(DISTINCT h.ID_hotel) > 1 AND COUNT(b.ID_booking) > 2
ORDER BY total_bookings DESC;
```

---

### Задача 2  
**Найти клиентов, которые сделали более двух бронирований в разных отелях и потратили более 500 долларов, затем объединить их данные с клиентами, потратившими более 500 долларов, и вывести результат, отсортированный по общей потраченной сумме.**  

```sql
WITH TotalSpent AS (
    SELECT 
        b.ID_customer, 
        SUM(r.price * DATEDIFF(b.check_out_date, b.check_in_date)) AS total_spent, 
        COUNT(b.ID_booking) AS total_bookings, 
        COUNT(DISTINCT h.ID_hotel) AS unique_hotels
    FROM Booking b
    JOIN Room r ON b.ID_room = r.ID_room
    JOIN Hotel h ON r.ID_hotel = h.ID_hotel
    GROUP BY b.ID_customer
    HAVING total_bookings > 2 AND unique_hotels > 1
)
SELECT 
    t.ID_customer, 
    c.name, 
    t.total_bookings, 
    t.total_spent, 
    t.unique_hotels
FROM TotalSpent t
JOIN Customer c ON t.ID_customer = c.ID_customer
WHERE t.total_spent > 500
ORDER BY t.total_spent ASC;
```

---

### Задача 3  
**Провести анализ предпочтений клиентов по типу отелей. Каждому отелю присвоить категорию (дешевый, средний, дорогой), а каждому клиенту — предпочитаемый тип отеля на основе его бронирований. Вывести информацию о клиенте, его предпочитаемом типе отеля и перечень отелей, которые он посетил. Результаты должны быть отсортированы по типу отеля.** 

```sql
WITH HotelCategories AS (
    SELECT 
        h.ID_hotel, 
        CASE 
            WHEN AVG(r.price) < 175 THEN 'Дешевый'
            WHEN AVG(r.price) BETWEEN 175 AND 300 THEN 'Средний'
            ELSE 'Дорогой'
        END AS hotel_category
    FROM Room r
    JOIN Hotel h ON r.ID_hotel = h.ID_hotel
    GROUP BY h.ID_hotel
),
CustomerPreferences AS (
    SELECT 
        b.ID_customer, 
        MAX(hc.hotel_category) AS preferred_hotel_type, 
        GROUP_CONCAT(DISTINCT h.name ORDER BY h.name) AS visited_hotels
    FROM Booking b
    JOIN Room r ON b.ID_room = r.ID_room
    JOIN Hotel h ON r.ID_hotel = h.ID_hotel
    JOIN HotelCategories hc ON h.ID_hotel = hc.ID_hotel
    GROUP BY b.ID_customer
)
SELECT 
    cp.ID_customer, 
    c.name, 
    cp.preferred_hotel_type, 
    cp.visited_hotels
FROM CustomerPreferences cp
JOIN Customer c ON cp.ID_customer = c.ID_customer
ORDER BY 
    CASE 
        WHEN cp.preferred_hotel_type = 'Дешевый' THEN 1
        WHEN cp.preferred_hotel_type = 'Средний' THEN 2
        ELSE 3
    END;
```
# IV
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