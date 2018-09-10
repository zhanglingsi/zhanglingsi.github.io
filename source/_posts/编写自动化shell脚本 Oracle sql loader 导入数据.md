---
title: 编写自动化shell脚本 Oracle sql loader 导入数据 
categories: 编程
copyright: true
---

# 编写自动化shell脚本 Oracle sql loader 导入数据

## 需求
1. 调度周期：日调度 
2. 数据源：ODS系统早8:00生成的文件推送到FTP服务器上
3. 编写自动化shell脚本，使用crontab定时调度实现文件入库。
4. 更新规则：全量更新
5. 入库表名 慢必赔新装表（INF_MBP_DRX）慢必赔报障表（INF_MBP_DRZ）

## 准备工作
### 调度服务器需要安装Oracle客户端，配置tns链接字符串，如下
```bash
EDWDB_NEW =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 137.32.61.13)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = ODSDB)
    )
  )
```
### 调度服务器单独划分一块区域存储从FTP下载下来的数据文件和校验文件。
文件存储路径 /data/manyidu/data
```bash
[root@server1 data]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/xvda3            128G   28G   94G  23% /
tmpfs                 4.9G   76K  4.9G   1% /dev/shm
/dev/xvda1            194M   44M  140M  24% /boot
/dev/mapper/vgdata-lvdata
                      148G  6.5G  134G   5% /data
```
### 文件服务器的网络环境要打通，网络环境配置。
文件服务器IP：137.32.126.137   
用户名：zx_order  
密码：zxOrder_12

### 新建目录，存放脚本如下：
```bash
[root@server1 data]# tree -L 1 /home/oracle/manyidu
/home/oracle/manyidu
|-- ctl     ##用于生成oracle控制文件
|-- data    ##用于存放下载文件，当然我们现在存储源文件是单独分区/data 这里就不用了
|-- log     ##输出日志的目录
|-- run_log ##输出日志的目录
|-- shell   ##存放shell脚本的地方
```

<!-- more -->
## 开始写脚本

### 文件名：load_table_info_DR.sh
```bash
###########################################################################################
#功能： 完成数据导入Oracle表中   
#时间:  2013-08-18
#作者： zhangls
#参数： $1 表名  $2 批次号  传0 表示今天  -1表示昨天 1表示明天  以此类推
#目标： 数据导入Oracle表
###########################################################################################
#!/bin/sh

#配置环境变量
export ORACLE_HOME=/home/oracle/app/oracle/product/10.2.0/client_1
export TNS_ADMIN=$ORACLE_HOME/network/admin
export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
export PATH=$ORACLE_HOME/bin:$PATH
#入参
table_name=$1  #表名
num_date=$2    #批次号
echo "程序开始:"
echo ===============================================
echo "开始时间：`date \"+%Y-%m-%d %H:%M:%S\"`";
## 定义日期及参数 
today=`date +%Y%m%d`    #今天
echo today=$today
numday=`date -d $num_date' days' +%Y%m%d` 
year_No=`expr substr $today 1 4`   #年份2013
#sleep 1
echo "the frist param of the script is :$1"
echo "the numday of the value is: $numday"
echo ===============================================

data_name = $table_name; #实际处理的表名
v_opertion_day = $numday;  #实际处理的批次号
shell_path = /home/oracle/manyidu/shell;
ctl_path = /home/oracle/manyidu/ctl;
log_path = /home/oracle/manyidu/log;
#下载FTP文件路径
pa=/data/manyidu/data
cd $pa
echo $pa

#去远程FTP上下载文件
echo ${data_name}---"[`date`]获取文件开始";
lftp -u ods_jk,hnOds_jk12 ftp://137.32.126.137 <<EOF #登陆FTP服务器
cd day/${v_opertion_day}                             #进入批次号命名的文件夹
mget ${data_name}_${v_opertion_day}*                 #下载文件名匹配成功的文件到本地，注意本地路径
bye                                                  #退出
EOF
echo ${data_name}---"[`date`]获取文件完成" ;

