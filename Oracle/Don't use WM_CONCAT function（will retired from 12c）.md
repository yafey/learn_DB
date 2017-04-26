
# Oracle will retire WM_CONTACT Function from lastest 12c version
>https://blogs.oracle.com/oracle-database-app-admin/entry/why_not_use_wm_concat

## Oracle 12c upgrade problems ##
>http://www.dba-oracle.com/t_oracle12c_upgrade_problems.htm

- The `wm_concat` procedure has been deprecated. **This was easy by using `listagg` as a replacement for `wm_concat`.**

### Replacement for `wm_contact` , like : `listagg` etc. （更多替代方案见该网址中的回答。）
>http://stackoverflow.com/questions/16674927/why-does-the-wm-concat-not-work-here
>You must avoid `wm_concat` function because it is undocumented and discovered as workaround at Oracle 8i times.

Workaround 1 - LISTAGG function, works in 11g:
```
select listagg(object_id,',') within group (order by rownum) id_string
from cr_object_group_entries_vw
```


Workaround 2 - SYS_CONNECT_BY_PATH, works since 10g:
```
select id_string from (
  select rn, substr(sys_connect_by_path(object_id, ','),2) id_string
  from (select object_id, rownum rn from cr_object_group_entries_vw)
  start with rn = 1
  connect by prior rn + 1 = rn
  order by rn desc
)
where rownum = 1

```
Workaround 3 - XMLAGG, works since 10g:
```

select replace(
         replace(
           replace(
             xmlagg(xmlelement("x",object_id)).getStringVal(),
             '</x><x>',
             ','
           ),
           '<x>',
           ''
         ),
         '</x>',
         ''
       ) id_string
from cr_object_group_entries_vw
```

P.S. I didn't know exactly in which Oracle versions sys_connect_by_path and xmlagg was introduced, but both works well on 10.2.0.4.0
