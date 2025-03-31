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