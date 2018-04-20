---
layout: post
title:  "关于js判断日期"
categories: JavaScript
tags:  js
author: YOYO
---

* content
{:toc}

## 目的：

这篇文章的由来主要是客户不想用时间控件，想要text直接输入时间。。。




## 测试js：

```js

function isdig(s)
{
	var regu = "^([1-9]*[0-9]*)$"
	var re = new RegExp(regu);
	if (s.search(re) != -1){
		return true;
	}else{
		return false;
	}
}

function check_date(datestr)
{

	var datetime_arr,date_arr,time_arr,year,mon,day;
	var monthDays = new Array(31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31);
	datetime_arr=datestr.split(" ")
	//Date
	date_arr=datetime_arr[0].split("-");
	year=date_arr[0];
	mon=date_arr[1];
	day=date_arr[2];		
	if(((year % 4 == 0) && (year % 100 != 0)) || (year % 400 == 0)) 
	   monthDays[1]=29;
	   alert(!isdig(year));
	   alert(mon);
	   alert(day);
	   alert(monthDays[mon-1])
	if(!isdig(year) || !isdig(mon) || !isdig(day) || mon<1 || mon>12 || day>monthDays[mon-1]){ 
	//Time
	time_arr=datetime_arr[1].split(":");
	hour=time_arr[0];
	min=time_arr[1];
	sec=time_arr[2];
	if (!isdig(hour) || !isdig(min) || !isdig(sec) || hour<0 || hour>23 || min<0 || min>59 || sec<0 || sec>59){
	     alert("正确");
	   
	}else{
	//Passed all check
	alert("时分秒错误啦");
	}
	}else{
	alert("年月日错误啦");
	}
}

```

以上只是一个小测试，在此记录一下客户的奇葩需求！！

但是客户至上哦~