# 接口方规定数据文件接口单表会生成两个文件  生成时间早8:00
# 1.表名_批次号.dat （INF_MBP_DRZ_20180821.dat） 数据文件   每行是一条数据，数据项以","分割 
# 2.表名_批次号.flg （INF_MBP_DRZ_20180821.flg） 数据校验文件  
# 校验文件规则 
# [root@server1 data]# cat INF_MBP_DRZ_20180821.flg
# INF_MBP_DRZ_20180821.dat 8403 25 2018-08-22 5:57:16
# 对应的数据文件名 数据文件字节数  总条数  文件生成时间


# 这里判断数据文件的行数小于1的话就直接退出，否则执行创建控制文件的shell
if [ `ls ${data_name}_$v_opertion_day.dat | awk 'END{print NR}'` -lt 1 ]
    then 
        exit
        EOF
else 
    $shell_path/mkdir_${table_name}_ctl.shell ${data_name} ${v_opertion_day}> /dev/null
    echo "生成控制文件完毕：`date \"+%Y-%m-%d %H:%M:%S\"`";
    ## 增加一个缓冲时间
    sleep 4
    #统计文件记录数作为比对依据
    cd $pa
    FN=$(cat ${data_name}_${v_opertion_day}.dat|wc -l)    # 数据文件的条数
    echo ${data_name}_${v_opertion_day}.dat ----- ${FN}
    dbuser=manyidu
    dbconn=manyidu/manyidu_256@newedw

    #删除现有数据库中表数据，因为是全量抽取
    sqlplus ${dbconn} << SQLEND
    delete from TI_${table_name} where opertion_date='${v_opertion_day}';
    commit;
    SQLEND
    echo ===============================================
    
    sqlldr ${dbconn} control=$ctl_path/${table_name}.ctl errors = 1000                bad=$log_path/${table_name}.bad log=$log_path/${table_name}.log rows=30000 direct = true  bindsize=10485760
    echo ${table_name}==========='导入成功'

    #登陆执行 分析脚本
    sqlplus ${dbconn} << SQLEND   
    exec dbms_stats.gather_table_stats(ownname=>'${dbuser}',tabname=>'TI_${table_name}',estimate_percent=>20,degree => 4,cascade=> TRUE);
    SQLEND
    
    echo "执行时间 [`date`] ";
 
    #登陆执行
    sqlplus ${dbconn}<< SQLEND  
    insert into etl_check_data_info
        ( V_ID,
          v_table_name,
          datafile_cnt, # 数据接口文件的数据量
          table_cnt,  # 加载入库的数据量
          load_date,
          opertion_date,
          remarks  #数据量差异 差异为0 正常，其他都不正常需要稽核数据
        )   
        select
            SEQ_ETL_CHECK_DATA_INFO.Nextval,
            '${table_name}',
            '${FN}',
            (select count(0) from ti_${table_name} where opertion_date='${v_opertion_day}'),
            sysdate,
            '$v_opertion_day',
            (select count(0) from ti_${table_name} where     opertion_date='${v_opertion_day}')-'${FN}' from dual; 
    SQLEND
    echo "[`date`] 插入日志表完成！"
    echo =============================================== 

fi

echo ${table_name} "[`date`] 整体完成日志！"
  
# rm /data/manyidu/data/${data_name}_${v_opertion_day}*
# echo /data/manyidu/data/${data_name}_${v_opertion_day}*
# echo ${table_name} "[`date`] 清除三天前记录！"
exit
```

## 创建生成控制文件的脚本 
脚本名称 mkdir_${table_name}_ctl.shell， 示例表名INF_MBP_DRX
在路径/home/oracle/manyidu/shell 新建mkdir_INF_MBP_DRX_ctl.shell

```bash
###########################################################################################
#功能： 完成表名INF_MBP_DRX接口控制文件生成   
#时间:  2013-08-18 
#作者： zhangls
#参数： $1 表名 $2 批次号
#目标： 按照日期生成文件 
###########################################################################################
#!/bin/sh

