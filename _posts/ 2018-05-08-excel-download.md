---
layout: post
title:  "java excel下载"
categories: java
tags:  excel java springBoot
author: YOYO
---

* content
{:toc}

下载指定excel模板文件，在对应单元格补充内容。
这种excel最多出现在一些需要打印报表账单的系统，还是比较常见的。
简单记录一下。



1.后端代码

```java
  @RequestMapping(value = "/output", method = RequestMethod.GET)
  public Void outputExcel(HttpServletRequest request, HttpServletResponse response) {
    this.request = request;
    this.response = response;

    try {
    	// 模板的存放位置
      File newFile = new File("E:/excel/excel.xlsx");
      InputStream inputStream = new FileInputStream(newFile);

      // 读取模板
      XSSFWorkbook book = new XSSFWorkbook(inputStream);
        // 读取sheet
        XSSFSheet sheet;
        sheet = book.getSheetAt(0);
        XSSFRow row = sheet.getRow(1);
        XSSFCell cell = row.getCell(2);
        cell.setCellValue("AAA");

        XSSFRow row1 = sheet.getRow(3);
        XSSFCell cell11 = row1.getCell(2);
        XSSFCell cell12 = row1.getCell(5);
        XSSFCell cell13 = row1.getCell(6);
        cell11.setCellValue("BBBB");
        cell12.setCellValue("CCCC");
        cell13.setCellValue("2018年06月22日");

        XSSFRow row2 = sheet.getRow(5);
        XSSFCell cell21 = row2.getCell(2);
        XSSFCell cell22 = row2.getCell(4);
        XSSFCell cell23 = row2.getCell(7);
        cell21.setCellValue("****");
        cell22.setCellValue("****");
        cell23.setCellValue("xxxx");

        XSSFRow row3 = sheet.getRow(6);
        XSSFCell cell31 = row3.getCell(2);
        XSSFCell cell32 = row3.getCell(7);
        cell31.setCellValue("KKKK");
        cell32.setCellValue("MMMM");

      // 生成的文件名
      String fileName = DateUtil.toDateTimeString(DateUtil.getSysdate(), DateUtil.DEFAULT_DATE_FORMAT) + "_test" + ".xlsx";
      response.setContentType("application/octet-stream");
      response.setHeader("Content-Disposition", "attachment" + ";filename=\"" + fileName + "\"");
      OutputStream outputStream = response.getOutputStream();
      book.write(outputStream);
      outputStream.flush();
      book.close();
      outputStream.close();
    } catch (IOException e) {
    } catch (Exception e) {
    }
    return null;
  }

```

2.前端代码

```html

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Excel下载</title>
</head>
<body>

<div style="clear:left; padding-top:50px; padding-left:50px">
 <form id="sendUserCodeToLawson"  method="get" action="http:.../v1/excelOutput/output">
 	<table width="494" border="0"  cellpadding="5" cellspacing="5">
      <tr>
        <td colspan="2"><button type="submit">下载</button></td>
      </tr>
     
    </table>
 </form>
</div>
</body>
</html>

```