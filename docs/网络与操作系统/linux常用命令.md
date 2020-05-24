# linux常用命令


### 日志分析

##### grep显示关键字前后几行信息

* grep -C n  keyWord log.file 显示日志文件log.file文件里匹配到keyWord的前后n行
* grep -B n  keyWord log.file 显示关键字的前n行
* grep -A n  keyWord log.file 显示关键字的后n行

#### cat显示文件的某几行
* cat log.file | tail -n +1000 | head -n 2000 
>从第1000行开始，显示2000行。即显示1000~2999行


