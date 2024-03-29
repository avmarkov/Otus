# Домашняя работа № 12. Триггеры, поддержка заполнения витрин.

### Скрипт и развернутое описание задачи – в ЛК (файл hw_triggers.sql) или по ссылке: https://disk.yandex.ru/d/l70AvknAepIJXQ. В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales). Есть запрос для генерации отчета – сумма продаж по каждому товару. БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.

### Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину). Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE
> Создал новую схему с таблицами и данными из предложенногос крипта hw_triggers.sql.
>
> Создал триггерную функцию sales_insert_update_delete_tf(). Описание этой функции оформил в виде комментариев непосредствено в самом скрипте триггерной функции
>
> Триггерная функция будет иметь следующий вид:

```sql
CREATE OR REPLACE FUNCTION pract_functions.sales_insert_update_delete_tf()
    RETURNS trigger
    LANGUAGE 'plpgsql'
  
AS $BODY$

DECLARE 
	goodprice numeric(12,2);	
	goodname character varying(63);
	goodssum numeric(16,2) = 0;
BEGIN
	-- Находим цену товара и его имя из таблицы goods и джойним найденный товар с таблицей good_sum_mart.
	-- Если sum_sale в таблице good_sum_mart не NULL, значит в таблице good_sum_mart точно есть этот товар и сумма по нему,
	-- т.к. на поля sum_sale и good_name в таблице good_sum_mart есть ограничения NOT NULL.
	-- Найденные данные по товару записываем в соответсвующие переменные
	SELECT g.good_price, g.good_name, gs.sum_sale into goodprice, goodname, goodssum 
	FROM pract_functions.goods g LEFT JOIN pract_functions.good_sum_mart gs ON g.good_name = gs.good_name
	WHERE (NEW.good_id = g.goods_id and TG_OP <> 'DELETE') or
	      (OLD.good_id = g.goods_id and TG_OP = 'DELETE');
	
	raise info 'goodprice, % ', goodprice;
	raise info 'goodname, % ', goodname;
	raise info 'goodssum, % ', goodssum;
	
	CASE TG_OP	
		WHEN 'INSERT' THEN 
			raise info 'INSERT';
			IF (goodssum is not null) -- значит сумма по товару и сам товар есть в таблице good_sum_mart
			THEN
				goodssum = goodssum + NEW.sales_qty * goodprice;
				raise info 'updating goodssum, %', goodssum;
				UPDATE pract_functions.good_sum_mart SET sum_sale = goodssum
				WHERE good_name = goodname; 
			ELSE -- значит суммы по товару и самого товара нет в таблице good_sum_mart, поэтому просто вставим товар с суммой по нему
				raise info 'inserting goodssum, %', goodssum;
				INSERT INTO pract_functions.good_sum_mart(good_name, sum_sale) 
				VALUES (goodname, NEW.sales_qty * goodprice);
			END IF;
			return new;
		WHEN 'UPDATE' THEN 
			raise info 'UPDATE';
			IF (goodssum is not null) -- значит сумма по товару и сам товар есть в таблице good_sum_mart
			THEN
				goodssum = goodssum - OLD.sales_qty * goodprice; -- вычитаем из суммы значение до обновления
				IF (goodssum < 0) 
				THEN
					goodssum = 0;
				END IF;
				goodssum = goodssum + NEW.sales_qty * goodprice; -- добаляем к сумме новое значение 
				raise info 'updating goodssum, %', goodssum;
				UPDATE pract_functions.good_sum_mart SET sum_sale = goodssum
				WHERE good_name = goodname; 
			ELSE -- значит суммы по товару и самого товара нет в таблице good_sum_mart, поэтому просто вставим товар с суммой по нему
				raise info 'inserting goodssum, %', goodssum;
				INSERT INTO pract_functions.good_sum_mart(good_name, sum_sale) 
				VALUES (goodname, NEW.sales_qty * goodprice);
			END IF;
			return new;
		WHEN 'DELETE' THEN -- значит сумма по товару и сам товар есть в таблице good_sum_mart
			raise info 'DELETE';
			IF (goodssum is not null)
			THEN
				goodssum = goodssum - OLD.sales_qty * goodprice; -- вычитаем из суммы по товары удаляемое значение
				IF (goodssum < 0) 
				THEN
					goodssum = 0;
				END IF;			
				raise info 'updating goodssum, %', goodssum;
				UPDATE pract_functions.good_sum_mart SET sum_sale = goodssum
				WHERE good_name = goodname; 
			ELSE -- если суммы по товару и самого товара нет в таблице good_sum_mart, то ничего делать и не надо, вычитать не и чего
			    raise info 'not founded good from good_sum_mart, %', goodssum;
			END IF;
			return old;
			
	END CASE;
	
END;
$BODY$;

ALTER FUNCTION pract_functions.sales_insert_update_delete_tf()
    OWNER TO postgres;
```

