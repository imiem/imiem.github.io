---
layout: post
title:  "GeoHash实现周边推荐"
date:   2017-03-31 13:06:05
categories: Java
tags: LBS GeoHash
---

* content
{:toc}

旅游，外卖等需要定位的项目中一般会有周边推荐的需求，如推荐出周边五公里的景点。目前实现的算法也有很多，这里简单的说下GeoHash的实现原理以及Java的实现代码。





## GeoHash基础

* GeoHash使用一个字符串来表示经度和纬度。这样在做周边搜索的时候可以在一列上加索引。
* GeoHash表示的不是一个点，而是一个矩形区域。
* 精度范围为(-180,180),纬度范围为(-90,90)

## GeoHash转换过程

### 转换

GeoHash就是将经纬度信息，转换为可以排序、比较的字符串编码。
首先将纬度范围(-90,90)分为两个区间(-90,0),(0,90),如果目标纬度在前一个区间，编码为0，否则编码为1。以此类推，直到精度符合要求为止。

给出一个经纬度(39.92324,116.3906)，转换过程如下

<table boder="1px" cellspacing="0px" style="width:100%">
	<tr>
		<td>纬度范围</td>
		<td>划分区间0</td>
		<td>划分区间1</td>
		<td>39.92324所属区间</td>
	</tr>
	<tr>
		<td>(-90, 90)</td>
		<td>(-90, 0.0)</td>
		<td>(0.0, 90)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(0.0, 90)</td>
		<td>(0.0, 45.0)</td>
		<td>(45.0, 90)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(0.0, 45.0)</td>
		<td>(0.0, 22.5)</td>
		<td>(22.5, 45.0)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(22.5, 45.0)</td>
		<td>(22.5, 33.75)</td>
		<td>(33.75, 45.0)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(33.75, 45.0)</td>
		<td>(33.75, 39.375)</td>
		<td>(39.375, 45.0)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(39.375, 45.0)</td>
		<td>(39.375, 42.1875)</td>
		<td>(42.1875, 45.0)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.375, 42.1875)</td>
		<td>(39.375, 40.7812)</td>
		<td>(40.7812, 42.1875)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.375, 40.7812)</td>
		<td>(39.375, 40.0781)</td>
		<td>(40.0781, 40.7812)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.375, 40.0781)</td>
		<td>(39.375, 39.7265)</td>
		<td>(39.7265, 40.0781)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(39.7265, 40.0781)</td>
		<td>(39.7265, 39.9023)</td>
		<td>(39.9023, 40.0781)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(39.9023, 40.0781)</td>
		<td>(39.9023, 39.9902)</td>
		<td>(39.9902, 40.0781)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.9023, 39.9902)</td>
		<td>(39.9023, 39.9462)</td>
		<td>(39.9462, 39.9902)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.9023, 39.9462)</td>
		<td>(39.9023, 39.9243)</td>
		<td>(39.9243, 39.9462)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.9023, 39.9243)</td>
		<td>(39.9023, 39.9133)</td>
		<td>(39.9133, 39.9243)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(39.9133, 39.9243)</td>
		<td>(39.9133, 39.9188)</td>
		<td>(39.9188, 39.9243)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(39.9188, 39.9243)</td>
		<td>(39.9188, 39.9215)</td>
		<td>(39.9215, 39.9243)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(39.9215, 39.9243)</td>
		<td>(39.9215, 39.9229)</td>
		<td>(39.9229, 39.9243)</td>
		<td>1</td>
	</tr>
	<tr>
		<td>(39.9229, 39.9243)</td>
		<td>(39.9229, 39.9236)</td>
		<td>(39.9236, 39.9243)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.9229, 39.9236)</td>
		<td>(39.9229, 39.92325)</td>
		<td>(39.92325, 39.9236)</td>
		<td>0</td>
	</tr>
	<tr>
		<td>(39.9229, 39.92325)</td>
		<td>(39.9229, 39.923075)</td>
		<td>(39.923075, 39.92325)</td>
		<td>1</td>
	</tr>
</table>

因此可以得到纬度的编码为 1011 1000 1100 0111 1001
经度也用同样的算法，对(-180, 180)依次细分，得到116.3906的编码为1101 0010 1100 0100 0100。
接下来将经度和纬度的编码合并，奇数位是纬度，偶数位是经度，得到编码 11100 11101 00100 01111 00000 01101 01011 00001。




### 合并

将 11100 11101 00100 01111 00000 01101 01011 00001转化为对应的十进制为 28 29 4 15 0 13 11 1

