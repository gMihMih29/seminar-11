## Задание 3

1. Создайте таблицу с большим количеством данных:
    ```sql
    CREATE TABLE test_cluster AS 
    SELECT 
        generate_series(1,1000000) as id,
        CASE WHEN random() < 0.5 THEN 'A' ELSE 'B' END as category,
        md5(random()::text) as data;
    ```

2. Создайте индекс:
    ```sql
    CREATE INDEX test_cluster_cat_idx ON test_cluster(category);
    ```

3. Измерьте производительность до кластеризации:
    ```sql
    EXPLAIN ANALYZE
    SELECT * FROM test_cluster WHERE category = 'A';
    ```
    
    *План выполнения:*
    ```
    Bitmap Heap Scan on test_cluster  (cost=59.17..7696.73 rows=5000 width=68) (actual time=13.386..91.058 rows=500210 loops=1)
    Recheck Cond: (category = 'A'::text)
    Heap Blocks: exact=8334
    ->  Bitmap Index Scan on test_cluster_cat_idx  (cost=0.00..57.92 rows=5000 width=0) (actual time=12.286..12.286 rows=500210 loops=1)
            Index Cond: (category = 'A'::text)
    Planning Time: 0.253 ms
    Execution Time: 104.200 ms
    ```
    
    *Объясните результат:*
    - используется BitMap
    - стоит запрос от 59.17 до 7696.73
    - планируемое время - 0.253 ms
    - фактическое время - 104.200 ms

4. Выполните кластеризацию:
    ```sql
    CLUSTER test_cluster USING test_cluster_cat_idx;
    ```
    
    *Результат:*
    ```
    [2024-12-09 22:36:42] completed in 833 ms
    ```

5. Измерьте производительность после кластеризации:
    ```sql
    EXPLAIN ANALYZE
    SELECT * FROM test_cluster WHERE category = 'A';
    ```
    
    *План выполнения:*
    ```
    Bitmap Heap Scan on test_cluster  (cost=5564.45..20137.20 rows=499100 width=39) (actual time=10.964..50.887 rows=500210 loops=1)
    Recheck Cond: (category = 'A'::text)
    Heap Blocks: exact=4169
    ->  Bitmap Index Scan on test_cluster_cat_idx  (cost=0.00..5439.68 rows=499100 width=0) (actual time=10.464..10.464 rows=500210 loops=1)
            Index Cond: (category = 'A'::text)
    Planning Time: 0.187 ms
    Execution Time: 64.182 ms
    ```
    
    *Объясните результат:*
    - используется BitMap
    - стоит запрос от 5564.45 до 20137.20
    - планируемое время - 0.187 ms
    - фактическое время - 64.182 ms

6. Сравните производительность до и после кластеризации:
    
    *Сравнение:*
    - максимальная стоимость запроса увеличилась почти в три раза
    - запрос после кластеризации занял почти в два раза меньше
