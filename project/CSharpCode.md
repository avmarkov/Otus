## Описание C#-приложения

### Для каждой таблицы формируем список полей с типом каждого поля:
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