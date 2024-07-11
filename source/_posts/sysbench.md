---
title: sysbench
date: 2024-07-11 13:41:12
categories:
- 测试工具
- sysbench
tags:
---

# sysbench
安装
```bash
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
```

## mysql测试
执行以下sysbench命令可以创建，sysbench的内置表。
```bash
sysbench --db-driver=mysql \
--mysql-host=127.0.0.1 \
--mysql-port=33106 \
--mysql-user=root \
--mysql-password=Abc123456 \
--mysql-db=test_db \
--table_size=25000 \
--tables=250 \
--events=0 \
--time=600 \
oltp_read_write prepare
```

只写
```bash
##准备数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_write_only run


##运行 workload
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
--events=0 \
--time=600 \
--threads=192 \
--percentile=95 \
--report-interval=1 \
oltp_write_only run

##清理数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \ 
oltp_write_only cleanup

```

![](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240711140848333.png)

![image-20240711142412779](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240711142412779.png)

![image-20240711141817805](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240711141817805.png)

只读（point select）

```bash
##准备数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_only prepare

##运行 workload
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
--events=0 \
--time=600 \
--threads=512 \
--percentile=95 \
--range_selects=0 \
--skip-trx=1 \
--report-interval=1 \
oltp_read_only run

##清理数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_only cleanup

```

只读（range select）
```bash
##准备数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_only prepare

##运行 workload
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
--events=0 \
--time=600 \
--threads=512 \
--percentile=95 \
--skip-trx=1 \
--report-interval=1 \
oltp_read_only run

##清理数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_only cleanup
```

混合读写point select
```bash

##准备数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_write run

##运行 workload
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
--events=0 \
--time=600 \
--range_selects=0 \
--threads=XXX \
--percentile=95 \
--report-interval=1 \
oltp_read_write run

##清理数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_write cleanup

```

混合读写range select
```bash

##准备数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_write run

##运行 workload
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
--events=0 \
--time=600 \
--threads=XXX \
--percentile=95 \
--report-interval=1 \
oltp_read_write run

##清理数据
sysbench --db-driver=mysql \
--mysql-host=XXX \
--mysql-port=XXX \
--mysql-user=XXX \
--mysql-password=XXX \
--mysql-db=XXX \
--table_size=XXX \
--tables=XXX \
oltp_read_write cleanup

```