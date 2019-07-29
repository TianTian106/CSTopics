## oracle schema 管理
reference: 
* [Oracle DDL Triggers](http://www.dba-oracle.com/t_ddl_triggers.htm)
* [Oracle Regular Expressions](https://docs.oracle.com/cd/E11882_01/appdev.112/e41502/adfns_regexp.htm#ADFNS231)

```
drop TRIGGER AUDIT_TRIGGER_AFTER_DDL;
drop TRIGGER AUDIT_TRIGGER_BEFORE_INSERT;
drop SEQUENCE SCHEMA_AUDIT_DDL_SEQUENCE;
drop TABLE SCHEMA_AUDIT_DDL; 


-- 记录审计信息
CREATE TABLE SCHEMA_AUDIT_DDL (
  ID INTEGER NOT NULL,
  EVENT_TIME DATE,
  OS_USER VARCHAR2(255),
  CURRENT_USER VARCHAR2(255),
  HOST VARCHAR2(255),
  TERMINAL VARCHAR2(255),
  OWNER VARCHAR2(30),
  TYPE VARCHAR2(30),
  NAME VARCHAR2(100),
  OLD_NAME VARCHAR2(100),
  OP_TYPE VARCHAR2(30),
  OP_CONTEXT CLOB
);

ALTER TABLE SCHEMA_AUDIT_DDL ADD (CONSTRAINT SCHEMA_AUDIT_DDL_PK PRIMARY KEY (ID));

CREATE SEQUENCE SCHEMA_AUDIT_DDL_SEQUENCE;

CREATE TRIGGER AUDIT_TRIGGER_BEFORE_INSERT
  BEFORE INSERT ON SCHEMA_AUDIT_DDL
  FOR EACH ROW
BEGIN
  SELECT SCHEMA_AUDIT_DDL_SEQUENCE.nextval
  INTO :new.id
  FROM dual;
END;


--记录审计信息触发trigger
--识别sql时注意去掉注释
CREATE OR REPLACE TRIGGER AUDIT_TRIGGER_AFTER_DDL
  after ddl on schema
DECLARE
  sql_text ora_name_list_t;
  op_context CLOB;
  n PLS_INTEGER;
begin
  if (ora_sysevent!='TRUNCATE' and ora_sysevent!='GRANT' and ora_dict_obj_name not in ('SCHEMA_AUDIT_DDL','V_SCHEMA') and ora_dict_obj_type in ('TABLE','VIEW','COLUMN'))
  then 
    n := ora_sql_txt(sql_text);
    FOR i IN 1..n LOOP
      op_context := op_context || sql_text(i);
    END LOOP;
    if (ora_sysevent='ALTER' and regexp_like(regexp_replace(op_context, '--.*','',1, 0, 'i'),'[[:space:]]*alter[[:space:]]+table[[:space:]]+[a-z0-9_]+[[:space:]]+rename[[:space:]]+to[[:space:]]+[a-z0-9_]+[[:space:]]*','i'))
    then 
        insert into SCHEMA_AUDIT_DDL(event_time,os_user,current_user,host,terminal,owner,type,name,old_name,op_type,op_context)
        values(
          sysdate,
          sys_context('USERENV','OS_USER') ,
          sys_context('USERENV','CURRENT_USER') ,
          sys_context('USERENV','HOST') ,
          sys_context('USERENV','TERMINAL') ,
          ora_dict_obj_owner,
          ora_dict_obj_type,
          replace(lower(regexp_replace(regexp_replace(op_context, '--.*','',1, 0, 'i'), '[[:space:]]*alter[[:space:]]+table[[:space:]]+[a-z0-9_]+[[:space:]]+rename[[:space:]]+to[[:space:]]+([a-z0-9_]+)[[:space:]]*', '\1', 1, 0, 'i' )),chr(0),''),
          replace(lower(regexp_replace(regexp_replace(op_context, '--.*','',1, 0, 'i'), '[[:space:]]*alter[[:space:]]+table[[:space:]]+([a-z0-9_]+)[[:space:]]+rename[[:space:]]+to[[:space:]]+[a-z0-9_]+[[:space:]]*', '\1', 1, 0, 'i' )),chr(0),''),
          'RENAME',
          replace(op_context,chr(0),'')
        );
    elsif (ora_sysevent='ALTER' and regexp_like(op_context,'alter[[:space:]]+table[[:space:]]+rpt\.[a-z0-9_]+[[:space:]]+(drop|add)[[:space:]]+partition[[:space:]]+part_[0-9_]+','i'))
    then
        null;
    else 
        insert into SCHEMA_AUDIT_DDL(event_time, os_user,current_user,host,terminal,owner,type,name,old_name,op_type,op_context)
        values(
          sysdate,
          sys_context('USERENV','OS_USER') ,
          sys_context('USERENV','CURRENT_USER') ,
          sys_context('USERENV','HOST') ,
          sys_context('USERENV','TERMINAL') ,
          ora_dict_obj_owner,
          ora_dict_obj_type,
          lower(regexp_replace(ora_dict_obj_name, '([a-z0-9_]+)\.[a-z0-9_]+', '\1', 1, 1, 'i')),
          null,
          ora_sysevent,
          replace(op_context,chr(0),'')
        );
    end if;
  end if;
end;


--全量schema信息视图
create or replace view v_schema as
select t3.table_name,nvl(table_type,'VIEW') table_type,comments,'{"columns":[' || column_desc || ']}' as column_desc,nvl(audit_id,0) as audit_id
from
( select table_name,wm_concat(col) as column_desc
  -- xmlagg(xmlparse(content col||',' wellformed) order by column_id).getclobval() as column_desc
  from
  ( select lower(t1.table_name) as table_name,t1.column_id,'{"name":"'|| lower(t1.column_name) || '","type":"' || t1.data_type || '","comment":"' || t2.comments || '","nullable":' || decode(t1.nullable,'N','false','true') || '}' col
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
    ( select table_name,column_name,replace(replace(replace(regexp_replace(comments,'\\','\\\\'),chr(10),'\\n'),chr(13),'\\n'),chr(9),'\\t') as comments from all_col_comments
      where owner='用户名' 
      and substr(table_name ,0, 4)!='SYS_' 
      and table_name not in ('表名')
    ) t2
    on t1.table_name=t2.table_name and t1.column_name=t2.column_name
    order by t1.table_name,t1.column_id
  ) t
  group by table_name
) t3
left outer join user_tab_comments t4
on t3.table_name=lower(t4.table_name)
cross join (select max(id) audit_id from schema_audit_ddl) t5;
```