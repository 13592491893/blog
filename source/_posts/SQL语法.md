---
abbrlink: 62230
title: SQL语法
comments: true
toc: true
description: SQL语法
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - sql
tags:
  - sql
  - 命令
  - 语法
date: 2021-5-20 16:00:00
---
# SQL语法

## 一、视图

### 1. sqlserver:

   #### 	a.sys_sh_rzp_item_mate_view:

   ``` sql
   SELECT a.EnterpriseId,b.ItemCode,c.ItemCode mateItemCode,a.MateItemTypeCode,a.MateItemTypeName,a.mateitemweight,a.DisPlayOrder,a.IsWeight,a.PreciseScope,
   a.MateItemTime,a.VersionCode,a.VersionName,d.OrgCode,a.IsEnabled,a.IsUsed,a.IsRepeat,a.IsWhole,a.IsUseBag,a.UseBagNumber
   FROM dbo.sys_sh_rzp_item_mate a
   LEFT JOIN dbo.sys_sh_item_master b ON b.EnterpriseId = a.EnterpriseId AND b.ItemId=a.ItemId
   LEFT JOIN sys_sh_item_master c ON c.EnterpriseId=a.EnterpriseId AND c.ItemId=a.MateItemId
   LEFT JOIN dbo.sys_sh_org d ON d.EnterpriseId=a.EnterpriseId AND d.OrgId=a.OrgId
   ```

#### b.sys_sh_rzp_material_contrast_view:

   ```sql
   SELECT c.EnterpriseId,
          c.ExItemId,
   	   exitem.ItemCode ExItemCode,
   	   exitem.ItemName ExItemName,
          c.ItemId,
   	   item.ItemCode,
   	   item.ItemName,
          c.ItemType,
          c.DisPlayOrder,
          c.OrgId,
   	   o.OrgCode,
   	   o.OrgName,
          c.CreateUserCode,
          c.CreateUserName,
          c.CreateDateTime,
          c.EditUserCode,
          c.EditUserName,
          c.EditDateTime 
   FROM sys_sh_rzp_material_contrast c
   LEFT JOIN sys_sh_item_master exitem ON exitem.EnterpriseId = c.EnterpriseId AND exitem.ItemId = c.ExItemId
   LEFT  JOIN sys_sh_item_master item ON item.EnterpriseId=c.EnterpriseId AND item.ItemId = c.ItemId
   LEFT JOIN dbo.sys_sh_org o ON o.EnterpriseId=c.EnterpriseId AND o.OrgId=c.OrgId
   ```


## 二、函数

### 1. sqlserver

#### a.funGetChildItemType:

   ```sql
   -- =============================================
   -- Author:		LH
   -- Create date: 2020-12-09
   -- Description:	根据类别代码获取所有子类别
   --SELECT * FROM funGetChildItemType(3,'RZP')
   -- =============================================
   ALTER FUNCTION [dbo].[funGetChildItemType]
   (	
   @enterpriseId INTEGER,---企业ID
   @ParentItemTypeCode VARCHAR(30)---类别代码
   )
   RETURNS TABLE 
   AS
   RETURN 
   (
   	WITH cr
   	AS (SELECT c.EnterpriseId,c.ItemTypeId,c.ItemTypeCode,c.ItemTypeName
   		FROM sys_sh_item_type c
   		WHERE c.EnterpriseId=@enterpriseId AND EXISTS(SELECT * FROM f_split(@ParentItemTypeCode,',') f WHERE f.item = c.ItemTypeCode )
   	UNION ALL
   	SELECT c.EnterpriseId,c.ItemTypeId,c.ItemTypeCode,c.ItemTypeName
   	FROM sys_sh_item_type c
   	INNER JOIN cr ON cr.EnterpriseId = c.EnterpriseId AND c.ParentItemTypeId=cr.ItemTypeId
   	)
   	SELECT cr.EnterpriseId,ItemTypeId,ItemTypeCode,ItemTypeName FROM cr
   )
   ```

   使用方法：在sql语句里直接使用

   ![image-20210810154630048](SQL语法.assets/image-20210810154630048.png)

   

