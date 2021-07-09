java集合

java 集合的两个基本接口

Collection

Map



特点

以上两个接口分别实现了Abtract方法，为其下所属类提供相同的例行方法，不必重新构建

List和set分别继承了Collection接口

arrayList，linkedList实现list接口

arrayList 初始化空数组

add 初始化10长度的数组

超过10，1.5被扩容



不管是添加还是删除都是copy数组，重新生成新的数组，效率极低



但是由于数组分配的是连续内存，随机查找效率大大提高



treeset，hashset实现set接口

