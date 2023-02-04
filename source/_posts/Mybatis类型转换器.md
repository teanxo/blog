---
title: Mybatis类型转换器
date: 2023-02-04 23:01:46
categories: 
    - 工作记录
tags:
    - Java
    - MyBatis
---

#### 描述
我们应该都遇到过这种需求或者是这种表结构设计的字段
`ids varchar(255)` 存储ID  逗号','分割 的字符串
 ids顾名思义 就是自己定义符合自己规则的字段每次查询在去特殊处理
 在Mybatis传统框架中使用方式 自定义一个转换器 继承 BaseTypeHandler类在去实现某些方法
 [传送门](https://blog.csdn.net/qq_48569009/article/details/124208363)
接下来我想说的是 Mybatis-plus的使用方式非常之方便
关键类
FastjsonTypeHandler 或者 JacksonTypeHandler 这里我使用的是 第二个
##### 1.定义转换器（上代码）
```java
public class ListLongTypeHandler extends JacksonTypeHandler {

    public ListLongTypeHandler(Class<?> type) {
        super(type);
    }

    @Override
    protected Object parse(String json) {
        if(VariableUtil.isNotEmpty(json)){
            try{
                return Arrays.stream(json.split(",")).map(Long::parseLong).toList();
            }catch (Exception e) {
                return new ArrayList<Long>();
            }
        }else{
            return new ArrayList<>();
        }
    }

    @Override
    protected String toJson(Object obj) {
        if(VariableUtil.isNotEmpty(obj)){
            List<Long> list = (List<Long>)obj;
            return list.stream().map(v -> v.toString()).collect(Collectors.joining(","));
        }else{
            return "";
        }
    }
}
```
其中只需要实现 toJson 控制 传入操作  parse传出操作 即可

##### 2.配置转换器
```java
1.给实体类添加 注解类型 
@TableName(autoResultMap = true)
2.给字段添加 转换器注解
@TableField(typeHandler = ListLongTypeHandler.class)
private List<Long> otherIds;
```
只需要这两步  使用mybatis-plus的操作就可以完成自动转换了。