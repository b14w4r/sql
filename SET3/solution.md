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