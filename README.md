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

# Лабораторная 4

## Создание генератора, сложного запроса и оптимизация его через индекс.

```sql
INSERT INTO shugaley2271.order(customer_id, order_date, order_cost, car_washer_id)
SELECT 
    floor(random() * 4 + 1)::int AS customer_id,
    to_char(NOW() - (random() * (INTERVAL '365 days')), 'DD.MM.YYYY') AS order_date,
    CASE WHEN random() <= 0.8 THEN (floor(random() * 200) + 1000)::int ELSE (floor(random() * 1000) + 300)::int END AS order_cost,
    floor(random() * 4 + 1)::int AS car_washer_id
FROM generate_series(1, 20000);

SELECT * FROM shugaley2271.order

CREATE OR REPLACE FUNCTION safe_to_date(text)
RETURNS date
LANGUAGE sql
IMMUTABLE
AS $$
    SELECT TO_DATE($1, 'DD.MM.YYYY');
$$;

-- Создаем индекс с использованием safe_to_date
CREATE INDEX idx_order_date_safe ON shugaley2271.order 
USING btree (safe_to_date(order_date));

-- И составной индекс
CREATE INDEX idx_order_date_cost_safe ON shugaley2271.order 
USING btree (safe_to_date(order_date), order_cost);

-- В запросе ТОЖЕ используем safe_to_date
EXPLAIN ANALYZE
SELECT 
    cw.address,    
    COUNT(o.order_id) AS total_orders,    
    SUM(o.order_cost) AS total_revenue 
FROM shugaley2271.order o
LEFT JOIN shugaley2271.car_washer cw_er ON o.car_washer_id = cw_er.car_washer_id
LEFT JOIN shugaley2271.car_wash cw ON cw_er.car_wash_id = cw.car_wash_id
WHERE    
    safe_to_date(o.order_date) >= DATE_TRUNC('month', NOW()) - INTERVAL '3 months'
    AND safe_to_date(o.order_date) < DATE_TRUNC('month', NOW())
    AND o.order_cost BETWEEN 1000 AND 1200
GROUP BY cw.address
ORDER BY total_revenue DESC;
```

# Лабораторная работа 5

## Создание триггерных функций

