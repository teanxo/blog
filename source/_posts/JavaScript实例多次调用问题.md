---
title: JavaScript实例多次调用问题
date: 2023-02-04 22:30:42
categories: 错误排查
tags:
    - JavaScript
---


今天在开发过程中遇到以下需求<br />
在一个视图内，渲染两次组织架构树，分别用于两种字段的存取值与渲染，由于同事已经写过这个功能，所以我就只需要将核心的代码拿来复用即可

写出来的代码结构大致如下
```JavaScript
api: {
    tree: {
        return {
            // tree的渲染配置参数
            config: {
                ...省略内部配置
            },
            // tree 初始化
            init() {
                ...根据conifg配置进行渲染tree
            },
            getList(){
                require(['jstree'], function (){
                    ...内部业务方法，每次tree操作事件 都会执行该方法作为刷新tree树事件
                })
            }
        }
    }
}

```

方法调用
```JavaScript
        // 设置A的配置信息 例如dom结构
        api.tree.config.userIdsEvent = "#rule_box"
        api.tree.config.checkAllEvent = "#rule_box_checkall"
        api.tree.config.expandAllEvent = "#rule_box_expandall"
        api.tree.config.ck = Config.rule_box_ck;
        // 加载tree初始化
        api.tree.init()

        // 延迟1秒执行 1秒后进行tree结构默认选中操作
        setTimeout(function (){
            ... 默认选中tree节点
            $(api.tree.channelEvent).jstree("check_node", api.tree.config.ck);
            api.tree.getList(that);
        }, 1000)

        // 设置B的配置信息 例如dom结构
        api.tree.config.userIdsEvent = "#push_box"
        api.tree.config.checkAllEvent = "#push_box_checkall"
        api.tree.config.expandAllEvent = "#push_box_expandall"
        api.tree.config.ck = Config.push_box_ck;

        // 加载tree初始化
        api.tree.init()

        // 延迟1秒执行 1秒后进行tree结构默认选中操作
        setTimeout(function (){
            ... 默认选中tree节点
            $(api.tree.channelEvent).jstree("check_node", api.tree.config.ck);
            api.tree.getList(that);
        }, 1000)

```

乍一看好像没啥问题，运行后才发现  第一个tree不会渲染 只会渲染第二个

经过仔细思考后觉得应该是require内部引用是异步的问题，当赋值完后 require拿到的是第二个的配置信息

于是再次进行修改

方法调用
```JavaScript
    require(['jstree'], function (){

        // 设置A的配置信息 例如dom结构
        api.tree.config.userIdsEvent = "#rule_box"
        api.tree.config.checkAllEvent = "#rule_box_checkall"
        api.tree.config.expandAllEvent = "#rule_box_expandall"
        api.tree.config.ck = Config.rule_box_ck;
            
        // 加载tree初始化
        api.tree.init()

        // 延迟1秒执行 1秒后进行tree结构默认选中操作
        setTimeout(function (){
             ... 默认选中tree节点赋值
            $(api.tree.channelEvent).jstree("check_node", api.tree.config.ck);
            api.tree.getList(that);
        }, 1000)

            
    })

    require(['jstree'], function (){
        // 设置B的配置信息 例如dom结构
        api.tree.config.userIdsEvent = "#push_box"
        api.tree.config.checkAllEvent = "#push_box_checkall"
        api.tree.config.expandAllEvent = "#push_box_expandall"
        api.tree.config.ck = Config.push_box_ck;
        // 加载tree初始化
        api.tree.init()

        // 延迟1秒执行 1秒后进行tree结构默认选中操作
        setTimeout(function (){
            ... 默认选中tree节点赋值
            $(api.tree.channelEvent).jstree("check_node", api.tree.config.ck);
            api.tree.getList(that);
        }, 1000)
    })

```

将require在外部两次引用，这次确实是将tree渲染出来了，但是没有默认选中，冷静下来思考后想起来js深拷贝浅拷贝的问题，当setTimeout执行的时候，第二次赋值将会覆盖掉第一次的赋值，因为该操作指向的都是同一个内存地址，对同一个内存属性进行操作

于是我就尝试使用JSON暴力拷贝方式
```JavaScript
    let that = JSON.parse(JSON.stringify(api.tree))
    that.config.userIdsEvent = "#push_box"
    that.config.checkAllEvent = "#push_box_checkall"
    that.config.expandAllEvent = "#push_box_expandall"
            
    // 加载tree初始化
    that.init()

    // 延迟1秒执行 1秒后进行tree结构默认选中操作
    setTimeout(function (){
        ... 默认选中tree节点
        $(that.channelEvent).jstree("check_node", that.config.ck);
        api.tree.getList(that);
    }, 1000)
```

结果抛出错误 init方法找不到，这里学到了新的知识点

>使用JSON暴力拷贝，不会进行拷贝函数

经过百般思考，想到了class 但是显然我的业务逻辑改为class稍微麻烦，而且对js的面向对象编程并不熟悉，决定将属性改为方法进行 `new`创建，互不冲突，最后的代码如下
```JavaScript
api: {
    tree: function (){
        return {
            config: {
               ...省略内部配置 
            },
            // tree 初始化
            init() {
                ...根据conifg配置进行渲染tree
            },
            getList(){
                require(['jstree'], function (){
                    ...内部业务方法，每次tree操作事件 都会执行该方法作为刷新tree树事件
                })
            }
        }
    }
}
```

```JavaScript
require(['jstree'], function (){
    let that = new api.tree()
    that.config.userIdsEvent = "#push_box"
    that.config.checkAllEvent = "#push_box_checkall"
    that.config.expandAllEvent = "#push_box_expandall"
            
    // 加载tree初始化
    that.init()

    // 延迟1秒执行 1秒后进行tree结构默认选中操作
    setTimeout(function (){
        ... 默认选中tree节点
        $(that.channelEvent).jstree("check_node", that.config.ck);
        that.getList(that);
    }, 1000)
})

require(['jstree'], function (){
        // 设置B的配置信息 例如dom结构
        let that = new api.tree()
        that.config.userIdsEvent = "#push_box"
        that.config.checkAllEvent = "#push_box_checkall"
        that.config.expandAllEvent = "#push_box_expandall"
        that.config.ck = Config.push_box_ck;
        // 加载tree初始化
        that.init()

        // 延迟1秒执行 1秒后进行tree结构默认选中操作
        setTimeout(function (){
            ... 默认选中tree节点赋值
            $(that.channelEvent).jstree("check_node", that.config.ck);
            that.getList(that);
        }, 1000)
    })
```

哎~ 又是趟坑的一天