data_name=$1
opertion_day=$2
echo ===============================================
echo "生成配置文件...."
cd /home/oracle/manyidu/ctl
address='"'/data/manyidu/data/${data_name}_${opertion_day}'"' 
parameter='"','"'
echo -e "load data 
infile $address
APPEND INTO table TI_${data_name}
fields terminated by $parameter
(
  cityname        ,
  exchname        ,
  orgname         ,
  personname      ,
  accnbr          ,
  contact_name    ,
  birthday        ,
  gender          ,
  menbership_name ,
  contact_tel     ,
  fix_addr        ,
  innet_date      ,
  access_type     ,
  prod_offer_name ,
  is_rh_flag      ,
  std_min_consume ,
  std_bandwidth   ,
  is_itv_flag     ,
  order_title     ,
  external_no     ,
  channel         ,
  applicator      ,
  applicator_tel  ,
  accept_date     ,
  ispretend       ,
  esp_date        ,
  changedate      ,
  finishdate      ,
  payvalue        , 
 OPERTION_DATE constant  \"$opertion_day\")
" > ${data_name}.ctl 
exit
```

## 配置调度crontab 

[root@server1 ~]# crontab -e
```bash
*/1 * * * * ntpdate 137.32.44.178     时间同步服务器
50 7-11 * * * nohup /home/oracle/manyidu/shell/load_table_INF_MBP_DR.sh INF_MBP_DRX 0 > /home/oracle/manyidu/run_log/INF_MBP_DRX.log &
50 7-11 * * * nohup /home/oracle/manyidu/shell/load_table_INF_MBP_DR.sh INF_MBP_DRZ 0 > /home/oracle/manyidu/run_log/INF_MBP_DRZ.log &
```
配置规则为 早上7点到11点钟    每到50分钟的时候调用一次
即 7:50  8:50  9:50  10:50  11:50 触发调度

## 编写总调度shell

如果同步的表比较多可以写个总调度的shell

文件名：load_control.sh
```bash
#!/bin/sh
date "+%Y%m%d"
v_day=$(date "+%Y%m%d")
shell_path = /home/oracle/manyidu/shell;
log_path = /home/oracle/manyidu/run_log;
export ORACLE_HOME=/home/oracle/app/oracle/product/10.2.0/client_1
export TNS_ADMIN=$ORACLE_HOME/network/admin
export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
export PATH=$ORACLE_HOME/bin:$PATH
echo "开始时间：`date \"+%Y-%m-%d %H:%M:%S\"`";
nohup $shell_path/load_table_info_DR.sh TI_BD_MSG_moble_resp 0 > 
$log_path/INF_TI_BD_MSG_MOBLE_RESP.log &

nohup $shell_path/load_table_info_DR.sh TI_BD_STAFF_ORG 0 > 
$log_path/INF_TI_BD_STAFF_ORG.log &

nohup $shell_path/load_table_info_DR.sh TI_BD_ORDER_CODE_INFO 0 > $log_path/INF_TI_BD_ORDER_CODE_INFO.log &

nohup $shell_path/load_table_info_DR.sh TI_BD_Fault_ORDER_INFO 0 > $log_path/INF_TI_BD_Fault_ORDER_INFO.log &

nohup $shell_path/load_table_info_DR.sh TI_BD_MSG_TELE_RESP 0 > 
$log_path/INF_TI_BD_MSG_TELE_RESP.log &

nohup $shell_path/load_table_info_DR.sh TI_BD_MSG_TELE_SEND 0 > 
$log_path/INF_TI_BD_MSG_TELE_SEND.log &

#nohup $shell_path/load_table_info_DR.sh TI_BD_MSG_MOBILE_SEND 0 > /home/oracle/manyidu/run_log$log_path/INF_TI_BD_MSG_MOBILE_SEND.log &

echo "结束时间：`date \"+%Y-%m-%d %H:%M:%S\"`";
```



