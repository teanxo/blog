---
title: 关联申请单使用方式.md
date: 2023-02-22 14:57:57
tags:
---

### add.html

1. 对应表单中引入通用列表视图
2. 添加一个hidden input作为容器数据载体
```html
<input type="hidden" id="c-record_sid" class="record_sid" name="row[record_sid]"  />
{include file="workflow/common/relevance_list"}

<!-- 实例代码 -->
<div class="form-group">
    <label class="control-label col-xs-12 col-sm-2">{:__('Record_sid')}:</label>
    <div class="col-xs-12 col-sm-8">
        <div class="input-group">
            <input type="hidden" id="c-record_sid" class="record_sid" name="row[record_sid]"  />
            {include file="workflow/common/relevance_list"}
        </div>
    </div>
</div>
```

### 对应的add js方法
1. 在define中引入zby
```javascript
define(['jquery', 'bootstrap', 'backend', 'table', 'form', 'zz', 'zby'], function ($, undefined, Backend, Table, Form) {
    // 业务代码...
```
2. add方法中
```js
    // 审批流中不可写入时处理
    if (form_field['record_sid']['write'] == 0) {
        // 设置关联申请单为不可操作模式
        Zby.data.relevanceIsOperation = false
    }

    // 初始化关联申请单 参数为容器数据载体
    Zby.initRelevance("#c-record_sid")

    // 注册按钮点击事件
    $("#record_sid_container").on('click', function (){
        // 打开选择关联申请单
        Zby.openRelevanceForm("#c-record_sid")
    })
```

### contoller add
```php
    // 处理关联申请单信息
    if (isset($params['record_sid'])){
        // 先删除当前的申请单数据
        Db::name('relevance_form')->where([
            'old_form_id' => $formid, // 当前审批中的表单ID
            'old_biz_id' => $result // 当前审批中的业务表ID
        ])->delete();
        // 批量插入数据
        $newRelevanceDatas = [];
        // 关联申请单默认传入JSON字符串
        $relevances = json_decode($params['record_sid'], true);
        // 循环写入关联申请表
        foreach ($relevances as $item){
            $newRelevanceDatas[] = [
                'old_form_id' => $formid, // 当前表单ID
                'old_biz_id' => $result, // 当前业务ID
                'new_workflow_id' => $item['workflow_id'], // 关联申请单工作流ID
                'new_form_id' => $item['form_id'], // 当前申请单表单ID
                'new_biz_id' => $item['biz_id'] // 当前申请单业务ID
            ];
        }
        Db::name('relevance_form')->insertAll($newRelevanceDatas);
        unset($params['record_sid']);
    }
```

### edit.html
>此处与新增操作一致
```html
<input type="hidden" id="c-record_sid" class="record_sid" name="row[record_sid]"  />
{include file="workflow/common/relevance_list"}

<!-- 实例代码 -->
<div class="form-group">
    <label class="control-label col-xs-12 col-sm-2">{:__('Record_sid')}:</label>
    <div class="col-xs-12 col-sm-8">
        <div class="input-group">
            <input type="hidden" id="c-record_sid" class="record_sid" name="row[record_sid]"  />
            {include file="workflow/common/relevance_list"}
        </div>
    </div>
</div>
```

### controoler edit return view

1.js中需要表单ID与业务ID 所以在响应视图前需要进行值存储
```php
$work_flow_id = $this->getWorkFlowId();
$formid = Db::name('workflow_list')->where("id", $work_flow_id)->value('form_id');

$this->assignconfig('form_id', $formid);
$this->assignconfig('biz_id', $ids);
```

### 对应的js方法
```js
    // 审批流中不可写入时处理
    if (form_field['record_sid']['write'] == 0) {
        // 设置关联申请单为不可操作模式
        Zby.data.relevanceIsOperation = false
    }

    // 初始化关联申请单 参数为容器数据载体 
    // 此处需要传入表单ID与业务ID  其它无变化
    Zby.initRelevance("#c-record_sid", Config.form_id, Config.biz_id)

    // 注册按钮点击事件
    $("#record_sid_container").on('click', function (){
        // 打开选择关联申请单
        Zby.openRelevanceForm("#c-record_sid")
    })
```


### controller edit save 
>此处与新增操作一致
```php
    // 处理关联申请单信息
    if (isset($params['record_sid'])){
        // 先删除当前的申请单数据
        Db::name('relevance_form')->where([
            'old_form_id' => $formid, // 当前审批中的表单ID
            'old_biz_id' => $result // 当前审批中的业务表ID
        ])->delete();
        // 批量插入数据
        $newRelevanceDatas = [];
        // 关联申请单默认传入JSON字符串
        $relevances = json_decode($params['record_sid'], true);
        // 循环写入关联申请表
        foreach ($relevances as $item){
            $newRelevanceDatas[] = [
                'old_form_id' => $formid, // 当前表单ID
                'old_biz_id' => $result, // 当前业务ID
                'new_workflow_id' => $item['workflow_id'], // 关联申请单工作流ID
                'new_form_id' => $item['form_id'], // 当前申请单表单ID
                'new_biz_id' => $item['biz_id'] // 当前申请单业务ID
            ];
        }
        Db::name('relevance_form')->insertAll($newRelevanceDatas);
        unset($params['record_sid']);
    }
```