用0-9、b-z（去掉a, i, l, o）这32个字母进行base32编码
<table boder="1px" cellspacing="0px" style="width:100%">
	
	<tr>
		<td>十进制</td>
		<td>0</td>
		<td>1</td>
		<td>2</td>
		<td>3</td>
		<td>4</td>
		<td>5</td>
		<td>6</td>
		<td>7</td>
		<td>8</td>
		<td>9</td>
		<td>10</td>
		<td>11</td>
		<td>12</td>
		<td>13</td>
		<td>14</td>
		<td>15</td>
	</tr>
	<tr>
		<td>base32</td>
		<td>0</td>
		<td>1</td>
		<td>2</td>
		<td>3</td>
		<td>4</td>
		<td>5</td>
		<td>6</td>
		<td>7</td>
		<td>8</td>
		<td>9</td>
		<td>b</td>
		<td>c</td>
		<td>d</td>
		<td>e</td>
		<td>f</td>
		<td>g</td>
	</tr>
	<tr>
		<td>十进制</td>
		<td>16</td>
		<td>17</td>
		<td>18</td>
		<td>19</td>
		<td>20</td>
		<td>21</td>
		<td>22</td>
		<td>23</td>
		<td>24</td>
		<td>25</td>
		<td>26</td>
		<td>27</td>
		<td>28</td>
		<td>29</td>
		<td>30</td>
		<td>31</td>
	</tr>
	<tr>
		<td>base32</td>
		<td>h</td>
		<td>j</td>
		<td>k</td>
		<td>m</td>
		<td>n</td>
		<td>p</td>
		<td>q</td>
		<td>r</td>
		<td>s</td>
		<td>t</td>
		<td>u</td>
		<td>v</td>
		<td>w</td>
		<td>x</td>
		<td>y</td>
		<td>z</td>
	</tr>
</table>

可以得到(39.92324, 116.3906)的编码为wx4g0ec1

## 误差

下表列出了不同的编码长度对应的精度：
<table boder="1px" cellspacing="0px" style="width:100%">
	<tr>
		<td>geohash length</td>
		<td>lat bits</td>
		<td>lng bits</td>
		<td>lat error</td>
		<td>lng error</td>
		<td>km error</td>
	</tr>
	<tr>
		<td>1</td>
		<td>2</td>
		<td>3</td>
		<td>±23</td>
		<td>±23</td>
		<td>±2500</td>
	</tr>
	<tr>
		<td>2</td>
		<td>5</td>
		<td>5</td>
		<td>±2.8</td>
		<td>±5.6</td>
		<td>±630</td>
	</tr>
	<tr>
		<td>3</td>
		<td>7</td>
		<td>8</td>
		<td>±0.70</td>
		<td>±0.7</td>
		<td>±78</td>
	</tr>
	<tr>
		<td>4</td>
		<td>10</td>
		<td>10</td>
		<td>±0.087</td>
		<td>±0.18</td>
		<td>±20</td>
	</tr>
	<tr>
		<td>5</td>
		<td>12</td>
		<td>13</td>
		<td>±0.022</td>
		<td>±0.022</td>
		<td>±2.4</td>
	</tr>
	<tr>
		<td>6</td>
		<td>15</td>
		<td>15</td>
		<td>±0.0027</td>
		<td>±0.0055</td>
		<td>±0.61</td>
	</tr>
	<tr>
		<td>7</td>
		<td>17</td>
		<td>18</td>
		<td>±0.00068</td>
		<td>±0.0068</td>
		<td>±0.076</td>
	</tr>
	<tr>
		<td>8</td>
		<td>20</td>
		<td>20</td>
		<td>±0.000085</td>
		<td>±0.00017</td>
		<td>±0.019</td>
	</tr>
</table>

