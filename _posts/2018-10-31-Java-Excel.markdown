---
layout: post
title:  "Java操作Excel"
date:   2018-10-31 22:14:54
categories: Java
tags: Java Excel
---

* content
{:toc}

自己在工作中用到Java操作Excel。




## 简介

操作Excel有两种常用的工具，即Jakarta POI(POI)和Java Excel(JXL)。

Jakarta POI 是一套用于访问微软格式文档的Java API。Jakarta POI有很多组件组成，其中有用于操作Excel格式文件的HSSF和用于操作Word的HWPF，在各种组件中目前只有用于操作Excel的HSSF相对成熟。官方主页[http://poi.apache.org/index.html](http://poi.apache.org/index.html)，API文档[http://poi.apache.org/apidocs/index.html](http://poi.apache.org/apidocs/index.html)。

Java Excel是一开放源码项目，通过它Java开发人员可以读取Excel文件的内容、创建新的Excel文件、更新已经存在的Excel文件。jxl 由于其小巧 易用的特点, 逐渐已经取代了 POI-excel的地位, 成为了越来越多的java开发人员生成excel文件的首选。这套API是纯Java的，并不依赖Windows系统，即使运行在Linux下，它同样能够正确的处理Excel文件。另外需要说明的是，这套API对图形和图表的支持很有限，而且仅仅识别PNG格式。在线帮助文档[http://jexcelapi.sourceforge.net/resources/javadocs/2_6_10/docs/index.html](http://jexcelapi.sourceforge.net/resources/javadocs/2_6_10/docs/index.html)。

## 开始使用

手动poi或jxl方式需要的jar包到项目,用Maven管理的项目导入依赖到pom.xml文件中。

```
<!-- poi方式 -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>${poi.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>${poi-ooxml.version}</version>
</dependency>
<!-- jxl方式-->
<dependency>
    <groupId>net.sourceforge.jexcelapi</groupId>
    <artifactId>jxl</artifactId>
    <version>${jxl.version}</version>
</dependency>
```

## jxl方式

### 写入Excel

```
import java.io.File;
import java.io.IOException;

import jxl.Workbook;
import jxl.write.Label;
import jxl.write.WritableSheet;
import jxl.write.WritableWorkbook;
import jxl.write.WriteException;

/**
 * jxl写Excel
 * @author panying
 * 
 */
public class JxlWriteDemo
{
   public static void main(String[] args) throws IOException, WriteException
   {
      File xlsFile = new File("jxl.xls");
      // 创建一个工作簿
      WritableWorkbook workbook = Workbook.createWorkbook(xlsFile);
      // 创建一个工作表
      WritableSheet sheet = workbook.createSheet("sheet1", 0);
      for (int row = 0; row < 10; row++)
      {
         for (int col = 0; col < 10; col++)
         {
            // 向工作表中添加数据
            sheet.addCell(new Label(col, row, "data" + row + col));
         }
      }
      workbook.write();
      workbook.close();
   }
}
```

### 读取Excel

```
import java.io.File;
import java.io.IOException;

import jxl.Sheet;
import jxl.Workbook;
import jxl.read.biff.BiffException;

/**
 * jxl读excel
 * @author panying
 * 
 */
public class JxlReadDemo
{
   public static void main(String[] args) throws BiffException, IOException
   {
      File xlsFile = new File("jxl.xls");
      // 获得工作簿对象
      Workbook workbook = Workbook.getWorkbook(xlsFile);
      // 获得所有工作表
      Sheet[] sheets = workbook.getSheets();
      // 遍历工作表
      if (sheets != null)
      {
         for (Sheet sheet : sheets)
         {
            // 获得行数
            int rows = sheet.getRows();
            // 获得列数
            int cols = sheet.getColumns();
            // 读取数据
            for (int row = 0; row < rows; row++)
            {
               for (int col = 0; col < cols; col++)
               {
                  System.out.printf("%10s", sheet.getCell(col, row)
                        .getContents());
               }
               System.out.println();
            }
         }
      }
      workbook.close();
   }
}
```

## poi方式

### 写入Excel

```
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;

/**
 * Poi写Excel
 * 
 * @author panying
 * 
 */
public class PoiWriteDemo
{
   public static void main(String[] args) throws IOException
   {
      // 创建工作薄
      HSSFWorkbook workbook = new HSSFWorkbook();
      // 创建工作表
      HSSFSheet sheet = workbook.createSheet("sheet1");

      for (int row = 0; row < 10; row++)
      {
         HSSFRow rows = sheet.createRow(row);
         for (int col = 0; col < 10; col++)
         {
            // 向工作表中添加数据
            rows.createCell(col).setCellValue("data" + row + col);
         }
      }

      File xlsFile = new File("poi.xls");
      FileOutputStream xlsStream = new FileOutputStream(xlsFile);
      workbook.write(xlsStream);
   }
}
```

### 读取Excel

```
import java.io.File;
import java.io.IOException;

import org.apache.poi.openxml4j.exceptions.InvalidFormatException;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.ss.usermodel.WorkbookFactory;

/**
 * Poi写Excel
 * 
 * @author jianggujin
 * 
 */
public class PoiReadDemo
{
   public static void main(String[] args) throws IOException,
         InvalidFormatException
   {
      File xlsFile = new File("poi.xls");
      // 获得工作簿
      Workbook workbook = WorkbookFactory.create(xlsFile);
      // 获得工作表个数
      int sheetCount = workbook.getNumberOfSheets();
      // 遍历工作表
      for (int i = 0; i < sheetCount; i++)
      {
         Sheet sheet = workbook.getSheetAt(i);
         // 获得行数
         int rows = sheet.getLastRowNum() + 1;
         // 获得列数，先获得一行，在得到改行列数
         Row tmp = sheet.getRow(0);
         if (tmp == null)
         {
            continue;
         }
         int cols = tmp.getPhysicalNumberOfCells();
         // 读取数据
         for (int row = 0; row < rows; row++)
         {
            Row r = sheet.getRow(row);
            for (int col = 0; col < cols; col++)
            {
               System.out.printf("%10s", r.getCell(col).getStringCellValue());
            }
            System.out.println();
         }
      }
   }
}
```