#### b.proc_commitflight:交班

   ```sql
   /*
   Description： 生成交接班记录
   Create   By：LH
   Create Date：2021-06-02
   Version:2021.7.6.1
   EXEC proc_commitflight '2021/06/09','bc03','前夜','999','管理员'
   */
   
   ALTER PROC proc_commitflight(
       @agitatedate varchar(10)='2021-06-02', --工作日期
       @flightscode VARCHAR(10)='bc02', --班次代码
       @flightsname VARCHAR(10)='白班', --班次名称
   	@usercode    VARCHAR(10),
   	@username    VARCHAR(10),
   	@warehousecode  VARCHAR(10),
   	@enterpriseId VARCHAR(10),
   	@resultnum INT=0 OUTPUT
       )
   AS   
   BEGIN
   	BEGIN TRAN 
   	SET   NOCOUNT   ON
   	SET   ANSI_WARNINGS   OFF
   
   	DECLARE @foreflightscode VARCHAR(10)=''
   	DECLARE @foreagitatedate VARCHAR(10)=@agitatedate
   	
   	SELECT TOP 1 @foreagitatedate=agitateDate,@foreflightscode=flightscode 
   	FROM sys_sh_rzp_material_flightsstock  
   	WHERE warehouseCode=@warehousecode
   	ORDER BY agitateDate DESC,flightscode DESC 
   	/*
   	IF @flightscode = 'bc02'
   		SET @foreflightscode='bc01'
   	ELSE IF @flightscode = 'bc03'
   		BEGIN
   			SET @foreflightscode='bc02'
   		END
   	ELSE IF @flightscode = 'bc01'
   		BEGIN
   			SET @foreflightscode='bc03'
   			SET @foreagitatedate=CONVERT(varchar(10),DATEADD(DAY,-1,CAST(@agitatedate as DATE)),111)
   		END*/
   	CREATE TABLE #inventory(warehousecode VARCHAR(20),warehousename VARCHAR(20),flightscode  VARCHAR(20),flightsname  VARCHAR(20),
   	itemcode  VARCHAR(20),itemname  VARCHAR(50),firstweight DECIMAL(18,4),inweight  DECIMAL(18,4),outweight DECIMAL(18,4),
   	stockweight  DECIMAL(18,4),nowweight  DECIMAL(18,4))
   	---库存盘点 期初
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT warehousecode,warehousename,@flightscode flightscode,@flightsname flightsname,itemcode,itemname,
   	ISNULL(stockweight,0.0000) firstweight,0.0000 inweight,0.0000 outweight,0.0000 stockweight,0.0000 nowweight 
   	FROM sys_sh_rzp_material_flightsstock m
   	WHERE EXISTS(SELECT 1 FROM sys_sh_rzp_material_contrast_view c WHERE c.EnterpriseId=m.EnterpriseId AND c.itemtype='Z' AND c.exitemcode=m.itemcode) AND 
   	EnterpriseId=@enterpriseId AND agitatedate=@foreagitatedate AND flightscode=@foreflightscode AND warehouseCode=@warehousecode AND  ISNULL(stockweight,0)>0
   
   	--原料接收 入库
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT  i.warehouseCode,i.warehousename,d.flightscode,d.flightsname,d.itemCode,d.ItemName,
   	0.0000,SUM(ISNULL(d.RealWeight,0.0000)),0.0000,0.0000,0.0000
   	FROM sys_sh_rzp_material_incept_detail d
   	INNER JOIN sys_sh_rzp_material_incept i ON i.EnterpriseId = d.EnterpriseId AND i.seq = d.seq
   	WHERE EXISTS(SELECT 1 FROM sys_sh_rzp_material_contrast_view c WHERE c.EnterpriseId=d.EnterpriseId AND c.itemtype='Z' AND c.exitemcode=d.itemcode) AND
   	d.EnterpriseId=@enterpriseId AND d.agitatedate=@agitatedate AND d.flightscode=@flightscode AND i.warehouseCode=@warehousecode
   	GROUP BY i.warehousecode,i.warehousename,d.flightscode,d.flightsname,d.itemCode,d.ItemName
   
   	--原料移库 出库
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT  OutWareHouseCode,OutWareHouseName,flightscode,flightsname,itemCode,ItemName,
   	0.0000,0.0000,SUM(ISNULL(netweight,0.0000)),0.0000,0.0000
   	FROM sys_sh_rzp_materialmove m
   	WHERE isDown!='Y' AND EXISTS(SELECT 1 FROM sys_sh_rzp_material_contrast_view c WHERE c.EnterpriseId=m.EnterpriseId AND c.itemtype='Z' AND c.exitemcode=m.itemcode) AND
   	 EnterpriseId=@enterpriseId AND agitatedate=@agitatedate AND flightscode=@flightscode AND OutWareHouseCode=@warehousecode
   	GROUP BY OutWareHouseCode,OutWareHouseName,flightscode,flightsname,itemCode,ItemName
   
   	
   	--原料移库 入库
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT  InWareHouseCode,InWareHouseName,flightscode,flightsname,itemCode,ItemName,
   	0.0000,SUM(ISNULL(netweight,0.0000)),0.0000,0.0000,0.0000
   	FROM sys_sh_rzp_materialmove m
   	WHERE isDown!='Y' AND EXISTS(SELECT 1 FROM sys_sh_rzp_material_contrast_view c WHERE c.EnterpriseId=m.EnterpriseId AND c.itemtype='Z' AND c.exitemcode=m.itemcode) AND
   	 EnterpriseId=@enterpriseId AND agitatedate=@agitatedate AND flightscode=@flightscode AND issign='Y'  AND InWareHouseCode=@warehousecode
   	GROUP BY InWareHouseCode,InWareHouseName,flightscode,flightsname,itemCode,ItemName
   
   	--原料销售 出库
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT  warehousecode,warehousename,flightscode,flightsname,itemCode,ItemName,
   	0,0,SUM(ISNULL(netweight,0.0000)),0.0000,0.0000
   	FROM sys_sh_rzp_materialsale m
   	WHERE  EXISTS(SELECT 1 FROM sys_sh_rzp_material_contrast_view c WHERE c.EnterpriseId=m.EnterpriseId AND c.itemtype='Z' AND c.exitemcode=m.itemcode) AND 
   	EnterpriseId=@enterpriseId AND agitatedate=@agitatedate AND flightscode=@flightscode  AND warehousecode=@warehousecode
   	GROUP BY warehousecode,warehousename,flightscode,flightsname,itemCode,ItemName
   
   	--原料领用 出库
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT  warehousecode,warehousename,flightscode,flightsname,itemCode,ItemName,
   	0.0000,0.0000,SUM(ISNULL(InputWeight,0.0000)),0.0000,0.0000
   	FROM sys_sh_rzp_material_xinput 
   	WHERE EnterpriseId=@enterpriseId AND agitatedate=@agitatedate AND flightscode=@flightscode  AND warehousecode=@warehousecode
   	GROUP BY warehousecode,warehousename,flightscode,flightsname,itemCode,ItemName
   	
   	--盘点库存
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT  warehousecode,warehousename,@flightscode,@flightsname,itemCode,ItemName,
   	0.0000,0.0000,0.0000,SUM(ISNULL(qtyweight,0.0000)),0.0000
   	FROM dbo.sys_sh_rzp_inventory_pda m
   	WHERE (iscancel IS NULL OR iscancel<>'Y') AND EXISTS(SELECT 1 FROM sys_sh_rzp_material_contrast_view c WHERE c.EnterpriseId=m.EnterpriseId AND c.itemtype='Z' AND c.exitemcode=m.itemcode) AND 
   	agitatedate=@agitatedate AND flightscode=@flightscode  AND warehousecode=@warehousecode
   	GROUP BY warehousecode,warehousename,itemCode,ItemName
   
   	--实时库存
   	INSERT INTO #inventory(warehousecode,warehousename,flightscode,flightsname,itemcode,itemname,
   	firstweight,inweight,outweight,stockweight,nowweight)
   	SELECT  warehousecode,warehousename,@flightscode,@flightsname,itemCode,ItemName,
   	0.0000,0.0000,0.0000,0.0000,SUM(ISNULL(nowWeight,0.0000))
   	FROM sys_sh_rzp_materialware m
   	WHERE EXISTS(SELECT 1 FROM sys_sh_rzp_material_contrast_view c WHERE c.EnterpriseId=m.EnterpriseId AND c.itemtype='Z' AND c.exitemcode=m.itemcode) AND 
   	EnterpriseId=@enterpriseId AND  nowweight>0  AND warehousecode=@warehousecode
   	GROUP BY warehousecode,warehousename,itemCode,ItemName
   
   	INSERT INTO sys_sh_rzp_material_flightsstock(enterpriseId,warehousecode,warehousename,agitateDate,flightscode,flightsname,itemcode,itemname,exweight,inweight,
   	outweight,StockWeight,theorystockweight,usercode,username,entryDate,entryTime)
   	SELECT @enterpriseId,warehousecode,warehousename,@agitateDate,flightscode,flightsname,itemcode,itemname,SUM(ISNULL(firstweight,0.0000)),SUM(ISNULL(inweight,0.0000)),
   	SUM(ISNULL(outweight,0.0000)),SUM(ISNULL(stockweight,0.0000)),SUM(ISNULL(nowweight,0.0000)),@usercode,@username,CONVERT(varchar(10),GETDATE(),111),CONVERT(varchar(100), GETDATE(), 8)
   	FROM #inventory
   	GROUP BY warehousecode,warehousename,flightscode,flightsname,itemCode,ItemName
   	SET @resultnum=@@ROWCOUNT
   	IF @@ERROR<>0 GOTO  errhandle
   	DROP TABLE #inventory
   
   	
   	COMMIT   TRAN 
   errhandle:    
   	IF   @@ERROR<>0    
   	BEGIN
   		DROP TABLE #inventory
   		SET @resultnum=-1
   		ROLLBACK   TRAN 
   	END
   
   END
   ```

   ​		使用方法：

