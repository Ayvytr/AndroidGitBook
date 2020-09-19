# Java面试题



### [Java基础总结大全（实用）](https://www.cnblogs.com/javastu/p/5519569.html)

### [Java基础总结！精华版！](https://blog.csdn.net/Song_JiangTao/article/details/80642188)



1. UTC时间字符串转本地时间（带T和Z的字符串）

   ```java
       public static void main(String args[]) throws ParseException {
   
           UTCToCST("2017-11-27T03:16:03.944Z", "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");
       }
   
       public static void UTCToCST(String UTCStr, String format) throws ParseException {
           Date date = null;
           SimpleDateFormat sdf = new SimpleDateFormat(format);
           date = sdf.parse(UTCStr);
           System.out.println("UTC时间: " + date);
           Calendar calendar = Calendar.getInstance();
           calendar.setTime(date);
           calendar.set(Calendar.HOUR, calendar.get(Calendar.HOUR) + 8);
           //calendar.getTime() 返回的是Date类型，也可以使用calendar.getTimeInMillis()获取时间戳
           System.out.println("北京时间: " + calendar.getTime());
       }
   
   ```

   

