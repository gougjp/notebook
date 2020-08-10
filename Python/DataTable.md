# JS通过DataTable实现表格前端分页

- html源码见DataTable/DataTable.html

- 依赖javascripts: jquery-1.12.4.js和datatables.min.js

- 依赖css: jquery.dataTables.min.css, 该css的还依赖images里面的几个图标, 用于排序时的箭头显示

- DataTable有多个配置项:

```js
<!-- DataTables -->
<link rel="stylesheet" href="/static/bower_components/datatables.net-bs/css/dataTables.bootstrap.min.css">

<!-- DataTables -->
<script src="/static/bower_components/datatables.net/js/jquery.dataTables.min.js"></script>
<script src="/static/bower_components/datatables.net-bs/js/dataTables.bootstrap.min.js"></script>

$('#table1').DataTable({
    autoWidth:true,////不开启自动宽度，用bootstrap的自适应去调整
    "lengthMenu": [10, 20, 50, 100],//表格行数选择框内数目 显示2条,4条,20条,50条
    "displayLength": 10,//默认的显示行数 (也就是每页显示几条数据)
    "order": [],
    "language": {//自定义语言提示
        "processing": "处理中...",
        "lengthMenu": "显示 _MENU_ 项结果",
        "zeroRecords": "没有找到相应的结果",
        "info": "第 _START_ 至 _END_ 行，共 _TOTAL_ 行",
        "infoEmpty": "第 0 至 0 项结果，共 0 项",
        "infoFiltered": "(由 _MAX_ 项结果过滤)",
        "infoPostFix": "",
        "url": "",
        "thousands": "'",
        "emptyTable": "表中数据为空",
        "loadingRecords": "载入中...",
        "infoThousands": ",",
        "paginate": {
            "first": "首页",
            "previous": "上页",
            "next": "下页",
            "last": "末页"
        }
    }
});
```

```js
$('#table1').dataTable({
        searching : false, //去掉搜索框方法一：百度上的方法，但是我用这没管用
        sDom : '"top"i',   //去掉搜索框方法二：这种方法可以，动态获取数据时会引起错误
        bFilter: false,    //去掉搜索框方法三：这种方法可以
        bLengthChange: false,   //去掉每页显示多少条数据方法
});
```
