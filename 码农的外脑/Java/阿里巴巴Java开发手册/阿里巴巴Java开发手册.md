
> 官网链接

[alibaba/p3c: Alibaba Java Coding Guidelines pmd implements and IDE plugin (github.com)](https://github.com/alibaba/p3c)

> idea插件

[p3c/idea-plugin/README_cn.md at master · alibaba/p3c (github.com)](https://github.com/alibaba/p3c/blob/master/idea-plugin/README_cn.md)


# 编程规约

## OOP 规约

### 关于基本数据类型与包装数据类型的使用标准如下： 

1）【强制】所有的 POJO 类属性必须使用包装数据类型。 

2）【强制】RPC 方法的返回值和参数必须使用包装数据类型。 

3）【推荐】所有的局部变量使用基本数据类型。 

- 说明：POJO 类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何 NPE 问题，或者入库检查， 都由使用者来保证。 
- 正例：数据库的查询结果可能是 null，因为自动拆箱，用基本数据类型接收有 NPE 风险。 
- 反例：某业务的交易报表上显示成交总额涨跌情况，即正负 x%，x 为基本数据类型，调用的 RPC 服务，调用不成功时， 返回的是默认值，页面显示为 0%，这是不合理的，应该显示成中划线-。===所以包装数据类型的 null 值，能够表示额外的信息===，如：远程调用失败，异常退出。

## 前后端规约

### 前后端数据列表相关的接口返回，如果为空，则返回空数组 `[]` 或空集合 `{}`

- 说明：此条约定有利于数据层面上的协作更加高效，减少前端很多琐碎的 `null` 判断。
