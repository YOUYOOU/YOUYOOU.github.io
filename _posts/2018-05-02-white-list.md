---
layout: post
title:  "IP白名单"
categories: java
tags:  白名单 java spring boot
author: YOYO
---

* content
{:toc}

网站刚上限进行测试的时候，需要指定IP才能够进入系统，这里讲讲怎样spring boot怎么限制IP
```js
inetaddress.config.allowIp=

```


1.配置文件配置白名单IP
```js
inetaddress.config.allowIp=

```
2.配置Configurer文件
```java
@Configuration
public class SecurityConfigurer extends WebMvcConfigurerAdapter {

  @Bean
  public InetAddressInterceptor inetAddressInterceptor() {
    return new InetAddressInterceptor();
  }

  @Override
  public void addInterceptors(InterceptorRegistry registry) {
  	// 哪些接口需要配置白名单
    registry.addInterceptor(inetAddressInterceptor()).addPathPatterns("/v1/").addPathPatterns("/v1/user/**");
    super.addInterceptors(registry);
  }

}
```
3.定义配置文件中的字段
```java
@ConfigurationProperties(prefix = InetAddressConfiguration.PREFIX)
public class InetAddressConfiguration {

  public static final String PREFIX = "inetaddress.config";

  // 允许访问IP地址
  private String allowIp;

  /**
   * @return the allowIp
   */
  public String getAllowIp() {
    return allowIp;
  }

  /**
   * @param allowIp
   *          the allowIp to set
   */
  public void setAllowIp(String allowIp) {
    this.allowIp = allowIp;
  }

}
```
4.白名单逻辑
```java
public class InetAddressInterceptor implements HandlerInterceptor {

  // 允许访问IPconfig
  @Autowired(required = false)
  InetAddressConfiguration config;

  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    String ip = request.getHeader("x-forwarded-for");
    // 获取客户端IP
    if (ip == null) {
      ip = request.getRemoteAddr();
    }
    boolean blnResult = false;
    if (StringUtils.isBlank(config.getAllowIp())) {
      blnResult = true;
    } else {
      String[] allowIpList = config.getAllowIp().split(",");
      int intStar = 0;
      for (String allowIp : allowIpList) {
        if (allowIp.equals(ip)) {
          blnResult = true;
          break;
        }

        intStar = allowIp.indexOf("*");
        // 允许IP地址出现*的情况
        if (intStar == 0) {
          // 不check IP地址
          blnResult = true;
          break;
        } else if (intStar > 0) {
          // *前面是否完全一致
          if (allowIp.substring(0, intStar).equals(ip.substring(0, intStar))) {
            blnResult = true;
            break;
          }
        }
      }
    }
    if (!blnResult) {
      response.setContentType("application/json, charset=UTF-8");
      response.getWriter().print(toJsonErrorString(ApiResponseCode.status_403));
      return false;
    }

    return true;
  }

  @Override
  public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
      throws Exception {

  }

  @Override
  public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
      throws Exception {

  }

  private String toJsonErrorString(ApiResponseCode responseCode) {
    StringBuilder builder = new StringBuilder();
    builder.append("{");
    builder.append("\"" + "code" + "\"");
    builder.append(":");
    builder.append(responseCode.getCode());
    builder.append(",");
    builder.append("\"" + "message" + "\"");
    builder.append(":");
    builder.append("\"" + responseCode.getMessage() + "\"");
    builder.append("}");
    return builder.toString();
  }

}
```java
   
