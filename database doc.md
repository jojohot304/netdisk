DataBase

在mysql中做统计的时候，count（*）会扫描所有的行数，不管行的值是否为空，count（'column name'）会过滤掉值为空的行，count（1）的意思就是扫描主键所对应的列，效率会比count（*）高
ifnull(code_mode,'unknown')的意思就是如果code_mode的值为NULL，那么对其赋值为unknown
substring(str, begin, end),表示从开始到结束的位置截取字符串字段，字段的位置从1开始算
case value
when xxx value1
when xxx value2
end
表示先取value的值，然后进行条件判断，然后根据判断赋予新的值
MYSQL语句之间，各个语句之间一般是不要加，号的，只有在同一个语句内的几个变量之间才需要加，号

修改mysql里面表的结构，使用alter tablename命令，例如修改表里面一列的类型定义
alter tablename modify 列名 varchar(255);   将会把该列的类型定义为255长度字符串
SQL查询中，添加distinct统计中去除重复数据（count（distinct xxx））

左链接会显示左表的所有数据，然后会根据链接中的条件显示右表中符合条件的数据（一般会设置两边的ID匹配，简单来说join就是在一张表上面格外再显示一些字段），如果右表里面会多个数据匹配的话，会重复显示左表的数据（比如一个表存基本信息，另外一个表存一些属性信息），这种情况，一般使用GROUPCONTACT函数，此函数可以将零散的属性信息合并到一起，相同ID仅显示一行数据，合并后的数据会使用；进行分隔

在mysql语句中，<>符号为不等于的意思，同时将变量与Null进行比较的时候，需要使用 is Null或者is not Null，不能使用 = Null的表达式
mysql中， case 
	      when  people.sex='male' then 'boy'
	      when  people.sex='female' then 'girl'
	      else 'ladyboy'
         end
要判断的值可以放在when里面，也可以直接放在case当中，就和switch case的用法一样了

创建index时需要为索引名称以及索引对应的column名称，然后会选择索引的类型
主要有三种索引类型：
（1）normal：
（2）full-text：用于文本的索引（列类型要是char、text、varchar），可以避免文本搜索使用like效率低下的问题，full-text一般使用的场景是长文本当中搜索一些词语，mysql在创建索引的时候会对文本进行自然语言处理，同时需要注意的是full-text索引无法为单列数据创建，需要针对多列数据联合创建full-text索引，由此可见，创建full-text索引的开销还是很大的
（3）unique： 如果可以保证索引列的值唯一就可以创建为此类型（一般为id类）
然后需要选择索引方法，在mysql中主要有两种
（1）b-tree，  也就是使用b-tree来储存、查找索引，同时这种类型的b-tree会在叶子节点一并存储索引所对应的行的其它列的信息
（2）hash，也就是通过hash-map来储存一行或者多行对应的hash值，查找的时候直接查找hash-map得到结果，所以有map所对应的各种特点，无法范围搜索（hash map一对一），查找的结果无序（hash map本身就是无序的）
使用了索引之后，你会发现删除的时候效率会比较低，这是因为每次删除数据会重新生成索引树（删除叶子节点，插入父节点等操作），所以使用索引的时候最好针对频繁查找但是偶尔进行修改、删除的数据
对于使用了索引又需要进行删除的数据，有一种解决办法就是进行假删除，只是让原本的数据检索不到，但是数据在数据库当中依然真实存在、亦或是可以只删除数据而不删除索引的数据

MYSQL里面一直复杂的操作，例如多表的删除、修改操作，最好使用事务来完成，事务可以在执行的时候避免这一组语句之间的冲突，而且在保证这些语句均执行成功后才提交保存改动，失败的时候会执行回退；
一般也可以创建函数或者存储过程（Procedure）来执行事务，其实相当于一个小脚本，把一堆SQL语句包裹起来，注意的是里面的每句均需要加上分号，同时需要定义好输入的参数，函数还需要定义好输出参数的类型、长度；（ALTER语句仅仅只能修改函数和过程的参数信息，无法修改body，所以需要先删除，再重新添加）
同时在写存储过程的时候可以在语句当中定义临时变量，使用select * INTO  xxx的形式进行赋值

MYSQL在使用DELETE语句做多表关联删除的时候有BUG，在使用subquery的时候是无法使用到索引的（看stackoverflow上面的老哥说的是因为subquery创建的是一张虚拟的表单，所以无法关联到索引，但是我索引关联的不是subquery出来的临时表单啊），总之类似这种形式的语句是无法使用索引的：
DELETE FROM t_aw_baw  WHERE tag_id IN (SELECT tag.uuid FROM t_aw_tag AS tag LEFT JOIN t_aw_info AS aw ON tag.aw_id = aw.uuid WHERE aw_id = _aw_id);
将其修改为：
DELETE t_aw_caw from t_aw_caw INNER JOIN (SELECT tag.uuid FROM t_aw_tag AS tag LEFT JOIN t_aw_info AS aw ON tag.aw_id = aw.uuid WHERE tag.aw_id=_aw_id) AS aw_tag ON (t_aw_caw.tag_id=aw_tag.uuid);
去掉了IN，先提取出子查询中的tag_id记录，然后再使用INNER JOIN将符合条件的caw表中内容列出，然后再将其删除；

在mysql当中，使用索引虽然可以大大提高查询的速度，但是由于索引是格外增加存储空间，所以对于列表更新（insert，update，delete）这些写操作的效率是降低的，因为写的时候要额外操作存储空间作为索引的使用，所以对于频繁要进行写操作的表单建立索引一定要慎重；


在存储过程当中，根据某一些条件进行操作的时候，可以设置一个局部变量var，利用SELECT xxx INTO var 的形式，首先为var赋值，然后再根据var的值来进行操作；

在迁移数据库的时候，需要一并将mysql的配置文件拷贝过去（mysql当中会有很多关于模式、视图的设置）
