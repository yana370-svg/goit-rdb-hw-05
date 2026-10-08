# HW5: Customer-Level Feature Store & Advanced SQL Pipeline

Цей проєкт демонструє побудову відтворюваного SQL-пайплайну для підготовки **customer-level feature store** на основі датасету Olist Brazilian E-commerce. Метою є підготовка даних для подальшого навчання ML-моделей прогнозування поведінки клієнтів з використанням складних трансформацій, кешування та оцінки вартості обчислень.

---

## Джерело датасету

- **Назва:** Olist Brazilian E-commerce Public Dataset
- **Джерело:** [Kaggle: Olist Brazilian E-commerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Опис:** Багатотабличний набір даних, що містить приблизно 100 тис. реальних замовлень на бразильській e-commerce-платформі Olist за період 2016–2018 років.
- **Використані таблиці:** `customers`, `orders`, `order_items`, `products`, `sellers`.

---

## Архітектура даних: Raw vs Typed шар

Для забезпечення надійності та цілісності даних використовується двошарова архітектура завантаження:

### 1. Raw Layer (Проміжний шар)
- **Назви таблиць:** `olist_*_raw`
- **Типи даних:** Усі колонки мають тип `TEXT`.
- **Призначення:** Пряме відображення вихідних CSV-файлів. Цей шар не має жодних обмежень (constraints). Це гарантує, що імпорт ніколи не "впаде" через невідповідність типів (наприклад, порожній рядок замість числа або нестандартний формат дати).

### 2. Typed Layer (Типізований шар)
- **Назви таблиць:** `olist_*` (без суфікса `_raw`)
- **Типи даних:** Строго типізовані (`INTEGER`, `NUMERIC`, `TIMESTAMP`, `CHAR(2)` тощо).
- **Обмеження (Constraints):** Впроваджено `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `CHECK` (наприклад, для валідації статусів замовлень або невід'ємних цін).
- **Призначення:** Єдине джерело правди для всіх аналітичних запитів, CTE-пайплайнів та Materialized Views. 
- *Примітка:* У вихідному CSV назви колонок `product_name_lenght` та `product_description_lenght` містять орфографічну помилку. У Typed-шарі вони нормалізовані до `product_name_length` та `product_description_length`.

---

## Вимоги до середовища

Для запуску проєкту необхідні наступні Python-бібліотеки:
```text
pgserver
psycopg2-binary
sqlalchemy
pandas
kagglehub



Проєкт розроблений для виконання в середовищі Google Colab або локальному Jupyter Notebook.
Крок 1: Встановлення залежностей та ініціалізація БД
Запустіть першу секцію ноутбука. Вона встановить необхідні бібліотеки, запустить локальний інстанс PostgreSQL через pgserver та перевірить з'єднання.
Крок 2: Завантаження даних
Виконайте комірку з kagglehub. Вона автоматично завантажить архів датасету та розпакує його в тимчасову директорію.
Крок 3: ETL (Raw → Typed)
Виконайте SQL-команди для завантаження CSV у raw-таблиці через pandas.to_sql.
Виконайте DDL-скрипт для створення typed-таблиць з усіма необхідними constraints.
Виконайте INSERT ... SELECT для міграції та типізації даних з raw у typed шар.
Переконайтеся у успішності через smoke-test (порівняння COUNT(*)).
Крок 4: Feature Engineering Pipeline
Виконайте великий SQL-запит із ланцюжком CTE (≥5 stages), який обчислює RFM-метрики, cohort quarter, rolling AOV, cancellation rate та preferred category.
Крок 5: Recursive CTE (Referral Graph)
Виконайте скрипт для створення синтетичної таблиці hw5_referrals та запустіть рекурсивний CTE для обчислення глибини та шляху в графі запрошень (із захистом від циклів).
Крок 6: Materialized View та EXPLAIN ANALYZE
Створіть mv_olist_customer_features на основі CTE-пайплайну.
Створіть UNIQUE INDEX на customer_id.
Виконайте REFRESH MATERIALIZED VIEW CONCURRENTLY.
Запустіть EXPLAIN (ANALYZE, BUFFERS) для порівняння вартості повного перерахунку CTE та швидкого lookup з Materialized View.