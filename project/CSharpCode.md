## Описание C#-приложения


### Для каждой таблицы формируем список полей с типом каждого поля.
Это позволяет не создавать вручную классы для каждоой таблицы, а формировать список полей с типами динамически.
Также позволяет не переписывать приложение, если в таблице добавилось или удалилось какое-то поле.

```cs
internal async Task<MSRes> GetFieldNameAndTypeByTable(CTableName tablename, List<CFieldNameAndType> FieldNameAndTypeList)
{
	msres.ResBool = false;
	msres.ErrmMess = "";

	using SqlCommand command = new SqlCommand(@"SELECT COLUMN_NAME, DATA_TYPE 
	                                            FROM INFORMATION_SCHEMA.COLUMNS
	                                            WHERE TABLE_NAME = '" + tablename.Name + "'", conn);
	{
		SqlDataReader reader = await command.ExecuteReaderAsync();

		try
		{
			while (await reader.ReadAsync())
			{
				FieldNameAndTypeList.Add(new CFieldNameAndType()
				{
					FieldName = reader.GetString(0),
					TypeName = reader.GetString(1)
				});
			}

			msres.ResBool = true;
			msres.ErrmMess = "";
		}
		catch (Exception e)
		{
			msres.ResBool = false;
			msres.ErrmMess = e.Message;
		}
		finally
		{
			reader.Close();
		}
	}           
	
	return msres;
}
```

### Миграция отдной таблицы   
```cs
while (true)
{
    // Формируем текст запроса SELECT к БД MS SQL
	GetQueryByList(ind, tablename, mincount, maxcount, fieldNameAndTypeList, ref selquery);

	try
	{	
	    // Выполняем SELECT, записываем результат в .csv-файл и из .csv пишем в БД PostgreSQL
		modelres = await SaveToCSVandCopy(tablename, rowcount, fieldNameAndTypeList, selquery);
	}
	catch (Exception ex)
	{
		modelres.Mess = ex.Message;
		return modelres;
	}	
	
	if ((modelres.Code == (Int32)ERR.NO_DATA) || (modelres.Code != (Int32)ERR.OK))
	{		
		return modelres;
	}

	Double prbarpercent = ((Double)(modelres.InsertedRow) / (Double)rowcount) * 100;



	PrgrAndCaptionSet(progress, prbarpercent.ToString(), "Таблица: [" + tablename.Name + "], мигрировано строк: " +
														  modelres.InsertedRow.ToString() + " из " + rowcount.ToString());

	ind++;
	
	if ((modelres.Code == (Int32)ERR.NO_DATA) || (modelres.Code != (Int32)ERR.OK))
	{		
		return modelres;
	}	

}
```

### Формируем SELECT на выборку данных
```cs
internal static Boolean GetQueryByList(Int32 ind, CTableName tablename, Int64 minrow, Int64 maxrow, List<CFieldNameAndType> fieldNameAndTypeList, ref string selquery)
{
	string fields = "";
	string paramemeters = "";
	string updtField = "";
	string firstField = "";

	selquery = "";
	

	if (fieldNameAndTypeList.Count > 0)
	{
				
		if (tablename.IsLargeTable)
		{
			Int64 maxrowcalc = (minrow + TopSel * (ind) - 1);
			if (maxrowcalc > maxrow)
			{
				maxrowcalc = maxrow;
			}
			selquery = "SELECT " + fields +
			           " FROM " + tablename.Name +
		               " WHERE " + tablename.KeyField + " BETWEEN " + (minrow + TopSel * (ind - 1)).ToString() + " AND " +
																	  maxrowcalc;

		}
		else
		{
			selquery = "SELECT " + fields +
		               " FROM " +
			           " (SELECT " + fields + ", ROW_NUMBER() OVER(ORDER BY " + firstField + ") as row_number " +
			           " FROM " + tablename.Name + ") SEL" +
			           " WHERE SEL.row_number BETWEEN " + (TopSel * (ind - 1) + 1).ToString() + " and " + (TopSel * ind).ToString();
		}

	}

	return true;
}
```

