* 外置finedb迁移至为imb db2时，长度为1000和3000的String对象入库后，类型会转换成varchar(1000)和varchar(3000)，长度为65536的String对象会转换成long varchar(32700)。  
* 在db2中，普通数据类型(如varchar)通常会存储到buffer pool，受限于该表占用的tablespace的page size。  
* long varchar则有单独的存储区，不会存储到buffer pool，但使用会受到限制(如无法使用GROUP BY,ORDER BY)。  
* 建表后，表占用的tablespace很难更改。  
* 考虑到部分用户将表空间page size定为8k，故当一个表建表时，该表所有字段(除长度定为32700的long varchar外的)长度之和不宜超过8101(由db2规定)。  
* 即在Entity中添加字段时，要注意(除长度定为65536的String外的)所有列对象长度之和不宜超过8101。