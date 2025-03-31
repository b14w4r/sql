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