### Выполняем сформированный SELECT, записываем результат в .CSV-файл и с помощью команды COPY записываем данные в PostgreSQL
```cs
internal static async Task<ModelRes> SaveToCSVandCopy(CTableName tablename, Int64 rowcount, List<CFieldNameAndType> fieldNameAndTypeList, string selquery)
{
	modelres.Code = (Int32)ERR.NO_DATA;
	modelres.Mess = "";
	int insertedrow = 0;
	SqlDataReader reader = null;

	using SqlCommand selcommand = new SqlCommand(selquery, ms.conn);
	{
		try
		{
			reader = await selcommand.ExecuteReaderAsync();
		}
		catch (Exception e)
		{
			modelres.Code = (Int32)ERR.MS_CREATE_READER;
			modelres.Mess = e.Message;
			return modelres;
		}

		string csvfileName = Directory.GetCurrentDirectory() + Path.DirectorySeparatorChar + tablename.Name + ".csv";
		if (File.Exists(csvfileName))
		{
			File.Delete(csvfileName);
		}

		using StreamWriter sw = new StreamWriter(csvfileName, false);
		{
			for (int i = 0; i < reader.FieldCount; i++)
			{
				sw.Write(reader.GetName(i));
				if (i < reader.FieldCount)
				{
					sw.Write(",");
				}

			}
			sw.Write(sw.NewLine);

			bool NoData = true;
			try
			{
				while (await reader.ReadAsync())
				{
					NoData = false;
					for (int i = 0; i < reader.FieldCount; i++)
					{

						if (!Convert.IsDBNull(reader.GetSqlValue(i)))
						{
							string value = reader.GetValue(i).ToString();
							object obj = null;

							FieldTypePars(i, reader, fieldNameAndTypeList[i], ref obj);
							value = obj.ToString().Trim();

							if ((value.Contains(",")) || (value.Contains("\"")))
							{
								value = String.Format("\'{0}\'", value);
								sw.Write(value);
							}
							else
							{
								sw.Write(value);
							}
						}
						if (i < reader.FieldCount - 1)
						{
							sw.Write(",");
						}
					}
					sw.Write(sw.NewLine);
					insertedrow = insertedrow + 1;
					if ((insertedrow >= rowcount) || NoData)
					{
						break;
					}
				}

			}
			catch (Exception e)
			{
				modelres.Code = (Int32)ERR.MS_SEL;
				modelres.Mess = e.Message;
			}
			finally
			{
				if (reader != null)
					if (!reader.IsClosed)
						reader.Close();
			}



			sw.Close();
			if (tablename.ReplacedFiled)
			{
				var delres = await pg.ExecSqlQuery(" delete from " + tablename.Name, pg.conn, null);
				if (!delres.ResBool)
				{
					modelres.Code = (Int32)ERR.PG_INSERT;
					modelres.Mess = delres.ErrmMess;
					return modelres;
				}
			}
			var execres = await pg.ExecSqlQuery(" COPY " + tablename.Name +
			                                   @" FROM '" + csvfileName + "'" +
			                                   @" DELIMITER ',' CSV QUOTE '''' NULL AS 'Null' header ;", pg.conn, null);

			if (!execres.ResBool)
			{
				modelres.Code = (Int32)ERR.PG_INSERT;
				modelres.Mess = execres.ErrmMess;
				return modelres;
			}

			File.Delete(csvfileName);


		}

	}
	modelres.InsertedRow = modelres.InsertedRow + insertedrow;
	if (tablename.IsLargeTable)
	{
		if (modelres.InsertedRow < rowcount)
		{
			modelres.Code = (Int32)ERR.OK;
			modelres.Mess = "";
		}
	}
	else
	if (insertedrow == TopSel)
	{
		modelres.Code = (Int32)ERR.OK;
		modelres.Mess = "";
	}

	return modelres;
}
```