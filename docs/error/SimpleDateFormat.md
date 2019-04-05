<h1 align="center">jdk8中SimpleDateFormat对错误日期的校验通过问题</h1>
###问题
之前写过一个扫描文件的功能，需要识别里边的日期，其中有一行的日期是非法的，2010-3-58，按理说应该会抛错，然而并没有...

还原代码
```java
public class Main {
    public static void main(String[] args) throws ParseException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        Date date = sdf.parse("2010-03-58");
        System.out.println(date);
    }
}
```
输出结果 2019-04-27，莫名其妙的日期。

###原因
开发环境使用的jdk8，于是追踪parse(String)方法，发现其内部调用的Calendar方法。

代码还原
```java
public class Main {
    public static void main(String[] args) throws ParseException {
        Calendar cal = Calendar.getInstance();
        cal.set(Calendar.YEAR, 2010);
        cal.set(Calendar.MONTH, 2);
        cal.set(Calendar.DATE, 58);
        System.out.println(cal.get(Calendar.YEAR));
        System.out.println(cal.get(Calendar.MONTH));
        System.out.println(cal.get(Calendar.DATE));
    }
}
```
发现Calendar会把日期进位，获得2010-04-27(Calendar的月是从0开始的)，其结果并不与问题结果相同。

原来SimpleDateFormat对年月日分别使用了不同的Calendar，造成只有日期进位，而年月正常的情况。

###解决方案
jdk8引入了LocalDate等线程安全的时间类，也引入了DateTimeFormatter来格式化时间

使用该方式能够校验其规则，错误的日期会直接报错
```java
public class Main {
    public static void main(String[] args) throws ParseException {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        LocalDate date = LocalDate.parse("2010-03-58", formatter);
        System.out.println(formatter.format(date));
    }
}
```
###题外话
Calendar的set方法注入值的时候并不会校验其值的对错，通过debug可以发现，set值之后会直接赋值给对象，只有调用get方法的时候才会进行校验，并对错误的数据进行进位处理，我觉得这样的处理方案是很糟糕的，SimpleDateFormat使用多个Calendar的处理方案就更加糟糕了。

SimpleDateFormat由于其线程不安全性与不稳定性，应尽量避免使用。