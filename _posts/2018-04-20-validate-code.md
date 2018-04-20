---
layout: post
title:  "图形验证码"
categories: java
tags:  java springboot 验证码
author: YOYO
---

* content
{:toc}

很多网站注册的时候会加一个图形验证码，防止机器人自动注册，但是前端生成的验证码会暴露数据无法防止机器人，
需要后端生成验证码图片发送给前端，再通过后端进行验证。今天就是用springboot实现图形验证码。



## 1.实现图形验证码：

```java

public class ValidateCode {  
    // 图片的宽度。  
    private int width = 160;  
    // 图片的高度。  
    private int height = 40;  
    // 验证码字符个数  
    private int codeCount = 5;  
    // 验证码干扰线数  
    private int lineCount = 150;  
    // 验证码  
    private String code = null;  
    // 验证码图片Buffer  
    private BufferedImage buffImg = null;  
  
    // 验证码范围,去掉0(数字)和O(拼音)容易混淆的(小写的1和L也可以去掉,大写不用了)  
    private char[] codeSequence = {'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J',  
            'K', 'L', 'M', 'N', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W',  
            'X', 'Y', 'Z', '1', '2', '3', '4', '5', '6', '7', '8', '9'};  
  
    /** 
     * 默认构造函数,设置默认参数 
     */  
    public ValidateCode() {  
        this.createCode();  
    }  
  
    /** 
     * @param width  图片宽 
     * @param height 图片高 
     */  
    public ValidateCode(int width, int height) {  
        this.width = width;  
        this.height = height;  
        this.createCode();  
    }  
  
    /** 
     * @param width     图片宽 
     * @param height    图片高 
     * @param codeCount 字符个数 
     * @param lineCount 干扰线条数 
     */  
    public ValidateCode(int width, int height, int codeCount, int lineCount) {  
        this.width = width;  
        this.height = height;  
        this.codeCount = codeCount;  
        this.lineCount = lineCount;  
        this.createCode();  
    }  
  
    public void createCode() {  
        int x = 0, fontHeight = 0, codeY = 0;  
        int red = 0, green = 0, blue = 0;  
  
        x = width / (codeCount + 2);//每个字符的宽度(左右各空出一个字符)  
        fontHeight = height - 2;//字体的高度  
        codeY = height - 4;  
  
        // 图像buffer  
        buffImg = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);  
        Graphics2D g = buffImg.createGraphics();  
        // 生成随机数  
        Random random = new Random();  
        // 将图像填充为白色  
        g.setColor(Color.WHITE);  
        g.fillRect(0, 0, width, height);  
        // 创建字体,可以修改为其它的  
        Font font = new Font("Fixedsys", Font.PLAIN, fontHeight);  
        g.setFont(font);  
  
        for (int i = 0; i < lineCount; i++) {  
            // 设置随机开始和结束坐标  
            int xs = random.nextInt(width);//x坐标开始  
            int ys = random.nextInt(height);//y坐标开始  
            int xe = xs + random.nextInt(width / 8);//x坐标结束  
            int ye = ys + random.nextInt(height / 8);//y坐标结束  
  
            // 产生随机的颜色值，让输出的每个干扰线的颜色值都将不同。  
            red = random.nextInt(255);  
            green = random.nextInt(255);  
            blue = random.nextInt(255);  
            g.setColor(new Color(red, green, blue));  
            g.drawLine(xs, ys, xe, ye);  
        }  
  
        // randomCode记录随机产生的验证码  
        StringBuffer randomCode = new StringBuffer();  
        // 随机产生codeCount个字符的验证码。  
        for (int i = 0; i < codeCount; i++) {  
            String strRand = String.valueOf(codeSequence[random.nextInt(codeSequence.length)]);  
            // 产生随机的颜色值，让输出的每个字符的颜色值都将不同。  
            red = random.nextInt(255);  
            green = random.nextInt(255);  
            blue = random.nextInt(255);  
            g.setColor(new Color(red, green, blue));  
            g.drawString(strRand, (i + 1) * x, codeY);  
            // 将产生的四个随机数组合在一起。  
            randomCode.append(strRand);  
        }  
        // 将四位数字的验证码保存到Session中。  
        code = randomCode.toString();  
    }  
  
    public void write(String path) throws IOException {  
        OutputStream sos = new FileOutputStream(path);  
        this.write(sos);  
    }  
  
    public void write(OutputStream sos) throws IOException {  
        ImageIO.write(buffImg, "png", sos);  
        sos.close();  
    }  
  
    public BufferedImage getBuffImg() {  
        return buffImg;  
    }  
  
    public String getCode() {  
        return code;  
    }  
  
    /** 
     * 测试函数,默认生成到d盘 
     * @param args 
     */  
    public static void main(String[] args) {  
        ValidateCode vCode = new ValidateCode(160,40,5,150);  
        try {  
            String path="D:/"+new Date().getTime()+".png";  
            System.out.println(vCode.getCode()+" >"+path);  
            vCode.write(path);  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }  
}  

```

## 2.前端调用验证码：

```html

<div class="form-group  col-lg-6">  
    <label for="id" class="col-sm-4 control-label">  
        验证码:  
    </label>  
    <div class="col-sm-8">  
        <input type="text" id="code" name="code" class="form-control" style="width:250px;"/>  
        <img id="imgObj" alt="验证码" src="/article/validateCode" onclick="changeImg()"/>  
        <a href="#" onclick="changeImg()">换一张</a>  
    </div>  
</div>  
  
<script type="text/javascript">  
    // 刷新图片  
    function changeImg() {  
        var imgSrc = $("#imgObj");  
        var src = imgSrc.attr("src");  
        imgSrc.attr("src", changeUrl(src));  
    }  
    //为了使每次生成图片不一致，即不让浏览器读缓存，所以需要加上时间戳  
    function changeUrl(url) {  
        var timestamp = (new Date()).valueOf();  
        var index = url.indexOf("?",url);  
        if (index > 0) {  
            url = url.substring(0, url.indexOf(url, "?"));  
        }  
        if ((url.indexOf("&") >= 0)) {  
            url = url + "×tamp=" + timestamp;  
        } else {  
            url = url + "?timestamp=" + timestamp;  
        }  
        return url;  
    }  
</script>

```
## controller层输入图形验证码

```Java
/** 
 * 响应验证码页面 
 * @return 
 */  
@RequestMapping(value="/validateCode")  
public String validateCode(HttpServletRequest request,HttpServletResponse response) throws Exception{  
    // 设置响应的类型格式为图片格式  
    response.setContentType("image/jpeg");  
    //禁止图像缓存。  
    response.setHeader("Pragma", "no-cache");  
    response.setHeader("Cache-Control", "no-cache");  
    response.setDateHeader("Expires", 0);  
  
    HttpSession session = request.getSession();  
  
    ValidateCode vCode = new ValidateCode(120,40,5,100);  
    session.setAttribute("code", vCode.getCode());  
    vCode.write(response.getOutputStream());  
    return null;  
}

```

## controller层验证验证码输入是否正确

```Java

    String code = request.getParameter("code");  
    HttpSession session = request.getSession();  
    String sessionCode = (String) session.getAttribute("code");  
    if (!StringUtils.equalsIgnoreCase(code, sessionCode)) {  //忽略验证码大小写  
        throw new RuntimeException("验证码对应不上code=" + code + "  sessionCode=" + sessionCode);  
    } 
     
```    
这里讲验证码存储在session里面，当然也可以不使用session，使用Redis也是可以的。