## oracle schema 管理
[reference](http://www.dba-oracle.com/t_ddl_triggers.htm)
```
drop TRIGGER RPT_AUDIT_DDL_AFTER_DDL;
drop TRIGGER RPT_AUDIT_DDL_BEFORE_INSERT;
drop SEQUENCE RPT_AUDIT_DDL_SEQUENCE;
drop TABLE RPT_AUDIT_DDL; 

-- 记录ddl信息
CREATE TABLE RPT_AUDIT_DDL (
  ID INTEGER NOT NULL,
  DATA_DATE DATE,
  OS_USER VARCHAR2(255),
  CURRENT_USER VARCHAR2(255),
  HOST VARCHAR2(255),
  TERMINAL VARCHAR2(255),
  OWNER VARCHAR2(30),
  TYPE VARCHAR2(30),
  NAME VARCHAR2(100),
  SYS_EVENT VARCHAR2(30));

ALTER TABLE RPT_AUDIT_DDL ADD (CONSTRAINT RPT_AUDIT_DDL_PK PRIMARY KEY (ID));

CREATE SEQUENCE RPT_AUDIT_DDL_SEQUENCE;

CREATE TRIGGER RPT_AUDIT_DDL_BEFORE_INSERT
  BEFORE INSERT ON RPT_AUDIT_DDL
  FOR EACH ROW
BEGIN
  SELECT RPT_AUDIT_DDL_SEQUENCE.nextval
  INTO :new.id
  FROM dual;
END;

create trigger RPT_AUDIT_DDL_AFTER_DDL 
  after ddl on schema
begin
  if (ora_sysevent='TRUNCATE' or ora_sysevent='GRANT' or ora_dict_obj_name='RPT_AUDIT_DDL')
  then
    null; -- I do not care about truncate
  else
    insert into RPT_AUDIT_DDL(data_date, os_user,current_user,host,terminal,owner,type,name,sys_event)
    values(
      sysdate,
      sys_context('USERENV','OS_USER') ,
      sys_context('USERENV','CURRENT_USER') ,
      sys_context('USERENV','HOST') ,
      sys_context('USERENV','TERMINAL') ,
      ora_dict_obj_owner,
      ora_dict_obj_type,
      ora_dict_obj_name,
      ora_sysevent
    );
  end if;
end;

--全量schema信息视图
create or replace view v_schema as 
select name,decode(table_type,'TABLE',0,'VIEW',1) as physical_type,comments as "comment",column_desc
from 
( select name,wm_concat(col) as column_desc
  -- xmlagg(xmlparse(content col||',' wellformed) order by column_id).getclobval() as column_desc
  from
  ( select lower(t1.table_name) as name,t1.column_id,'{"name":"'|| lower(t1.column_name) || '","type":"' || t1.data_type || '","comment":"' || t2.comments || '","nullable":' || decode(t1.nullable,'N','false','true') || '}' col
    from
    ( select table_name,column_id,column_name,
      case when data_type in ('VARCHAR2','NVARCHAR2','CHAR','NVCHAR') then data_type || '(' || data_length || ')' 
        when data_type = 'NUMBER' and data_scale is not null then data_type || '(' || nvl2(data_precision,to_char(data_precision),'*') || ','|| data_scale || ')'
        else data_type end as data_type,nullable 
      from user_tab_columns 
      where substr(table_name ,0, 4)!='SYS_' 
      and table_name not in ('表名')
    ) t1
    left outer join
    ( select table_name,column_name,comments from all_col_comments 
      where owner='用户名' 
      and substr(table_name ,0, 4)!='SYS_' 
      and table_name not in ('表名')
    ) t2
    on t1.table_name=t2.table_name and t1.column_name=t2.column_name
    order by t1.table_name,t1.column_id
  ) t
  group by name
) t3 
left outer join user_tab_comments t4
on t3.name=lower(t4.table_name);
```