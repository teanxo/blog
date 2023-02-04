---
title: JDK8之Stream流用法
date: 2023-02-04 23:03:35
categories:
    - 工作记录
tags:
    - Java
---


##Java8 Stream 流操作
现在的业务越来越复复杂话、随着JDK版本主键的更替一种新的操作诞生 流操作
以下描述一些我自己用到的一些场景与用法

####List转Map
常见的
```java
Map<Long, MessageDetail> messageMap = list.stream().collect(Collectors.toMap(MessageDetail::getMessageTypeId, v -> v));
```
如果按照MessageTypeId作为Key转成Map遇到相同的typeId处理方式
```java
Map<Long, MessageDetail> messageMap = list.stream().collect(Collectors.toMap(MessageDetail::getMessageTypeId, v -> v,(a,b)->a));
```
如果遇到相同的Key怎么按照自己的规则去取舍？
```java
    public static <T, K, U>
    Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                    Function<? super T, ? extends U> valueMapper,
                                    BinaryOperator<U> mergeFunction) {
        return toMap(keyMapper, valueMapper, mergeFunction, HashMap::new);
    }
    1.第一个参数是以什么作为Key 
    2.第二个参数是以什么作为Value
    3.如果遇到相同的怎么取舍？

Map<Long, MessageDetail> messageMap = 
list.stream()
.collect(Collectors.toMap(MessageDetail::getMessageTypeId, v -> v,BinaryOperator.minBy((v1,v2) ->v1.getId().compareTo(v2.getId()))));
也可以简写
Map<Long, MessageDetail> messageMap = 
list.stream().collect(Collectors.toMap(MessageDetail::getMessageTypeId, v -> v,BinaryOperator.minBy(Comparator.comparing(MessageDetail::getId))));
    1.按照messageTypeId作为Key 转Map
    2.返回自己作为 Value
    3.如果遇到 messageTypeId相同的 根据对象的 id判断取最小保留

```

####List 排序
贴一个常用的类所有stream的操作都用以下实体类
```java
public class Student {
    private Long id;
    private Integer age;
    private Integer sort;
    private String name;
    private Integer sex;
}
```
1.默认排序
2.按照字段排序
3.按照字段倒序
3.多字段排序
```java
默认排序
list.stream().sorted().toList();
按照 id排序（默认是正序）
list.stream().sorted(Comparator.comparing(Student::getId)).toList();
按照ID 排序 倒序
list.stream().sorted(Comparator.comparing(Student::getId).reversed()).toList();
多字段排序 关键词  thenComparing
list.stream().sorted(Comparator.comparing(Student::getId).thenComparing(Student::getAge).reversed()).toList();
```
