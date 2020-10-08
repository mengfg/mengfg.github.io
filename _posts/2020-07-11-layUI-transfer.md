---
layout: post
title: layUI穿梭框 transfer
date: 2020-10-08
tags: layUI
---

transfer 组件可以进行数据的交互筛选,如：

![1602144217326](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1602144217326.png)

上代码：

HTML：

```html
<body>
    <div class="layui-form layuimini-form">
    <div class="layui-form-item">    
        <label class="layui-form-label required">选择人员</label> 
    	<div class="layui-input-block" id="users" name="users"></div>
    </div>
    <div class="layui-form-item">
        <div class="layui-input-block">
            <button class="layui-btn layui-btn-normal" lay-submit lay-filter="saveBtn">提交				</button>
        </div>
    </div>
</div>
    
<script src="../../lib/layui-v2.5.5/layui.js" charset="utf-8"></script>
<!-- 注意th:inline="javascript"，否则行内写法会有问题 -->
<script th:inline="javascript">
    layui.use(['form','transfer','util'], function () {
        var form = layui.form,
            layer = layui.layer,
            transfer = layui.transfer,
            util = layui.util,
            $ = layui.$;

        //监听提交
        form.on('submit(saveBtn)', function (data) {
            //获取选择的数据
            var userData = transfer.getData("checkUser");
            var userList = [];
            for(var p in userData) {
                userList.push(userData[p].value);
            }

            var params = {
                'users':userList
            };

            //ajax提交保存请求
            $.ajax({
                type: 'POST',
                beforeSend: function() {
                    layer.load(2);
                },
                url: '/group/saveGroup',
                data: JSON.stringify(params),
                dataType: 'JSON',
                contentType: 'application/json; charset=UTF-8',
                success: function (res) {
                    if(res.code == '200') {
                        layer.msg(res.msg, { icon: 1 ,time: 1000},function () {
                            var iframeIndex = parent.layer.getFrameIndex(window.name);
                            parent.layer.close(iframeIndex);
                            //刷新父页面的table
                            parent.layui.table.reload('groupTable');
                        });
                    }else {
                        layer.msg(res.msg, {icon: 5, time: 2000});
                    }
                }, complete: function() {
                    layer.closeAll("loading");
                },
                error: function() {
                    layer.msg('系统繁忙请稍后重试', {icon: 5, time: 2000});
                }
            });
            return false;
        });

        //穿梭框初始化人员数据
        $.ajax({
            url: '/user/getUsers',
            method: 'post',
            dataType: 'JSON',
            success: function (res) {
                data1 = res;
                var userIds = [];//存放已经选择的用户，用于修改页面回显

                //js中thymeleaf的行内写法。注意配合<script th:inline="javascript">使用
                if([[${group.users}]] != null ) {
                    var temp = [[${group.users}]];
                    for(var i=0;i<temp.length;i++) {
                        userIds.push(temp[i]);
                    }
                }
               //配置穿梭框
                transfer.render({
                    id: 'checkUser',//获取选择数据时用到
                    elem: '#users',//对应divID
                    data:data1,
                    value:userIds,//用于初始化右侧框的数据。用于更新页面的人员信息回显
                    title: ['待选人员', '已选人员'],
                    showSearch: true,//显示搜索框
                    parseData: function (res) {//解析数据
                        return{
                            "value":res.userId,
                            "title":res.userNick
                        }
                    }

                })
            }
        })

    });
</script>
</body>
```

java后台接受穿梭框的值时，直接使用List接收即可

1.entity实体

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@TableName("SYS_GROUP")
public class Group implements Serializable {
    private static final long serialVersionUID = 1L;
    .........//其他属性省略
        
    //组中成员
    @TableField(exist = false)
    private List<String> users;
}

```

2.controller

```java
@PostMapping("/saveGroup")
    @ResponseBody
    public ResponseResult<String> saveGroup(@RequestBody Group group) {
        //逻辑代码略
    }
```

总结要点：

1.在页面上ajax请求数据，数据请求成功后   transfer.render({…………}）配置

2.后台接受穿梭框数据时，直接使用List接收即可

3.如果保存页面和更新页面共用一个。需要注意更新时信息回显，使用  value 属性，并且注意js中thymeleaf的行内写法