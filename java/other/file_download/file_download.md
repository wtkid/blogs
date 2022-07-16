> 服务器文件下载

​
内容比较简单，直接上代码解释了。

文件下载使用get方式，传参数需要使用?key=falue&key=value的形式跟在url后面，所有最简单明了的方式就是直接将参数慢慢拼接，然后发起get请求。

这里主要讲第二种方式，使用form表单。

> Controller

```java
@RequestMapping("/exportExcel")
public void exportExcel(SeparateMessageOrderMO separateMessageOrderMO, HttpServletResponse response) throws IOException {
    List<SeparateMessageOrderMO> list = separateMessageOrderService.findByPage(separateMessageOrderMO, new PageQuery());
    String filename = "导入模板.xls";
    response.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);
    response.setHeader("Content-Disposition", "attachment;filename=" + new String(filename.getBytes("UTF-8"), "ISO8859-1"));
    String[] headers ={"header1","header2"};
    try {
        List<CellDefine> column = new ArrayList<>();
        for (String header : headers) {
        column.add(new CellDefine(header, true));
    }
        ExcelExport.exportToOutputStream(column, null, ExcelExport.ExcelType.XLS, response.getOutputStream());
    } catch (IOException e) {
        throw new BusinessException("generategenerateTemp happened error", e);
    }
}
```

> form表单

```html
<form action="" method="get">
  <p>startTime<input type="text" name="startTime" /></p>
  <p>entTime<input type="text" name="endTime" /></p>
</form>
<button class="btn btn-primary" id="exportExcel" >导出</button>
```

> js

```js
$("#exportExcel").bind("click", function () {
$("#search_form").attr("action", contextPath+"/separateManagement/exportExcel");//改变表单的提交地址为下载的地址
$("#search_form").submit();//提交表单
});
```


这种方式，form表单的method对于post还是get都可以。这里也可以直接在form表单里面添加一个submit类型的input，不过我个人比较喜欢使用外边的button标签而已。

over~~~~
