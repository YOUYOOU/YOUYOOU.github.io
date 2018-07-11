---
layout: post
title:  "Java httpclient实现post和get请求"
categories: java
tags:  httpclient java springBoot
author: YOYO
---

* content
{:toc}

这个也是基础咯，我就整理一下。有post方式和get方式。其中post有参数是json形式和param形式的以及url后面接参数的。
接下来会一一整理一下。





1.post接收格式为param

```java
  private static String doPostWithParams(String url, Map<String, String> params, String appkey, String password, String host) {
    StringBuffer response = new StringBuffer();
    HttpClient client = new HttpClient();
    PostMethod method = new PostMethod(url);
    try {
      // nonce
      String nonce = MD5Util.string2MD5(UUIDUtil.generateUUID());
      // created
      String created = getUTCTimeStr();
      String digestStr = nonce + created + password;
      MessageDigest mDigest = MessageDigest.getInstance(DEFAULT_HASH_ALGORITHM);
      String pDigest = Base64.encodeBase64String(mDigest.digest(digestStr.getBytes()));
      // 请求header
      method.setRequestHeader("Host", host);
      method.setRequestHeader("Authorization", "WSSE realm=\"SDP\", profile=\"UsernameToken\", type=\"Appkey\"");
      method.setRequestHeader("X-WSSE", "UsernameToken Username=\"" + appkey + "\", PasswordDigest=\"" + pDigest + "\", Nonce=\""
          + nonce + "\", Created=\"" + created + "\"");
      method.setRequestHeader("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
      // 请求body
      if (params != null) {
        for (Map.Entry<String, String> entry : params.entrySet()) {
          method.addParameter(entry.getKey(), entry.getValue());
        }
      }
      // http请求发送
      client.executeMethod(method);
      BufferedReader reader = new BufferedReader(new InputStreamReader(method.getResponseBodyAsStream(), "utf-8"));
      String tmp = null;
      while ((tmp = reader.readLine()) != null) {
        response.append(tmp + "\r\n");
      }
    } catch (Exception e) {
    } finally {
      method.releaseConnection();
    }
    return response.toString();
  }

```

2.接收参数为json

```java
  private static String doPostWithJson(String url, String jsonObject, String appkey, String password, String host) {
    StringBuffer response = new StringBuffer();
    HttpClient client = new HttpClient();
    PostMethod method = new PostMethod(url);

    try {
      // nonce
      String nonce = MD5Util.string2MD5(UUIDUtil.generateUUID());
      // created
      String created = getUTCTimeStr();
      String digestStr = nonce + created + password;
      MessageDigest mDigest = MessageDigest.getInstance(DEFAULT_HASH_ALGORITHM);
      String pDigest = Base64.encodeBase64String(mDigest.digest(digestStr.getBytes()));
      // 请求header
      method.setRequestHeader("Host", host);
      method.setRequestHeader("Authorization", "WSSE realm=\"SDP\",profile=\"UsernameToken\", type=\"Appkey\"");
      method.setRequestHeader("X-WSSE", "UsernameToken Username=\"" + appkey + "\", PasswordDigest=\"" + pDigest + "\", Nonce=\""
          + nonce + "\", Created=\"" + created + "\"");

      // 请求body
      if (jsonObject != null) {
        RequestEntity se = new StringRequestEntity(jsonObject, "application/json", "UTF-8");
        method.setRequestEntity(se);
      }
      // http请求发送
      client.executeMethod(method);
      BufferedReader reader = new BufferedReader(new InputStreamReader(method.getResponseBodyAsStream(), "utf-8"));
      String tmp = null;
      while ((tmp = reader.readLine()) != null) {
        response.append(tmp + "\r\n");
      }

    } catch (Exception e) {
      logger.error("执行HTTP Post请求" + url + "时，发生异常！", e);
    } finally {
      method.releaseConnection();
    }
    return response.toString();
  }

```

3.post头带有参数，并且xml参数形式

```java

  private String JdResolveCustomerPin(CustomerPinForm form, PrincipalApiLog principalApiLog) throws UnsupportedEncodingException {
    // 拿到bodysign_method
    Document document = DocumentHelper.createDocument();
    // 添加节点信息
    Element rootElement = document.addElement("request");
    Element customerId = rootElement.addElement("customerId");
    Element customerPin = rootElement.addElement("customerPin");
    customerId.setText(form.getCustomerId());
    customerPin.setText(form.getCustomerPin());
    String xmlString = document.asXML();
    String url = buf.toString();
    String sign = SignatureUtil.sign(url, xmlString, config.getSecretKey());
    buf.append("&sign=").append(sign);
    url = buf.toString();
    // 创建httpclient工具对象
    HttpClient client = new HttpClient();
    HashMap<String, String> map = new HashMap<String, String>();
    map.put("method", jdResolveCustomerPin);
    map.put("timestamp", timestamp);
    map.put("v", v);
    map.put("format", format);
    map.put("app_key", config.getAppKey());
    map.put("customerId", form.getCustomerId());
    map.put("sign_method", sign_method);
    map.put("sign", sign);
    StringBuilder q = new StringBuilder("");
    for (String key : map.keySet()) {
      q.append(key + "=" + URLEncoder.encode(map.get(key), "UTF-8") + "&");
    }
    principalApiLog.setTransferParam(q.toString());
    logger.info("+++++++++++++post方法，地址【" + config.getJdurl() + q.toString() + "】+++++++++");
    // 创建post请求方法
    PostMethod myPost = new PostMethod(config.getJdurl() + q.toString());
    String responseString = null;
    try {
      // 设置请求头部类型
      myPost.setRequestHeader("Content-Type", "application/xml");
      myPost.setRequestHeader("charset", "utf-8");
      RequestEntity se = new StringRequestEntity(xmlString, "application/xml", "UTF-8");
      myPost.setRequestEntity(se);
      client.executeMethod(myPost);
      InputStream inputStream = myPost.getResponseBodyAsStream();
      BufferedReader br = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
      StringBuffer stringBuffer = new StringBuffer();
      String str = "";
      while ((str = br.readLine()) != null) {
        stringBuffer.append(str);
      }
      responseString = stringBuffer.toString();
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      myPost.releaseConnection();
    }
    return responseString;
  }

```

4.get请求

```java

  private static String doGet(String url) {
    StringBuffer response = new StringBuffer();
    HttpClient client = new HttpClient();

    GetMethod method = new GetMethod(url);
    try {
      // 请求header
      method.setRequestHeader("Authorization", "WSSE realm=\"SDP\", profile=\"UsernameToken\", type=\"Appkey\"");
      method.setRequestHeader("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
      // http请求发送
      client.executeMethod(method);
      BufferedReader reader = new BufferedReader(new InputStreamReader(method.getResponseBodyAsStream(), "utf-8"));
      String tmp = null;
      while ((tmp = reader.readLine()) != null) {
        response.append(tmp + "\r\n");
      }
    } catch (Exception e) {
      logger.error("+++++++++++++++++调用接口" + url + "失败！+++++++++++++");
    } finally {
      method.releaseConnection();
    }
    return response.toString();
  }

```
