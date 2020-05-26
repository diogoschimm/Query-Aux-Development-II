# Query-Aux-Development-II
Contém Querys utilizando tabelas de sistema do SQL Server para Gerar Comandos

## Comando
Utilize as tabelas de sistema (sys.tables, sys.columns e sys.types) para gerar um comando de criação de tabelas de LOG através de um Cursor do T-SQL.

```sql
declare @object_id as int, @table_name varchar(1000)

declare c_tabelas cursor for select object_id, name from sys.tables 
				 where name not in ('__RefactorLog', 'sysdiagrams')
				 order by name 

open c_tabelas
fetch next from c_tabelas into @object_id, @table_name
while @@FETCH_STATUS = 0 begin


	print 'CREATE TABLE LOG_' + @table_name + ' ('

	declare @column_name as varchar(1000), 
			@column_id int, 
			@column_type_id int, 
			@max_length int, 
			@precision int, 
			@scale int, 
			@type_name as varchar(1000), 
			@type_app varchar(1000)

	declare c_colunas cursor for select c.name, 
                  c.column_id, 
                  c.system_type_id, 
                  c.max_length, 
                  c.precision, 
                  c.scale, 
                  tp.name as type_name,
                  case when c.system_type_id in (167) then 
                      'varchar(' + 
                        case when c.max_length = -1 then 
                            'max' 
                           else 
                            cast(c.max_length AS varchar(10)) 
                        end +  ')' 
                     when c.system_type_id in (106) then 
                        tp.name + '(' + cast(c.precision as varchar(10)) + ', ' + cast(c.scale as varchar(10)) + ')' 
                     else 
                        tp.name 
                  end as type_app

              from sys.columns c
              join sys.types tp on c.system_type_id = tp.system_type_id
              where object_id = @object_id
              order by column_id
	open c_colunas
	fetch next from c_colunas into @column_name, @column_id, @column_type_id, @max_length, @precision, @scale, @type_name, @type_app
	while @@FETCH_STATUS = 0 begin

		print '	' + @column_name + ' ' + @type_app + ' NULL, '

		fetch next from c_colunas into @column_name, @column_id, @column_type_id, @max_length, @precision, @scale, @type_name, @type_app
	end 
	close c_colunas
	deallocate c_colunas
	
	print '	LOG_SESSION_idUsuario INT NULL, '
	print '	LOG_DATA DATETIME NOT NULL, '
	print '	LOG_BD_usuario varchar(256) NOT NULL, '
	print '	LOG_hostname varchar(256) NOT NULL, '
	print '	LOG_operacao varchar(1) NULL'

	print ');'
	print 'GO'

	fetch next from c_tabelas into @object_id, @table_name
end
close c_tabelas
deallocate c_tabelas
 
```

## Log na tabela principal (dados de criação e última modificação)

```tsql
ALTER TABLE TBL_NAME
ADD CreatedById int,
	CreationDate datetime,
	CreatedByName varchar(100),
	ModifiedById int,
	ModifiedDate datetime,
	ModifiedByName varchar(100)
```
