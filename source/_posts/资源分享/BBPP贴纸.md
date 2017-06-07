---
layout: post
title: BBPP智能贴纸参数
date: 2016-01-23
category: 资源
tags: BBPP
keywords: BBPP
description: BBPP智能贴纸参数
---

## 说明

参数分别为x坐标，y坐标，高度，旋转角度

相对坐标，偏移的位置占宽或者高的比重（0，1），左右开区间表示相对坐标

x 可以使用相对或者绝对坐标

y 可以使用相对或者绝对坐标

height 必须使用相对距离，宽度进行等比例适应

rotate 旋转角度，绝对数值

## 不同尺寸图片绝对坐标换算

图片尺寸变化，横坐标不需要换算，纵坐标需要换算

bitH 图片高度

bitW 图片宽度

float ratio = (bitH * 1.0f / bitW) * (3.0f / 4f);

如果是绝对坐标需要   纵坐标＊ratio


## <font color = "red">第一套</font>

### 左上位置

昵称 			10, -15, 0.125f, 0

左上角日期  	-5, 60f, 0.125f, 0

箭头 			0.2f, 110f, 0.042f, 30

出生天数,显示多张

照片时间 		0.54f, 0.1f, 0.07f, 20f

### 左下位置

身高 			12, 50f, 0.1f, 0

体重  			12, 0f, 0.1f, 0

---
## <font color = "red">第二套</font>
### 左上位置

外框 			0, -5, 0.999f, 0

文字 			0, 0.125f, 0.125f, 0

### 左下位置

爸爸 			40f, 40f, 0.14f, 0

### 左中间位置

体重 			20, -40, 0.18f, 0

花的line 		10, 15f, 0.08f, 0

身高 			20, 55f, 0.125f, 0

男女 			20, 105f, 0.09f, 0

### 右下位置

妈妈 			40f, 40f, 0.14f, 0

### 右中间位置

月 				0, -75, 0.1f, 0

日 				20, -42, 0.07f, 0

年  			25, 0, 0.11f, 0

星座 			20, 57, 0.08f, 0

生肖 			20, 105f, 0.09f, 0



---

```c
//生肖算法
char *GetDayOf(PSYSTEMTIME pSt) 
{ 
/*天干名称*/ 
const char *cTianGan[] = {"甲","乙","丙","丁","戊","己","庚","辛","壬","癸"}; 
/*地支名称*/ 
const char *cDiZhi[] = {"子","丑","寅","卯","辰","巳","午", 
"未","申","酉","戌","亥"}; 
/*属相名称*/ 
const char *cShuXiang[] = {"鼠","牛","虎","兔","龙","蛇", 
"马","羊","猴","鸡","狗","猪"}; 
/*农历日期名*/ 
const char *cDayName[] = {"*","初一","初二","初三","初四","初五", 
"初六","初七","初八","初九","初十", 
"十一","十二","十三","十四","十五", 
"十六","十七","十八","十九","二十", 
"廿一","廿二","廿三","廿四","廿五", 
"廿六","廿七","廿八","廿九","三十"}; 
/*农历月份名*/ 
const char *cMonName[] = {"*","正","二","三","四","五","六", 
"七","八","九","十","十一","腊"}; 
 
/*公历每月前面的天数*/ 
const int wMonthAdd[12] = {0,31,59,90,120,151,181,212,243,273,304,334}; 
/*农历数据*/ 
const int wNongliData[100] = 
{2635,333387,1701,1748,267701,694,2391,133423,1175,396438 
,3402,3749,331177,1453,694,201326,2350,465197,3221,3402 
,400202,2901,1386,267611,605,2349,137515,2709,464533,1738 
,2901,330421,1242,2651,199255,1323,529706,3733,1706,398762 
,2741,1206,267438,2647,1318,204070,3477,461653,1386,2413 
,330077,1197,2637,268877,3365,531109,2900,2922,398042,2395 
,1179,267415,2635,661067,1701,1748,398772,2742,2391,330031 
,1175,1611,200010,3749,527717,1452,2742,332397,2350,3222 
,268949,3402,3493,133973,1386,464219,605,2349,334123,2709 
,2890,267946,2773,592565,1210,2651,395863,1323,2707,265877}; 
static int wCurYear,wCurMonth,wCurDay; 
static int nTheDate,nIsEnd,m,k,n,i,nBit; 
TCHAR szNongli[30], szNongliDay[10],szShuXiang[10]; 
/*---取当前公历年、月、日---*/ 
wCurYear = pSt->wYear; 
wCurMonth = pSt->wMonth; 
wCurDay = pSt->wDay; 
/*---计算到初始时间1921年2月8日的天数：1921-2-8(正月初一)---*/ 
nTheDate = (wCurYear - 1921) * 365 + (wCurYear - 1921) / 4 + wCurDay + wMonthAdd 
[wCurMonth - 1] - 38; 
if((!(wCurYear % 4)) && (wCurMonth > 2)) 
nTheDate = nTheDate + 1; 
 
/*--计算农历天干、地支、月、日---*/ 
nIsEnd = 0; 
m = 0; 
while(nIsEnd != 1) 
{ 
if(wNongliData[m] < 4095) 
k = 11; 
else 
k = 12; 
n = k; 
while(n>=0) 
{ 
//获取wNongliData(m)的第n个二进制位的值 
nBit = wNongliData[m]; 
for(i=1;i<n+1;i++) 
nBit = nBit/2; 
 
nBit = nBit % 2; 
 
if (nTheDate <= (29 + nBit)) 
{ 
nIsEnd = 1; 
break; 
} 
 
nTheDate = nTheDate - 29 - nBit; 
n = n - 1; 
} 
if(nIsEnd) 
break; 
m = m + 1; 
} 
wCurYear = 1921 + m; 
wCurMonth = k - n + 1; 
wCurDay = nTheDate; 
if (k == 12) 
{ 
if (wCurMonth == wNongliData[m] / 65536 + 1) 
wCurMonth = 1 - wCurMonth; 
else if (wCurMonth > wNongliData[m] / 65536 + 1) 
wCurMonth = wCurMonth - 1; 
} 
 
/*--生成农历天干、地支、属相 ==> wNongli--*/ 
wsprintf(szShuXiang,"%s",cShuXiang[((wCurYear - 4) % 60) % 12]); 
wsprintf(szNongli,"%s(%s%s)年",szShuXiang,cTianGan[((wCurYear - 4) % 60) % 
10],cDiZhi[((wCurYear - 4) % 60) % 12]); 
 
/*--生成农历月、日 ==> wNongliDay--*/ 
if (wCurMonth < 1) 
wsprintf(szNongliDay,"闰%s",cMonName[-1 * wCurMonth]); 
else 
strcpy(szNongliDay,cMonName[wCurMonth]); 
 
strcat(szNongliDay,"月"); 
strcat(szNongliDay,cDayName[wCurDay]); 
return strcat(szNongli,szNongliDay); 
 
} 
```