> Сам триггер:

> ```sql
> CREATE TRIGGER tr_sales
> BEFORE INSERT OR DELETE OR UPDATE 
> ON pract_functions.sales
> FOR EACH ROW
> EXECUTE FUNCTION pract_functions.sales_insert_update_delete_tf();
> ```
### Демонстрация работы триггера
##### Сначала заполним витрину 
>
> ```sql
> INSERT INTO pract_functions.good_sum_mart (good_name, sum_sale)
> (
> 	SELECT g.good_name, sum(g.good_price * s.sales_qty)
>  	FROM pract_functions.goods g, pract_functions.sales s
> 	WHERE g.goods_id=s.good_id
> 	GROUP BY g.good_name
> )
> ```
>
> Посмотрим данные в витрине
> ```sql
> SELECT * FROM pract_functions.good_sum_mart
> ```
>
> <image src="images/first.png" alt="first">


##### Проверим INSERT:
> ```sql
> INSERT INTO pract_functions.sales (good_id, sales_qty) VALUES (1, 5)
> ```
> Сумма по спичкам должна увеличится на 2.5
> ```sql
> SELECT * FROM pract_functions.good_sum_mart
> ```
> Результат:
>
> <image src="images/insert.png" alt="insert">
>
> Действительно увеличились на 2.5

##### Проверим UPDATE. 
>
> Сначала посмотрим, что есть в таблице sales
> 
>
> ```sql
> SELECT * FROM pract_functions.sales
> ORDER BY sales_id ASC
> ```
>
> <image src="images/sales.png" alt="sales">
>
> Обновим запись с sales_id = 20
> ```sql
> update pract_functions.sales SET sales_qty = 3
> where sales_id = 20
> ```
> Сумма по спичкам должна уменьшится на 1
>
> <image src="images/update.png" alt="update">
>
> Действительно уменьшилась на 1

##### Проверим DELETE:
> ```sql
> delete from pract_functions.sales
> where sales_id = 20
> ```
> Сумма по спичкам должна уменьшится на 1.5
> ```sql
> SELECT * FROM pract_functions.good_sum_mart
> ```
> Результат:
>
> <image src="images/delete.png" alt="delete">
>
> Действительно уменьшилась на 1.5
### Задание со звездочкой*. Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)? Подсказка: В реальной жизни возможны изменения цен.
> Витрина+триггер предпочтительнее, само собой, производительностью, т.к. select из готовой таблицы выполнится гораздо быстрее, чем при вычислении суммы в самом select.
>
> Т.к. в реальной жизни возможно изменение цен товаров, то в текущем виде таблиц sales и goods, когда цена товара хранится в таблице goods, отчет, вычисляющий сумму товаров, будет вычилсять сумму по текущей цене, а не по реальной цене, по которой они были проданы. Следовательно, отчет, вообще, в данной конфигурации не совсем корректно работает.
>
> Поэтому, чтобы решить эту проблему, я бы предложил хранить цену товара good_price в непосредственно в таблице sales. Тогда отчет вычисляющий сумму продаж по товарам, будет брать реальную цену из таблицы sales, а не текущую из таблицы goods.
> 
> И вообще говоря, в схеме витрина+триггер при UPDATE или DELETE тоже будет такая же проблема, которая решается также добавлением цены товара good_price в непосредственно в таблицу sales. Причем нужно хранить еще и текущую цену в таблице goods, чтобы правильно вычислить сумму при UPDATE или DELETE. 
>
> А вот в случае INSERT в схеме витрина+триггер проблемы нет, работать будет корректно.