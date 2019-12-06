---
title: liunx查询日志常用命令收集
date: 2019-09-17
tag: liunx
---

每次查日志很多命令都记不熟，在这里做个整理吧

 ### 是否有新增记录  tail

- **tail -400f demo.log**        
    #demo.log后400行日志 

### 关键记录查找 grep

- **grep '5033' demo.log **      
    #查找demo.log下所有匹配'5033'的行 

-  **grep -o 'order-fix.curr_id:\([0-9]\+\)' demo.log**    
      #查找并提取（不是一行）匹配正则表达式的内容 

- **grep -c  'ERROR' demo.log**      
     #输出所有包含‘ERROR'的行的数量 

- **grep -v 'ERROR' demo.log**       
    #输出所有不包含‘ERROR’的行 

-   或操作：包含123或者abc的行 

     **grep -E '123|abc' demo.log**

-  与操作：既包含123并且包含abc的行 

    **grep '123' demo.log | grep 'abc'**

  

  

  