```sql
-- Таблица для журналирования всех изменений
CREATE TABLE IF NOT EXISTS change_log (
    log_id SERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    operation VARCHAR(10) NOT NULL CHECK (operation IN ('INSERT', 'UPDATE', 'DELETE')),
    record_id INTEGER,
    old_data JSONB,
    new_data JSONB,
    change_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
    username VARCHAR(100) DEFAULT CURRENT_USER NOT NULL,
    application_name VARCHAR(100) DEFAULT current_setting('application_name', true)
);

-- Индексы для ускорения поиска в журнале
CREATE INDEX idx_change_log_table ON change_log(table_name);
CREATE INDEX idx_change_log_time ON change_log(change_time);
CREATE INDEX idx_change_log_operation ON change_log(operation);
CREATE INDEX idx_change_log_record ON change_log(table_name, record_id);


-- ТРИГГЕР 1: Удаление мойщиков при удалении автомойки
CREATE OR REPLACE FUNCTION delete_car_washers_on_car_wash_delete()
RETURNS TRIGGER AS $$
BEGIN
    RAISE NOTICE 'Триггер delete_car_washers_on_car_wash_delete: удаление мойщиков автомойки ID=%', OLD.car_wash_id;
    
    -- Удаляем всех мойщиков, работавших на этой автомойке
    DELETE FROM car_washer WHERE car_wash_id = OLD.car_wash_id;
    
    RAISE NOTICE 'Удалены все мойщики автомойки ID=%', OLD.car_wash_id;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_cascade_delete_car_washer
BEFORE DELETE ON car_wash
FOR EACH ROW
EXECUTE FUNCTION delete_car_washers_on_car_wash_delete();

COMMENT ON FUNCTION delete_car_washers_on_car_wash_delete() IS 'Удаляет всех мойщиков при удалении автомойки';
COMMENT ON TRIGGER trig_cascade_delete_car_washer ON car_wash IS 'Каскадное удаление мойщиков при удалении автомойки';

-- ТРИГГЕР 2: Удаление заказов при удалении клиента
CREATE OR REPLACE FUNCTION delete_orders_on_customer_delete()
RETURNS TRIGGER AS $$
BEGIN
    RAISE NOTICE 'Триггер delete_orders_on_customer_delete: удаление заказов клиента ID=%', OLD.customer_id;
    
    -- Удаляем все заказы этого клиента
    DELETE FROM "order" WHERE customer_id = OLD.customer_id;
    
    RAISE NOTICE 'Удалены все заказы клиента ID=%', OLD.customer_id;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_cascade_delete_orders
BEFORE DELETE ON customer
FOR EACH ROW
EXECUTE FUNCTION delete_orders_on_customer_delete();

COMMENT ON FUNCTION delete_orders_on_customer_delete() IS 'Удаляет все заказы при удалении клиента';
COMMENT ON TRIGGER trig_cascade_delete_orders ON customer IS 'Каскадное удаление заказов при удалении клиента';

-- ТРИГГЕР 3: Удаление заказов при удалении мойщика
CREATE OR REPLACE FUNCTION delete_orders_on_car_washer_delete()
RETURNS TRIGGER AS $$
BEGIN
    RAISE NOTICE 'Триггер delete_orders_on_car_washer_delete: удаление заказов мойщика ID=%', OLD.car_washer_id;
    
    -- Удаляем все заказы, назначенные этому мойщику
    DELETE FROM "order" WHERE car_washer_id = OLD.car_washer_id;
    
    RAISE NOTICE 'Удалены все заказы мойщика ID=%', OLD.car_washer_id;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_cascade_delete_orders_by_washer
BEFORE DELETE ON car_washer
FOR EACH ROW
EXECUTE FUNCTION delete_orders_on_car_washer_delete();

COMMENT ON FUNCTION delete_orders_on_car_washer_delete() IS 'Удаляет все заказы при удалении мойщика';
COMMENT ON TRIGGER trig_cascade_delete_orders_by_washer ON car_washer IS 'Каскадное удаление заказов при удалении мойщика';


-- Функция для логирования изменений в любой таблице
CREATE OR REPLACE FUNCTION log_changes()
RETURNS TRIGGER AS $$
DECLARE
    v_old_data JSONB;
    v_new_data JSONB;
    v_record_id INTEGER;
BEGIN
    -- Определяем данные в зависимости от типа операции
    IF TG_OP = 'INSERT' THEN
        v_new_data = row_to_json(NEW);
        v_record_id = NEW.id;
    ELSIF TG_OP = 'UPDATE' THEN
        v_old_data = row_to_json(OLD);
        v_new_data = row_to_json(NEW);
        v_record_id = NEW.id;
    ELSIF TG_OP = 'DELETE' THEN
        v_old_data = row_to_json(OLD);
        v_record_id = OLD.id;
    END IF;
    
    -- Определяем ID записи (ищем поле с суффиксом _id)
    IF TG_OP IN ('INSERT', 'UPDATE') THEN
        IF NEW.car_wash_id IS NOT NULL THEN
            v_record_id = NEW.car_wash_id;
        ELSIF NEW.car_washer_id IS NOT NULL THEN
            v_record_id = NEW.car_washer_id;
        ELSIF NEW.customer_id IS NOT NULL THEN
            v_record_id = NEW.customer_id;
        ELSIF NEW.order_id IS NOT NULL THEN
            v_record_id = NEW.order_id;
        END IF;
    ELSIF TG_OP = 'DELETE' THEN
        IF OLD.car_wash_id IS NOT NULL THEN
            v_record_id = OLD.car_wash_id;
        ELSIF OLD.car_washer_id IS NOT NULL THEN
            v_record_id = OLD.car_washer_id;
        ELSIF OLD.customer_id IS NOT NULL THEN
            v_record_id = OLD.customer_id;
        ELSIF OLD.order_id IS NOT NULL THEN
            v_record_id = OLD.order_id;
        END IF;
    END IF;
    
    -- Вставляем запись в журнал
    INSERT INTO change_log (
        table_name,
        operation,
        record_id,
        old_data,
        new_data,
        username,
        application_name
    ) VALUES (
        TG_TABLE_NAME,
        TG_OP,
        v_record_id,
        v_old_data,
        v_new_data,
        CURRENT_USER,
        current_setting('application_name', true)
    );
    
    RAISE NOTICE 'Триггер log_changes: записано изменение в таблице % (операция: %)', 
        TG_TABLE_NAME, TG_OP;
    
    -- Возвращаем соответствующую запись
    IF TG_OP = 'DELETE' THEN
        RETURN OLD;
    ELSE
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;


-- Журналирование для таблицы car_wash
CREATE TRIGGER trig_log_car_wash_changes
AFTER INSERT OR UPDATE OR DELETE ON car_wash
FOR EACH ROW
EXECUTE FUNCTION log_changes();

COMMENT ON TRIGGER trig_log_car_wash_changes ON car_wash IS 'Журналирование изменений в таблице автомоек';

-- Журналирование для таблицы car_washer
CREATE TRIGGER trig_log_car_washer_changes
AFTER INSERT OR UPDATE OR DELETE ON car_washer
FOR EACH ROW
EXECUTE FUNCTION log_changes();

COMMENT ON TRIGGER trig_log_car_washer_changes ON car_washer IS 'Журналирование изменений в таблице мойщиков';

-- Журналирование для таблицы customer
CREATE TRIGGER trig_log_customer_changes
AFTER INSERT OR UPDATE OR DELETE ON customer
FOR EACH ROW
EXECUTE FUNCTION log_changes();

COMMENT ON TRIGGER trig_log_customer_changes ON customer IS 'Журналирование изменений в таблице клиентов';

-- Журналирование для таблицы order
CREATE TRIGGER trig_log_order_changes
AFTER INSERT OR UPDATE OR DELETE ON "order"
FOR EACH ROW
EXECUTE FUNCTION log_changes();

COMMENT ON TRIGGER trig_log_order_changes ON "order" IS 'Журналирование изменений в таблице заказов';
```