![image-20210810155014118](https://gitee.com/gsshy/picgo/raw/master/img/image-20210810155014118.png)

## 三、建DBlink

	### 	1.sqlserver

#### 移库同步数据

1. ```sql
   EXEC sp_addlinkedserver
   @server='10.1.6.14',--被访问的服务器别名（习惯上直接使用目标服务器IP，或取个别名如：JOY）
   @srvproduct='',
   @provider='SQLOLEDB',
   @datasrc='10.1.6.14' --要访问的服务器
   ```

2. ```sql
   EXEC sp_addlinkedsrvlogin
   '10.1.6.14', --被访问的服务器别名（如果上面sp_addlinkedserver中使用别名JOY，则这里也是JOY）
   'false',
   NULL,
   'sa', --帐号
   'shxxzx123!@#' --密码 
   ```

3. 建触发器

   ``` sql
   create trigger trig_move_insert
   on sys_sh_rzp_materialmove
   after insert
   as
   BEGIN
       declare @names nvarchar(50)=null;
       set @names=(select top(1) name from dbo.test1 order by id desc );
       if(select @names) is not null
       begin
       	insert into dbo.test2(name) values(@names);
       end
   END;
   ```

   参考：https://www.cnblogs.com/vuenote/p/9765156.html

## 四、语法

### 	1.exists和in

参考：https://www.cnblogs.com/emilyyoucan/p/7833769.html

``` sql
select a.* from A a where exists(select 1 from B b where a.id=b.id)
以上查询使用了exists语句,exists()会执行A.length次,它并不缓存exists()结果集,因为exists()结果集的内容并不重要,重要的是结果集中是否有记录,如果有则返回true,没有则返回false. 它的查询过程类似于以下过程

List resultSet=[]; Array A=(select * from A)
for(int i=0;i<A.length;i++) {    if(exists(A[i].id) {    //执行select 1 from B b where b.id=a.id是否有记录返回        resultSet.add(A[i]);    } } return resultSet;

当B表比A表数据大时适合使用exists(),因为它没有那么遍历操作,只需要再执行一次查询就行. 如:A表有10000条记录,B表有1000000条记录,那么exists()会执行10000次去判断A表中的id是否与B表中的id相等. 如:A表有10000条记录,B表有100000000条记录,那么exists()还是执行10000次,因为它只执行A.length次,可见B表数据越多,越适合exists()发挥效果. 再如:A表有10000条记录,B表有100条记录,那么exists()还是执行10000次,还不如使用in()遍历10000*100次,因为in()是在内存里遍历比较,而exists()需要查询数据库,我们都知道查询数据库所消耗的性能更高,而内存比较很快.
```

**结论:exists()适合B表比A表数据大的情况**

**当A表数据与B表数据一样大时,in与exists效率差不多,可任选一个使用.**

### 2.rownum

#### 	a.sqlserver

``` sql
row_number函数
select row_number() over(order by seq) rownum from fix_sh_card;
此方法把括号里的查询结果放到变量:temp 里面( 我也不确定是不是变量), 并用row_number()函数进行一个行号跟踪, 再用over 函数进行一个列的排序规则( 是这必须的), 并指定列名为'rownum'
```

#### 	b.oracle

``` sql
--直接使用rownum
select rownum,id,name from student where rownum=1
```

扩展：oracle中rownum和row_number()
row_number()over(partition by col1 order by col2)表示根据col1分组，在分组内部根据col2排序，而此函数计算的值就表示每组内部排序后的顺序编号（组内连续的唯一的）。 与rownum的区别在于：使用rownum进行排序的时候是先对结果集加入伪劣rownum然后再进行排序，而row_number()在包含排序从句后是先排序再计算行号码

详细查看：https://www.cnblogs.com/CandiceW/p/6869167.html

### 3.stuff和for xml path

原表数据：![image-20210810182002445](https://gitee.com/gsshy/picgo/raw/master/img/image-20210810182002445.png)

把isInventory分组，seq加起来用“，”隔开

![image-20210810182118637](https://gitee.com/gsshy/picgo/raw/master/img/image-20210810182118637.png)

``` sql
select 
	IsInventory, 
	seqs = (
		stuff(
			(select ',' + CAST (seq as varchar) from fix_sh_inventory_set where IsInventory = A.IsInventory for xml path('')),
			1,
			1,
			''
		)
	) 
from fix_sh_inventory_set as A group by IsInventory
```

解析：
stuff(param1, startIndex, length, param2)
将param1中自startIndex(SQL中都是从1开始，而非0)起，删除length个字符，然后用param2替换删掉的字符。

```
select STUFF('abcdefg',1,0,'1234')       --结果为'1234abcdefg'  
select STUFF('abcdefg',1,1,'1234')       --结果为'1234bcdefg'  
select STUFF('abcdefg',2,1,'1234')       --结果为'a1234cdefg'  
select STUFF('abcdefg',2,2,'1234')       --结果为'a1234defg'
```

for xml path是将查询结果集以XML形式展现

`select ',' + CAST (seq as varchar) from fix_sh_inventory_set for xml path`

![image-20210810182434517](https://gitee.com/gsshy/picgo/raw/master/img/image-20210810182434517.png)

**比上面多个括号**：

`select ',' + CAST (seq as varchar) from fix_sh_inventory_set for xml path('')`

![image-20210810182510470](C:\Users\45917\AppData\Roaming\Typora\typora-user-images\image-20210810182510470.png)



记住下面这个固定格式：

![image-20210810182632136](C:\Users\45917\AppData\Roaming\Typora\typora-user-images\image-20210810182632136.png)

### 4.union all

​	union all只是合并查询结果，并不会进行去重和排序操作，在没有去重的前提下，使用union all的执行效率要比union高

1. **union**: 对两个结果集进行并集操作, 不包括重复行,相当于distinct, 同时进行默认规则的排序;

   **union all**: 对两个结果集进行并集操作, 包括重复行, 即所有的结果全部显示, 不管是不是重复;

2. **union**: 会对获取的结果进行排序操作

   **union all**: 不会对获取的结果进行排序操作

### 5.查最大编号

 老语句：

   `SELECT IsNull(SUBSTRING(MAX(filmNo),LEN(MAX(filmNo))  ,1),0)+1 number`

   ![image-20210324150630614](https://gitee.com/gsshy/picgo/raw/master/img/image-20210324150630614.png)

   因为filmno是字符类型，直接用MAX函数比较最后一位值的时候永远是9大,就是最后一个'-'后面是10或者11,用MAX函数取值的时候也是9,造成新生成的最大number一直是10

   改后的语句:

   `SELECT isnull(max(cast(REPLACE( SUBSTRING(filmNo, len(filmno)-1, 2) , '-', '') AS int)),0)+1 number `

   注意：max比较的时候要用CAST函数把字符转为int

``` sql
SELECT 'BK' + '3' + convert(varchar,getdate(),12) +RIGHT('000' + CAST(isnull(SUBSTRING(MAX(inwareSeq), len(MAX(inwareSeq)) - 3, 4), 0) + 1 AS nvarchar(5)), 4)
					 from fix_sh_spareparts_inware
        where inwareSeq like 'BK3210524%'
          and enterpriseId = 3
```

### 6.convert

```sql
Select CONVERT(varchar(100), GETDATE(), 0) : 01 26 2021 11:32PM
Select CONVERT(varchar(100), GETDATE(), 1) : 01/26/21
Select CONVERT(varchar(100), GETDATE(), 2) : 21.01.26
Select CONVERT(varchar(100), GETDATE(), 3) : 26/01/21
Select CONVERT(varchar(100), GETDATE(), 4) : 26.01.21
Select CONVERT(varchar(100), GETDATE(), 5) : 26-01-21
Select CONVERT(varchar(100), GETDATE(), 6) : 26 01 21
Select CONVERT(varchar(100), GETDATE(), 7) : 01 26, 21
Select CONVERT(varchar(100), GETDATE(), 8) : 23:35:25
Select CONVERT(varchar(100), GETDATE(), 9) : 01 26 2021 11:35:34:873PM
Select CONVERT(varchar(100), GETDATE(), 10) : 01-26-21
Select CONVERT(varchar(100), GETDATE(), 11) : 21/01/26
Select CONVERT(varchar(100), GETDATE(), 12) : 210126
Select CONVERT(varchar(100), GETDATE(), 13) : 26 01 2021 23:36:08:347
Select CONVERT(varchar(100), GETDATE(), 14) : 23:36:15:593
Select CONVERT(varchar(100), GETDATE(), 20) : 2021-01-26 23:36:22
Select CONVERT(varchar(100), GETDATE(), 21) : 2021-01-26 23:36:29.990
Select CONVERT(varchar(100), GETDATE(), 22) : 01/26/21 11:36:41 PM
Select CONVERT(varchar(100), GETDATE(), 23) : 2021-01-26
Select CONVERT(varchar(100), GETDATE(), 24) : 23:36:58
Select CONVERT(varchar(100), GETDATE(), 25) : 2021-01-26 23:37:05.717
Select CONVERT(varchar(100), GETDATE(), 100) : 01 26 2021 11:37PM
Select CONVERT(varchar(100), GETDATE(), 101) : 01/26/2021
Select CONVERT(varchar(100), GETDATE(), 102) : 2021.01.26
Select CONVERT(varchar(100), GETDATE(), 103) : 26/01/2021
Select CONVERT(varchar(100), GETDATE(), 104) : 26.01.2021
Select CONVERT(varchar(100), GETDATE(), 105) : 26-01-2021
Select CONVERT(varchar(100), GETDATE(), 106) : 26 01 2021
Select CONVERT(varchar(100), GETDATE(), 107) : 01 26, 2021
Select CONVERT(varchar(100), GETDATE(), 108) : 23:38:38
Select CONVERT(varchar(100), GETDATE(), 109) : 01 26 2021 11:38:45:950PM
Select CONVERT(varchar(100), GETDATE(), 110) : 01-26-2021
Select CONVERT(varchar(100), GETDATE(), 111) : 2021/01/26
Select CONVERT(varchar(100), GETDATE(), 112) : 20210126
Select CONVERT(varchar(100), GETDATE(), 113) : 26 01 2021 23:39:18:693
Select CONVERT(varchar(100), GETDATE(), 114) : 23:39:25:183
Select CONVERT(varchar(100), GETDATE(), 120) : 2021-01-26 23:39:45
Select CONVERT(varchar(100), GETDATE(), 121) : 2021-01-26 23:39:53.440
Select CONVERT(varchar(100), GETDATE(), 126) : 2021-01-26T23:40:00.727
Select CONVERT(varchar(100), GETDATE(), 130) : 13 ????? ??????? 1442 11:40:07:847PM
Select CONVERT(varchar(100), GETDATE(), 131) : 13/06/1442 11:40:14:917PM
```

### 7.cast

语法：CAST (expression AS data_type)

### 8.DATEADD

DATEADD() 函数在日期中添加或减去指定的时间间隔。
语法：DATEADD(datepart,number,date)![image-20210811182307381](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811182307381.png)

### 9.with as

```sql
with 
cte1 as 
( 
    select * from table1 where name like 'abc%' 
), 
cte2 as 
( 
    select * from table2 where id > 20 
), 
cte3 as 
( 
    select * from table3 where price < 100 
) 
select a.* from cte1 a, cte2 b, cte3 c where a.id = b.id and a.id = c.id 
--CTE后面必须直接跟使用CTE的SQL语句（如select、insert、update等），否则，CTE将失效。如下面的SQL语句将无法正常使用CTE：
```

### 10.left join一对多只保留一条结果的解决方法

```sql
select * from
（select user_id from tabel_1）a
left join
(select user_id from tabel_2 group by user_id) b
on a.user_id = b.user_id

```

