## Миграция хранимых процедур

### Исходная функция нахождения полноты собранных данных по группе в MS SQL:

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
           AVG( 
                    CASE 
                         WHEN u.Units_ID IS NOT NULL AND UnitSWithPercent.Units_ID IS NULL THEN  0
                         WHEN u.Units_ID IS NOT NULL AND UnitSWithPercent.Units_ID IS NOT NULL THEN  UnitSWithPercent.Date1PercentGood
                    END) AS AVGFullness

          FROM TestCTE T 

          INNER JOIN GroupsDevices GD ON T.Groups_ID = GD.GroupsDevices_Groups_ID
          LEFT JOIN Units U ON U.Units_Devices_ID = GD.GroupsDevices_Devices_ID AND U.Units_GuideEnergy_Code = 0
          LEFT JOIN
          (SELECT Units_ID, MIN(Date1) AS Date1, ROUND(
		                                                   CAST(COUNT(Date1) AS real)*100/
		                                                   (DATEDIFF(day, @Date1, @Date2)+1), 
													   2) AS Date1PercentGood
           FROM
               (SELECT DISTINCT U.Units_ID, DR1.DataRecord_Date AS Date1
                FROM Units U
                LEFT JOIN DataRecord DR1 ON (DR1.DataRecord_Units_ID = U.Units_ID AND
                                             DR1.DataRecord_GuideDataType_Code = 3 AND
                                             U.Units_GuideEnergy_Code = 0)
                WHERE  DR1.DataRecord_Date BETWEEN @Date1  AND @Date2                              
               ) AS tt          
           GROUP BY  Units_ID
          ) UnitSWithPercent ON UnitSWithPercent.Units_ID = U.Units_ID
     ) rtt
)
```

### Аналогичная функция  нахождения полноты собранных данных по группе приборов в PostgreSQL:  
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
          INNER JOIN GroupsDevices GD 
          ON T.Groups_ID = GD.GroupsDevices_Groups_ID

          LEFT JOIN Units U
          ON U.Units_Devices_ID = GD.GroupsDevices_Devices_ID AND U.Units_GuideEnergy_Code = 0

          LEFT JOIN

          (SELECT Units_ID, MIN(Date1) AS Date1, 
                  ROUND(
				        CAST(
						      CAST(COUNT(date1) AS double precision)*100/
							 (DATE_PART('day', date_2::timestamp - date_1::timestamp) + 1) 
							  as numeric 
							) , 2
						)  AS Date1PercentGood
           FROM
                (
                    SELECT DISTINCT U.Units_ID, DR1.DataRecord_Date AS Date1
                    FROM Units U
                    LEFT JOIN DataRecord DR1 ON ((DR1.DataRecord_Units_ID = U.Units_ID) AND
                                                 (DR1.DataRecord_GuideDataType_Code = 3) AND
                                                 (U.Units_GuideEnergy_Code = 0))
                    WHERE  DR1.DataRecord_Date BETWEEN date_1  AND date_2                              
                 ) AS tt

          
          GROUP BY  Units_ID
          ) UnitSWithPercent     ON UnitSWithPercent.Units_ID = U.Units_ID
     ) rtt

);
END; 

$BODY$;


```
