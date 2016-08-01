#BDPE4.5元数据与数据治理融合研发文档

## 第1章 概述
### 1.1任务概述
    BDPE4.1、4.3版本已经具备独立部署能力。BDPE产品合并入DACP产品线后，需要明确以下产品能力划分：
    -  bdpe专注以数据采集；
    -  元数据部分由数据治理产品线承建；
    -  用户管理、系统管理等模块从长期规划来看会上升到产品线公共模块；
    -  逐步剥离业务熟悉，如模块管理、一经上报逐步剥离；
    -  数据质量由数据治理产品线承建。
    基于以上产品功能拆分，我们首先会进行元数据融合，综合考虑方案。本次不采取数据服务的方式（后台组件修改、前台修改，修改工作量大，任务时间不可控，功能需要全面测试）。
    项目实施策略为：在有DACP数据治理产品的情况下，在新建项目点采取bdpe4.5(采集）+统一调度（dacp现有）+数据开发（dacp原有）+数据治理（dacp原有）+数据分发（新建）实现大数据平台数据接入、开发、管控、数据内部交换等业务需求。
###1.2本次 bdpe4.5版本改造任务：
    1. 合并配置库。从目前资料介绍看mysql与postgresql并没有明显的性能差异，性能部分后期安排专人对mysql进行优化研究。部署实施时做mysql主备同步;
    2. postgresql的sql语法转Mysql语法;
    3. 进行用户统一登录认证。即在DACP大平台统一进行用户名、密码管理，登录后直接可以点击bdpe界面访问。
    4. 元数据表使用数据治理元数据。
###1.3文档阅读对象
	*  研发人员
	*  维护人员
	*  QA测试人员。

##第2章 需求概述
###2.1	需求来源
1.	研发内部产品规划需求
2.	贵州移动、安徽移动、辽宁移动、南方基地、广东移动都要求DACP内部产品整合，能实现统一登录、统一元数据。

###2.2	研发基线任务列表
| 任务ID   |    任务名称  | 任务简要描述            | 依赖任务|负责人|完成时间|
| :-------| :--------   | :------               |:---   |:----|:-----|
| task01  | mysql初始化脚本编写| 建表脚本、存储过程改写| 无|黄泽元、宋志国、雷佳佳|20160701|
| task02  | 后台mysql脚本转换  | server端代码支持mysql| task01、task04    |薛晨、于清水|20160705|
| task03  | 前台mysql脚本转换  | web端代码支持msql    | task01、task04     |宋志国、薛高飞|20160705|
| task04  | git代码迁移| 采取合理方式将4.1版本迁移到git服务器| 无     |于清水 |20160701|
| task05  | 打包环境构建 |在云环境新建一个打包环境支持git| task04、task02     | 冯文 |20160705|
| task06  | 测试环境构建| 在测试环境新建基于mysql数据环境| task05    | 冯文 |20160705|
| task07  | mysql主备安装| 在测试环境安装mysql主备，输出主备文档，及优化参数。优化可以后续做，需要提供研发操作的mysql环境| 无|郭孟振|20160628|
| task08  | 梳理数据治理平台元数据表|元数据梳理，构建统一可访问的数据表，核对表结构差异 | 无| 薛高飞、薛晨、陈强|20160701|
| task09  | 与数据治理平台联调测试|配置库合并，需要注意schema等问题| task06     | 薛晨、宋志国、薛高飞、于清水|20160708|
| task10  | 与统一调度平台接口实现| 实现bdpe加载后给出表完成事件给统一调度平台的外部事件表|task06| 黄泽元  |20160708|
| task11  | 总体测试 |平台可用性测试| task10| 郭孟振、陈强、宋春玲、黄泽元、雷佳佳 |20160713|
| task12  | 研发交付物  | 1、mysql初始化脚本 2、安装手册 3、git代码管理 4、打包步骤及环境 5、安装步骤 6、与数据治理结合的操作手册。部分文档由研发人员统一输出，汇总到郭梦征、陈强、宋春玲处|task11| 郭孟振、陈强、宋春玲、黄泽元、雷佳佳|20160713|


##第3章 目标系统描述


``` flow
st=>start: Start
e=>end: End
op0=>operation: bdpe Server | SchedualSever| 数据治理前台
op1=>operation: 主mysql
op2=>operation: 备mysql
op3=>operation: bdpe Server | SchedualSever | 数据治理前台

op0->op1
op1(right)->op2
op2->op3
```
##第4章
###4.1 dacp表结构
####4.1.1 元数据表tablefile
```sql
mysql建表语句
CREATE TABLE `tablefile` (
  `XMLID` varchar(64) NOT NULL COMMENT '对象ID',
  `DATANAME` varchar(120) NOT NULL COMMENT '表名',
  `DATACNNAME` varchar(120) NOT NULL COMMENT '中文名',
  `TEAM_CODE` varchar(32) DEFAULT NULL COMMENT '团队代码',
  `SCHEMA_NAME` varchar(120) DEFAULT NULL COMMENT '模式名',
  `DATATYPE` varchar(20) DEFAULT NULL COMMENT '数据类型（V:视图、T:表）',
  `DBNAME` varchar(32) DEFAULT NULL COMMENT '存储数据库(metadbcfg.dbname)',
  `TABSPACE` varchar(120) DEFAULT NULL COMMENT '表空间.如果是hive存放路径',
  `INDEX_TABSPACE` varchar(120) DEFAULT NULL COMMENT '索引空间',
  `LEVEL_VAL` varchar(32) NOT NULL COMMENT '层次（ODS、DWD、DW、DM、ST）',
  `RIGHTLEVEL` varchar(32) DEFAULT NULL COMMENT '敏感级别',
  `DELIMITER` char(10) DEFAULT NULL COMMENT '分割符(hadoop平台文件分割。也可作为接口平台入库文件依据)',
  `SPLITTYPE` varchar(8) DEFAULT NULL,
  `TOPICNAME` varchar(32) DEFAULT NULL COMMENT '主题（客户域\n            用户域\n            服务域\n            行为域\n            资源域\n            事件域\n            账务域\n            资源域\n            财务域\n            维表域\n            集团用户\n            专题分析\n            KPI分析\n            多维成本\n            重点应用\n            ）',
  `CYCLETYPE` varchar(32) DEFAULT NULL COMMENT '周期类型(日、周、月、年、多日、多月、多年）',
  `COMPRESSION` varchar(1) DEFAULT NULL COMMENT '是否压缩(Y/N)',
  `FIELDNUM` int(11) DEFAULT '0' COMMENT '字段个数',
  `TABSIZES` decimal(15,0) DEFAULT '0' COMMENT '表大小',
  `ROWNUM_VAL` decimal(10,0) DEFAULT '0' COMMENT '记录条数',
  `REFCOUNT` int(11) DEFAULT '0' COMMENT '引用次数',
  `EFF_DATE` date NOT NULL COMMENT '创建时间',
  `CREATER` varchar(32) NOT NULL COMMENT '创建者',
  `STATE` varchar(32) NOT NULL COMMENT '当前状态（新建、审核、发布、未上线、开放()）',
  `STATE_DATE` date DEFAULT NULL COMMENT '状态时间',
  `DEVELOPER` varchar(32) DEFAULT NULL COMMENT '开发者',
  `CURDUTYER` varchar(32) DEFAULT NULL COMMENT '负责人（如果是新建，与CREATER一样。根据状态修改人员而变化）',
  `VERSEQ` int(11) DEFAULT NULL COMMENT '版本号',
  `DESIGNER` varchar(32) DEFAULT NULL COMMENT '设计人员',
  `AUDITER` varchar(32) DEFAULT NULL COMMENT '审核人员',
  `DATEFIELD` varchar(32) DEFAULT NULL COMMENT '生命周期',
  `DATEFMT` varchar(16) DEFAULT NULL COMMENT '生命周期单位（日、月、永久)',
  `DATETYPE` varchar(16) DEFAULT NULL,
  `EXTEND_CFG` varchar(1024) DEFAULT NULL COMMENT '{location:"文件路径",sprate:''分隔符}',
  `REMARK` varchar(512) DEFAULT NULL COMMENT '备注',
  `OPEN_STATE` varchar(32) DEFAULT NULL COMMENT '开放状态',
  `dwdbdura` int(11) DEFAULT NULL,
  PRIMARY KEY (`XMLID`,`DATANAME`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='数据表.存储RDB、hive元模型';
```

####4.1.2 column_val表字段信息
```sql
CREATE TABLE `column_val` (
  `XMLID` varchar(64) NOT NULL COMMENT '对象ID',
  `DATANAME` varchar(120) NOT NULL COMMENT '表名',
  `COLNAME` varchar(80) NOT NULL COMMENT '字段名',
  `COLCNNAME` varchar(120) NOT NULL COMMENT '字段中文名',
  `DATATYPE` varchar(32) NOT NULL COMMENT '数据类型',
  `LENGTH` int(11) DEFAULT '0' COMMENT '字段长度',
  `REMARK` varchar(800) DEFAULT NULL COMMENT '备注',
  `ISNULLABLE` varchar(1) DEFAULT NULL COMMENT '是否允许为空',
  `ISPRIMARYKEY` int(11) DEFAULT '0' COMMENT '是否主键',
  `PRECISION_VAL` int(11) DEFAULT '0' COMMENT '精度',
  `SCALE` int(11) DEFAULT '0' COMMENT '范围',
  `COL_SEQ` int(11) NOT NULL DEFAULT '1' COMMENT '字段顺序1开始',
  `KEY_SEQ` int(11) DEFAULT NULL,
  `INDEX_SEQ` int(11) DEFAULT NULL,
  `PARTY_SEQ` int(11) DEFAULT NULL,
  `FKTABNAME` varchar(120) DEFAULT NULL COMMENT '引用表',
  `FKCOLNAME` varchar(80) DEFAULT NULL COMMENT '引用表字段',
  `STARTPOS` int(11) DEFAULT '0' COMMENT '开始字符位置（定长文件加载时生效）',
  `ENDPOS` int(11) DEFAULT '0' COMMENT '结束字符位置（定长文件加载时生效）',
  `EFF_DATE` date DEFAULT NULL COMMENT '生效时间',
  `CREATER` varchar(32) DEFAULT NULL COMMENT '创建者',
  `STATE_DATE` date DEFAULT NULL COMMENT '状态变更时间',
  `STATE` varchar(12) DEFAULT NULL COMMENT '当前状态',
  `UNICODE` varchar(40) DEFAULT NULL COMMENT '字符集',
  `DBDATATYPE` varchar(32) DEFAULT NULL COMMENT 'DB2字段（重庆临时使用，后期全部使用DATATYPE)',
  `HIVEDATATYPE` varchar(32) DEFAULT NULL COMMENT 'HIVE字段（重庆临时使用，后期全部使用DATATYPE)',
  `segment_seq` varchar(30) DEFAULT NULL,
  `coding` varchar(30) DEFAULT NULL,
  `order_seq` varchar(30) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

###4.2 数据治理平台
目前数据治理平台演示环境:
1、dacp访问方式：
http://10.1.235.39:8080/me/ftl/me/dataos/index       username/passwd:    sys/123 
dacp服务器： 10.1.235.39     username/passwd:  dacp/dacp
2、基础数据库
mysql数据库： 10.1.234.136:3306   dacp/dacp    dacp数据库.

可以查看元数据添加、及数据库表访问等。

##第5章待定问题列表

| 编号   |    简要描述              |负责人    |完成时间|
| :-------| :--------             | :------     |:---         |
|1| 无文件元数据对应表|冯文||
|2|部分dacp系统管理模块已提供数据管理、ftp管理如何与bdpe统一结合的问题|冯文||
|3|同时部署时，如何解决用户操作问题|冯文||
|4|mysql性能问题|冯文||