# SpreadJS_ColumnHeaderCheckboxSelectAll
列头添加复选框全选功能


### SpreadJS 示例，复选框全选功能
该示例包括使用 SpreadJS API 的演示脚本，可用于实现复选框全选功能
有关 SpreadJS API 的更多信息，请参阅[SpreadJS API指南]( https://demo.grapecity.com.cn/spreadjs/help/api/) 和[帮助手册]( https://help.grapecity.com.cn/pages/viewpage.action?pageId=5963808)。

### 运行步骤
1、在开始之前，请确保您已满足以下先决条件：
要运行 SpreadJS，浏览器必须支持 HTML5，客户端导入和导出 Excel 需要 IE10及以上。
请先了解 [SpreadJS 的产品使用环境]( https://www.grapecity.com.cn/developer/spreadjs/selection-guide/product-use-environment)，并申请临时部署授权激活
安装并更新NodeJS和NPM
2、克隆或下载此代码库
3、初始化控件，并运行示例脚本
#### 控件初始化
首先，创建一个新页面，并在页面上输入以下代码：
```
<!DOCTYPE html>
    <html>
    <head>
        <title>SpreadJS HTML Test Page</title>
```
2、在页面中添加对 SpreadJS 的引用。代码如下。需要注意的是，SpreadJS 提供压缩过
```
//（minified）的 JavaScript 文件和和用于调试的文件：
<script src="[Your_Scripts_Path]/gc.spread.sheets.all.xxxx.min.js" type="text/javascript"></script>
```
3、添加 CSS 文件以改变Spread.JS 的外观。默认的CSS文件名为： 
gc.spread.sheets.xxxx.css，里面包含了所有的默认样式。该 CSS 文件将会影响滚动条，筛选框及其子元素，单元格和下方标签栏的样式。引入 CSS 的代码如下：
```
//<link href="[Your_CSS_Path]/gc.spread.sheets.xxxx.css" rel="stylesheet" type="text/css"/>
```
4、添加产品授权，代码为（本地测试可以不添加）：
```
GC.Spread.Sheets.LicenseKey = "xxx";
```
5. 添加控件初始化代码。本例会在一个 id 为 “ss” 的 DOM 元素上初始化 SpreadJS：
```
<script type="text/javascript">
// Add your license
// If run this in local for testing, remove or comment below code
 GC.Spread.Sheets.LicenseKey = "xxx";

// Add your code
 window.onload = function(){
var spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"),{sheetCount:3});
var sheet = spread.getActiveSheet();
 }
</script>
</head>
<body>
```
6、 创建一个 id 为 “ss” 的元素，SpreadJS 将在该 DOM 中初始化：
```
<div id="ss" style="height: 500px; width: 800px"></div>
</body>
</html>
```
#### 示例代码
```
HTML：
<p>添加单列复选框全选功能</p>
<div id="ss"></div>

CSS：
p{
    color: #336699;
    text-align: center;
}

#ss{
    width: 100%;
    height: 450px;
}

JavaScript：
//Title: 列头全选框
//Description：设置列头全选框
//Tag：列头全选框

window.onload = function() {
    var spread = new GC.Spread.Sheets.Workbook($("#ss").get(0), {
        sheetCount: 1
    });
    var sheet = spread.getActiveSheet();

    sheet.setCellType(0, 0, new MyCheckBoxCellType(), GC.Spread.Sheets.SheetArea.colHeader);

    for (var i = 0; i < 8; i++) {
        var c = new GC.Spread.Sheets.CellTypes.CheckBox();
        c.textAlign(GC.Spread.Sheets.CellTypes.CheckBoxTextAlign.right);
        sheet.setCellType(i, 0, c, GC.Spread.Sheets.SheetArea.viewport);
    }

    spread.bind(GC.Spread.Sheets.Events.ButtonClicked,
        function(e, args) {
            var sheet = args.sheet,
                row = args.row,
                col = args.col;
            var cellType = sheet.getCellType(row, col);
            if (cellType instanceof GC.Spread.Sheets.CellTypes.CheckBox) {
                var colHeaderCell = cellType = sheet.getCell(0, col, GC.Spread.Sheets.SheetArea.colHeader);
                if (colHeaderCell.cellType() instanceof MyCheckBoxCellType) {
                    var checkStatus = true;
                    for (var i = 0; i < sheet.getRowCount(); i++) {
                        var cell = sheet.getCell(i, col);
                        if (cell.cellType() instanceof GC.Spread.Sheets.CellTypes.CheckBox && !cell.value()) {
                            checkStatus = false;
                            break;
                        }
                    }
                    colHeaderCell.tag(checkStatus);
                    sheet.repaint();
                }
            }
        });

};

function MyCheckBoxCellType() {
    GC.Spread.Sheets.CellTypes.CheckBox.apply(this);
    this.caption("All");
}
MyCheckBoxCellType.prototype = new GC.Spread.Sheets.CellTypes.CheckBox();
var basePaint = GC.Spread.Sheets.CellTypes.CheckBox.prototype.paint;
MyCheckBoxCellType.prototype.paint = function(ctx, value, x, y, width, height, style, context) {

    //var tag = context.sheet.getColumn(context.col,context.sheetArea).tag();
    var tag = context.sheet.getTag(context.row, context.col, context.sheetArea);
    if (tag !== true) {
        tag = false;
    }
    basePaint.apply(this, [ctx, tag, x, y, width, height, style, context]);
};
MyCheckBoxCellType.prototype.getHitInfo = function(x, y, cellStyle, cellRect, context) {
    if (context) {
        return {
            x: x,
            y: y,
            row: context.row,
            col: context.col,
            cellRect: cellRect,
            sheetArea: context.sheetArea,
            isReservedLocation: true,
            sheet: context.sheet
        };
    }
    return null;
};

MyCheckBoxCellType.prototype.processMouseUp = function(hitInfo) {

    var sheet = hitInfo.sheet,
        row = hitInfo.row,
        col = hitInfo.col,
        sheetArea = hitInfo.sheetArea;

    if (sheet.getCell(0, 0, GC.Spread.Sheets.SheetArea.colHeader).locked() && sheet.options.isProtected) {
        return;
    }

    var tag = sheet.getTag(row, col, sheetArea);
    //var tag = sheet.getColumn(col,sheetArea).tag();
    if (tag === undefined || tag === null) { //first time
        sheet.setTag(row, col, true, sheetArea);
        //sheet.getColumn(col,sheetArea).tag(true)
    } else {
        sheet.setTag(row, col, !tag, sheetArea);
        //sheet.getColumn(col,sheetArea).tag(!tag)
    }
    //add your code here
    tag = sheet.getTag(row, col, sheetArea);
    //tag = sheet.getColumn(col,sheetArea).tag();
    // sheet.setValue(-1, 0, tag);
    sheet.suspendPaint();
    for (var i = 0; i < sheet.getRowCount(); i++) {
        var cell = sheet.getCell(i, col);
        if (cell.cellType() instanceof GC.Spread.Sheets.CellTypes.CheckBox) {
            cell.value(tag);
        }
    }
    sheet.resumePaint();
};
```


#### 关于 SpreadJS
[SpreadJS]( https://www.grapecity.com.cn/developer/spreadjs) 是一款基于 HTML5 的纯前端表格控件，兼容 450 多种 Excel 公式，具备“高性能、跨平台、与 Excel 高度兼容”的产品特性。使用 SpreadJS，可直接在 Angular、 React、 Vue 等前端框架中实现高效的模板设计、在线编辑和数据绑定等功能，为最终用户提供高度类似 Excel 的使用体验。
