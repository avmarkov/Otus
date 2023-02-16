## Миграция хранимых процедур

### 1. Функция расчета полноты собранных данных по группе приборов

#### Исходная функция расчета полноты собранных данных по группе в MS SQL:

```sql
ALTER function [dbo].[GetAVGFullnessByGroup]( @Groups_ID int, @Date1 date, @Date2 date )
returns table
as
return
(
	WITH TestCTE(Groups_ID, Groups_Name, Groups_Parent_ID)
	AS
	(	
		-- Находим якорь рекурсии	
		SELECT Groups_ID, Groups_Name, Groups_Parent_ID
		FROM dbo.Groups WHERE Groups_ID = @Groups_ID
		UNION ALL
		--Делаем объединение с TestCTE
		SELECT t1.Groups_ID, t1.Groups_Name, t1.Groups_Parent_ID 
		FROM dbo.Groups t1 
		JOIN TestCTE t2 ON t1.Groups_Parent_ID=t2.Groups_ID
	)

	SELECT AVGFullness 
	FROM (
			SELECT  
			AVG( CASE 
					 WHEN u.Units_ID IS NOT NULL AND UnitSWithPercent.Units_ID IS NULL THEN  0
					 WHEN u.Units_ID IS NOT NULL AND UnitSWithPercent.Units_ID IS NOT NULL THEN  UnitSWithPercent.Date1PercentGood
				 END) AS AVGFullness

			FROM TestCTE T 

			INNER JOIN GroupsDevices GD ON T.Groups_ID = GD.GroupsDevices_Groups_ID
			LEFT JOIN Units U ON U.Units_Devices_ID = GD.GroupsDevices_Devices_ID AND U.Units_GuideEnergy_Code = 0
			LEFT JOIN
						(SELECT Units_ID, MIN(Date1) AS Date1,
						        ROUND(CAST(COUNT(Date1) AS real)*100/(DATEDIFF(day, @Date1, @Date2)+1), 2) AS Date1PercentGood
						 FROM
						(
							SELECT DISTINCT U.Units_ID, DR1.DataRecord_Date AS Date1
							FROM Units U
							LEFT JOIN DataRecord DR1 ON (DR1.DataRecord_Units_ID = U.Units_ID AND
							                             DR1.DataRecord_GuideDataType_Code = 3 AND
							                             U.Units_GuideEnergy_Code = 0)
							WHERE  DR1.DataRecord_Date BETWEEN @Date1 AND @Date2				
						) AS tt		
			GROUP BY  Units_ID
			) UnitSWithPercent ON UnitSWithPercent.Units_ID = U.Units_ID
	) rtt
)	
```

#### Результат
```sql
SELECT * FROM [dbo].[GetAVGFullnessByGroup] (
   2563,
   '01.12.2021',
   '31.12.2021')
```

<image src="images/resfun1_ms.png" alt="resfun1_ms">

#### Аналогичная функция расчета полноты собранных данных по группе приборов в PostgreSQL:  
```sql

CREATE OR REPLACE FUNCTION public.getavgfullnessbygroup(
	groupsid integer,
	date_1 date,
	date_2 date)
    RETURNS TABLE(avgfullness double precision) 
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
    ROWS 1000

AS $BODY$

BEGIN

RETURN query 
(
	WITH RECURSIVE TestCTE(Groups_ID, Groups_Name, Groups_Parent_ID)
	AS
	(	
		-- Находим якорь рекурсии	
		SELECT Groups_ID, Groups_Name, Groups_Parent_ID
		FROM public.Groups WHERE Groups_ID = groupsid
		
		UNION ALL
		
		--Делаем объединение с TestCTE (хотя мы его еще не дописали)
		SELECT t1.Groups_ID, t1.Groups_Name, t1.Groups_Parent_ID 
		FROM public.Groups t1 
		JOIN TestCTE t2 ON t1.Groups_Parent_ID = t2.Groups_ID
	)

SELECT cast(rtt.avgfullness as double precision)  
FROM (

		SELECT  
		AVG( 
			  CASE 
				 WHEN (u.Units_ID IS NOT NULL) AND (UnitSWithPercent.Units_ID IS NULL)  THEN  0
				 WHEN (u.Units_ID IS NOT NULL) AND (UnitSWithPercent.Units_ID IS NOT NULL)  THEN  UnitSWithPercent.Date1PercentGood
			  END
		    ) AS avgfullness

		FROM TestCTE T 
	
		INNER JOIN GroupsDevices GD ON T.Groups_ID = GD.GroupsDevices_Groups_ID
		LEFT JOIN Units U ON U.Units_Devices_ID = GD.GroupsDevices_Devices_ID AND U.Units_GuideEnergy_Code = 0
		LEFT JOIN

		(SELECT Units_ID, MIN(Date1) AS Date1, 
		        ROUND(CAST(
		                    CAST(COUNT(date1) AS double precision)*100/(DATE_PART('day', date_2::timestamp - date_1::timestamp) + 1)
		                    as numeric
		                   ), 2  
		             )  AS Date1PercentGood
		 FROM
			 (
			    SELECT DISTINCT U.Units_ID, DR1.DataRecord_Date AS Date1
				FROM Units U
				LEFT JOIN DataRecord DR1 ON DR1.DataRecord_Units_ID = U.Units_ID AND
				                            DR1.DataRecord_GuideDataType_Code = 3 AND
				                            U.Units_GuideEnergy_Code = 0
				WHERE  DR1.DataRecord_Date BETWEEN date_1  AND date_2						
			  ) AS tt
		
		GROUP BY  Units_ID
		) UnitSWithPercent
		ON UnitSWithPercent.Units_ID = U.Units_ID
	) rtt
);
END; 

$BODY$;
```

