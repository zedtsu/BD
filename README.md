# Лабораторная 1

## Таблица `customer` (Клиент)
| Поле | Тип | Описание |
|------|-----|----------|
| `customer_id` (PK) | `int` | Уникальный идентификатор клиента |
| `full_name` | `string` | Полное имя клиента |
| `phone` | `string` | Номер телефона клиента |
| `car_number` | `string` | Номер автомобиля клиента |
| `car_stamp` | `string` | Марка автомобиля |
| `car_model` | `string` | Модель автомобиля |

---

## Таблица `order` (Заказ)
| Поле | Тип | Описание |
|------|-----|----------|
| `order_id` (PK) | `int` | Уникальный идентификатор заказа |
| `customer_id` | `int` | Ссылка на клиента (FK → `customer.customer_id`) |
| `order_date` | `string` | Дата заказа |
| `order_cost` | `int` | Стоимость заказа |
| `car_washer_id` | `int` | Ссылка на мойщика (FK → `car_washer.car_washer_id`) |

---

## Таблица `car_wash` (Автомойка)
| Поле | Тип | Описание |
|------|-----|----------|
| `car_wash_id` (PK) | `int` | Уникальный идентификатор автомойки |
| `car_washer_id` | `int` | Ссылка на мойщика (FK → `car_washer.car_washer_id`) |
| `address` | `string` | Адрес автомойки |
| `type_of_service` | `string` | Тип услуги (например, ручная, автоматическая) |

---

## Таблица `car_washer` (Мойщик)
| Поле | Тип | Описание |
|------|-----|----------|
| `car_washer_id` (PK) | `int` | Уникальный идентификатор мойщика |
| `car_wash_id` | `int` | Ссылка на автомойку (FK → `car_wash.car_wash_id`) |
| `phone` | `int` | Номер телефона мойщика |
| `full_name` | `string` | Полное имя мойщика |

---

## Основные связи между таблицами:

1. **`customer` ↔ `order`**  
   Один клиент может иметь несколько заказов (`1:N`).

2. **`order` ↔ `car_washer`**  
   Один мойщик может выполнять несколько заказов (`1:N`).

3. **`car_washer` ↔ `car_wash`**  
   Одна автомойка может иметь нескольких мойщиков (`1:N`).

4. **`order` ↔ `car_wash`**  
   Связь через `car_washer_id` (заказ связан с мойщиком, который работает на определённой мойке).

---

# Лабораторная 2

# SQL-код для создания и заполнения таблиц системы автомойки

## Полный код PostgreSQL

```sql
-- 1. Создание таблицы customer
CREATE TABLE customer (
    customer_id INTEGER PRIMARY KEY,
    full_name VARCHAR(50) NOT NULL,
    phone VARCHAR(20) NOT NULL,
    car_number VARCHAR(20) NOT NULL,
    car_stamp VARCHAR(20) NOT NULL,
    car_model VARCHAR(20) NOT NULL
);

-- 2. Создание таблицы car_wash
CREATE TABLE car_wash (
    car_wash_id INTEGER PRIMARY KEY,
    phone INTEGER NOT NULL,
    address VARCHAR(100) NOT NULL,
    type_of_service VARCHAR(100) NOT NULL
);

-- 3. Создание таблицы car_washer
CREATE TABLE car_washer (
    car_washer_id INTEGER PRIMARY KEY,
    car_wash_id INTEGER NOT NULL,
    phone NUMERIC NOT NULL,
    full_name VARCHAR(50) NOT NULL,
    FOREIGN KEY (car_wash_id) REFERENCES car_wash(car_wash_id)
);

-- 4. Создание таблицы "order"
-- Используем кавычки, так как order - зарезервированное слово в PostgreSQL
CREATE TABLE "order" (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date VARCHAR(10) NOT NULL,
    order_cost NUMERIC NOT NULL,
    car_washer_id INTEGER NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id),
    FOREIGN KEY (car_washer_id) REFERENCES car_washer(car_washer_id)
);
```

