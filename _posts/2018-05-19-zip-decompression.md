---
layout: post
title:  "java zip解压"
categories: java
tags:  excel java springBoot
author: YOYO
---

* content
{:toc}

上传文件为了避免多次上传，会选择上传压缩文件。那么解压就必不可少了。
这里只要介绍一下怎么解压zip文件。需要注意的是，有些不能解压文件夹，只能解压文件。
以下介绍的是直接解压文件的。




1.后端代码

```java
  /**
   * 文件解压 输入文件路径(D:/tsv/memberprice/discount.zip)
   * 
   * @param inputPath
   *          输出文件夹路径(D:/tsv/memberprice/)
   * @param outPath
   */
  public static void zipDecompress(String inputFilePath, String outPath) {
    try {
      // 输入源zip路径
      ZipInputStream Zin = new ZipInputStream(new FileInputStream(inputFilePath), Charset.forName("GBK"));
      BufferedInputStream Bin = new BufferedInputStream(Zin);
      // 输出路径（文件夹目录）
      String Parent = outPath;
      File Fout = null;
      ZipEntry entry;
      try {
        while ((entry = Zin.getNextEntry()) != null && !entry.isDirectory()) {
          Fout = new File(Parent, entry.getName());
          if (!Fout.exists()) {
            (new File(Fout.getParent())).mkdirs();
          }
          FileOutputStream out = new FileOutputStream(Fout);
          BufferedOutputStream Bout = new BufferedOutputStream(out);
          int b;
          while ((b = Bin.read()) != -1) {
            Bout.write(b);
          }
          Bout.close();
          out.close();
        }
        Bin.close();
        Zin.close();
      } catch (IOException e) {
        logger.debug("错误" + e);
        e.printStackTrace();
      }
    } catch (FileNotFoundException e) {
      logger.debug("错误" + e);
      e.printStackTrace();
    }
  }

  /**
   * 解压
   * 
   * @param sftpPath
   * @param storeFilePath
   */
  public static void Decompress(String sftpPath, String filePath) {

    Resource[] resourcesArr = null;
    ResourcePatternResolver resourceLoader = new PathMatchingResourcePatternResolver();
    // 查询匹配文件:
    try {
      resourcesArr = resourceLoader.getResources("file:" + sftpPath);
    } catch (IOException e) {
      logger.debug("错误" + e);
      e.printStackTrace();
    }
    if (resourcesArr.length != 0) {
      for (Resource resource : resourcesArr) {
        try {
          File file = resource.getFile();
          // 如果resource是文件
          if (file.isFile()) {
            // 如果文件后缀名是zip 则解压
            if (file.getName().endsWith(".zip")) {
              try {
                // 文件解压
                ZipUtil.zipDecompress(sftpPath, filePath);
                file.delete();
                file.deleteOnExit();
              } catch (Exception e2) {
                logger.error(file.getPath() + "文件解压失败" + e2);
                continue;
              }
            }
          } else {
            logger.info(file.getPath() + file.getName() + "不是文件");
            throw new IllegalArgumentException(file.getPath() + file.getName() + "不是文件");
          }
        } catch (Exception e) {
          logger.error("文件解压失败" + e);
        }
      }
    }
  }
```