#### Результат
```sql
SELECT public.getavgfullnessbygroup(
	2563, 
	'01.12.2021',
    '31.12.2021'
)
```

<image src="images/resfun1_pg.png" alt="resfun1_pg">

### 1. Функция парсинга строки

Входная трока имеет следующий вид:
0=43=1;1=11111=1;2=44454=1;3=102654834=1;4=402222=1;5=0=1;6=0=1;7=1=1;8=7084=1;9=7084=1;10=1760113=1;11=0=1;12=1=1;13=4=1;14=0=1;15=0=1;16=3415=1;18=16=1;

#### Функция парсинга строкив MS SQL: 

```sql

ALTER FUNCTION [dbo].[Web_C2_RA4_GetRole_f]
(
	@STR VARCHAR(MAX)
)
RETURNS int
AS
BEGIN
	DECLARE @STR_ID VARCHAR(max), @STR_VAL VARCHAR(max) 
    DECLARE @TEMP_TAB table(ID int, VAL NVARCHAR(max))
    
    WHILE @STR <>''BEGIN
					    SET @STR_VAL = LEFT(@STR,CHARINDEX(';',@STR)-1)
					    SET @STR_ID = LEFT(@STR_VAL,CHARINDEX('=',@STR_VAL)-1)
					    SET @STR_VAL = RIGHT(@STR_VAL,LEN(@STR_VAL)-CHARINDEX('=',@STR_VAL))
					    SET @STR_VAL = LEFT(@STR_VAL,CHARINDEX('=',@STR_VAL)-1)
					    INSERT INTO @TEMP_TAB SELECT @STR_ID, @STR_VAL           
					    SET @STR = RIGHT(@STR,LEN(@STR)-CHARINDEX(';',@STR))
                    END
	RETURN (Select VAL From @TEMP_TAB Where ID = 18)

END

```

#### Результат:
```sql
SELECT * FROM [dbo].[Web_C2_RA4_GetDetailParams_f] (901)
```

<image src="images/resfun2_ms.png" alt="resfun2_ms">

#### Функция парсинга строкив PostgreSQL: 

```sql

CREATE OR REPLACE FUNCTION public.web_c2_ra4_getdetailparams_f(
	deviceid integer)
    RETURNS TABLE(id integer, val character varying) 
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
    ROWS 1000

AS $BODY$

DECLARE
STR character varying;
STR_ID character varying;
STR_VAL character varying; 
BEGIN

    SELECT Devices.devices_connectioninfo INTO STR
	FROM public.Devices
	WHERE Devices.devices_id = deviceid;
	
	WHILE STR <>'' 
	LOOP	
		STR_VAL = LEFT(STR, strpos(STR, ';')-1);
		STR_ID = LEFT(STR_VAL, strpos(STR_VAL, '=') -1);
		STR_VAL = RIGHT(STR_VAL, LENGTH(STR_VAL) - strpos(STR_VAL, '='));
		STR_VAL = LEFT(STR_VAL, strpos(STR_VAL, '=')-1);
		
		
		ID := cast(STR_ID as integer);
		VAL := STR_VAL;		
		STR = RIGHT(STR, LENGTH(STR) - strpos(STR, ';'));
		RETURN next; 		
    END LOOP;	
END;

$BODY$;

```

#### Результат:
```sql
SELECT * from public.web_c2_ra4_getdetailparams_f(901)
```

<image src="images/resfun2_pg.png" alt="resfun2_pg">