## Заполнение таблиц данными

### Таблица customer
```sql
INSERT INTO customer (customer_id, full_name, phone, car_number, car_stamp, car_model) VALUES
(1, 'Mega Petr Sem', '8593658236', '123456-0V', 'Toyota', 'Prius'),
(2, 'Och van silov', '8593655573', '654322-VB', 'Toyota', 'Harrier'),
(3, 'Tokov ya Stepanov', '8593582573', '2a4375-tV', 'Honda', 'Accord'),
(4, 'Shiov Fedor Mosin', '8599572173', '572947-PT', 'Mazda', 'x4');
```

### Таблица car_wash
```sql
INSERT INTO car_wash (car_wash_id, phone, address, type_of_service) VALUES
(1, 845334789, 'Irkutsk, Neulitsa St., 12', 'dry cleaning'),
(2, 845333789, 'Irkutsk, Nevskogo St., 15/4', 'washing'),
(3, 845336789, 'Irkutsk, Nevskogo St., 20', 'dry cleaning'),
(4, 845333785, 'Irkutsk, Angeles St., 1', 'dry cleaning');
```

### Таблица car_washer
```sql
INSERT INTO car_washer (car_washer_id, car_wash_id, phone, full_name) VALUES
(1, 1, 8987654321, 'Los Petr Ivanov'),
(2, 3, 8975634072, 'Fid Alexey Petrov'),
(3, 4, 8975636092, 'Wid Fedor Makrov'),
(4, 2, 8975635257, 'Pos Ivan Iqragim');
```

### Таблица order
```sql
INSERT INTO "order" (order_id, customer_id, order_date, order_cost, car_washer_id) VALUES
(1, 1, '12.09 2025', 350, 3),
(2, 2, '12.09 2025', 750, 4),
(3, 3, '16.09 2025', 1200, 1),
(4, 4, '14.09 2025', 550, 2);
```

# Лабораторная 3

## Создание функции для вывода заказов по дате с сортировкой по цене и дате.

```sql
-- Создание представления для функции order_details
CREATE OR REPLACE VIEW shugaley2271.order_full_info AS
SELECT
    o.order_id,
    cw.type_of_service,
    o.order_cost,
    c.full_name AS customer_name,
    o.order_date,
    c.phone AS customer_phone,
    c.car_number,
    c.car_stamp,
    c.car_model,
    wash.address AS car_wash_address,
    washer.full_name AS washer_name,
    washer.phone AS washer_phone
FROM shugaley2271."order" o
JOIN shugaley2271.customer c
    ON c.customer_id = o.customer_id
JOIN shugaley2271.car_washer washer
    ON washer.car_washer_id = o.car_washer_id
JOIN shugaley2271.car_wash wash
    ON wash.car_wash_id = washer.car_wash_id
JOIN shugaley2271.car_wash cw
    ON cw.car_wash_id = wash.car_wash_id;
```

```sql
CREATE OR REPLACE FUNCTION shugaley2271.order_details(p_date VARCHAR(100))
RETURNS TABLE (
    order_id INT,
    type_of_service VARCHAR(100),
    order_cost NUMERIC,
    customer_name VARCHAR(100),
    order_date VARCHAR(100)
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT
        o.order_id,
        cw.type_of_service,
        o.order_cost,
        c.full_name AS customer_name,
        o.order_date AS order_date
    FROM shugaley2271.order o
    JOIN shugaley2271.customer c
        ON c.customer_id = o.customer_id
    JOIN shugaley2271.car_wash cw
        ON cw.car_wash_id = o.car_washer_id
    WHERE o.order_date = p_date
    ORDER BY 
        o.order_cost,
        o.order_date;
END;
$$;

SELECT * FROM shugaley2271.order_details('14.09.2025');


INSERT INTO shugaley2271.order(customer_id, order_id, order_date, order_cost)
```