<table boder="1px" cellspacing="0px" style="width:100%">
	<tr>
		<td>geohash length</td>
		<td>width</td>
		<td>height</td>
	</tr>
	<tr>
		<td>1</td>
		<td>5009.4km</td>
		<td>4992.6km</td>
	</tr>
	<tr>
		<td>2</td>
		<td>1252.3km</td>
		<td>624.1km</td>
	</tr>
	<tr>
		<td>3</td>
		<td>156.5km</td>
		<td>156km</td>
	</tr>
	<tr>
		<td>4</td>
		<td>39.1km</td>
		<td>19.5km</td>
	</tr>
	<tr>
		<td>5</td>
		<td>4.9km</td>
		<td>4.9km</td>
	</tr>
	<tr>
		<td>6</td>
		<td>1.2km</td>
		<td>609.4m</td>
	</tr>
	<tr>
		<td>7</td>
		<td>152.9m</td>
		<td>152.4m</td>
	</tr>
	<tr>
		<td>8</td>
		<td>38.2m</td>
		<td>19m</td>
	</tr>
	<tr>
		<td>9</td>
		<td>4.8m</td>
		<td>4.8m</td>
	</tr>
	<tr>
		<td>10</td>
		<td>1.2m</td>
		<td>59.5m</td>
	</tr>
	<tr>
		<td>11</td>
		<td>14.9cm</td>
		<td>14.9cm</td>
	</tr>
	<tr>
		<td>12</td>
		<td>3.7cm</td>
		<td>1.9cm</td>
	</tr>

</table>

## Java实现代码

```java
import java.util.BitSet;
import java.util.HashMap;  
  
  
public class GeoHash {  
  
    private static int numbits = 6 * 5;  
    final static char[] digits = { '0', '1', '2', '3', '4', '5', '6', '7', '8',  
            '9', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'j', 'k', 'm', 'n', 'p',  
            'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z' };  
      
    final static HashMap<Character, Integer> lookup = new HashMap<Character, Integer>();  
    static {  
        int i = 0;  
        for (char c : digits)  
            lookup.put(c, i++);  
    }  

    public double[] decode(String geohash) {  
        StringBuilder buffer = new StringBuilder();  
        for (char c : geohash.toCharArray()) {  
  
            int i = lookup.get(c) + 32;  
            buffer.append( Integer.toString(i, 2).substring(1) );  
        }  
          
        BitSet lonset = new BitSet();  
        BitSet latset = new BitSet();  
          
        //even bits  
        int j =0;  
        for (int i=0; i< numbits*2;i+=2) {  
            boolean isSet = false;  
            if ( i < buffer.length() )  
              isSet = buffer.charAt(i) == '1';  
            lonset.set(j++, isSet);  
        }  
          
        //odd bits  
        j=0;  
        for (int i=1; i< numbits*2;i+=2) {  
            boolean isSet = false;  
            if ( i < buffer.length() )  
              isSet = buffer.charAt(i) == '1';  
            latset.set(j++, isSet);  
        }  
          
        double lon = decode(lonset, -180, 180);  
        double lat = decode(latset, -90, 90);  
          
        return new double[] {lat, lon};       
    }  
      
    private double decode(BitSet bs, double floor, double ceiling) {  
        double mid = 0;  
        for (int i=0; i<bs.length(); i++) {  
            mid = (floor + ceiling) / 2;  
            if (bs.get(i))  
                floor = mid;  
            else  
                ceiling = mid;  
        }  
        return mid;  
    }  
      
      
    public String encode(double lat, double lon) {  
        BitSet latbits = getBits(lat, -90, 90);  
        BitSet lonbits = getBits(lon, -180, 180);  
        StringBuilder buffer = new StringBuilder();  
        for (int i = 0; i < numbits; i++) {  
            buffer.append( (lonbits.get(i))?'1':'0');  
            buffer.append( (latbits.get(i))?'1':'0');  
        }  
        return base32(Long.parseLong(buffer.toString(), 2));  
    }  
  
    private BitSet getBits(double lat, double floor, double ceiling) {  
        BitSet buffer = new BitSet(numbits);  
        for (int i = 0; i < numbits; i++) {  
            double mid = (floor + ceiling) / 2;  
            if (lat >= mid) {  
                buffer.set(i);  
                floor = mid;  
            } else {  
                ceiling = mid;  
            }  
        }  
        return buffer;  
    }  
  
    public static String base32(long i) {  
        char[] buf = new char[65];  
        int charPos = 64;  
        boolean negative = (i < 0);  
        if (!negative)  
            i = -i;  
        while (i <= -32) {  
            buf[charPos--] = digits[(int) (-(i % 32))];  
            i /= 32;  
        }  
        buf[charPos] = digits[(int) (-i)];  
  
        if (negative)  
            buf[--charPos] = '-';  
        return new String(buf, charPos, (65 - charPos));  
    }  
  
}

```


## 参考内容
* [http://www.cnblogs.com/dengxinglin/archive/2012/12/14/2817761.html](http://www.cnblogs.com/dengxinglin/archive/2012/12/14/2817761.html)
* [http://blog.csdn.net/xiaojimanman/article/details/50358506](http://blog.csdn.net/xiaojimanman/article/details/50358506)