##### Timer 和 TimerTask

Timer 是 Jdk 中 提供的一个 定时器工具，使用的时候会在主线程之外起一个单独的线程执行指定的计划任务，可以指定执行一次或者 反复执行多次。

TimerTask 是一个实现了Runnable接口的抽象类，代表一个 可以被Timer 执行的任务



~~~ java
Timer的调用

1、Timer.schedule(TimerTask task,Date time)//安排在制定的时间执行指定的任务
2、Timer.schedule(TimerTask task,Date firstTime,long period)//安排指定的任务在指定的时间开始进行重复的固定延迟执行。
3、Timer.schedule(TimerTask task,long long delay)//安排在指定延迟后执行指定的任务。
4、Timer.schedule(TimerTask task,long delay,long period)//安排指定的任务从指定的延迟后开始进行重复的固定延迟执行。
5、Timer.scheduleAtFixedRate(TimerTask task,Date firstTime,long period)//安排指定的任务在指定的时间开始进行重复的固定速率执行
6、Timer.scheduleAtFixedRate(TimerTask task,long delay,long period)//安排指定的任务在指定的延迟后开始进行重复的固定速率执行
~~~

示例

~~~ java
//定时任务
class TimerTask1 extends TimerTask{
    private String taskName = "";
    public TimerTask1(String taskName){
        this.taskName = taskName;
    }
    @Override
    public void run(){
         System.out.println(taskName+new Date()+"");
    }
}
~~~

~~~ java
//创建定时器
Timer timer = new Timer("定时器1",false)；
//给定时器安排任务，延迟10毫秒执行，执行完后间隔3000毫秒执行
timer.schedule(new TimerTask1("定时器A"),10,3000);
timer.schedule(new TimerTask1("定时器B"),10,1000);

Scanner input=new Scanner(System.in);
System.out.print("是否要结束任务：");
input.nextLine();
timer1.cancel();  //结束任务
System.out.println("定时任务结束了");

~~~

### 实现Runnable 接口 比继承 Thread 类所具有的优势

1. 适合多个相同的程序代码的线程去处理同一个资源
2. 可以避免java中单继承的限制
3. 增加程序的健壮性，代码可以被多个线程共享，代码和数据独立
4. 线程池只能放入实现Runnable或callable类线程，不能直接放入继承Thread的类

线程的状态：

![](https://images2018.cnblogs.com/blog/63651/201807/63651-20180723105056988-1034562406.png)

案例：

1. 每隔5秒向 某个目录下 写入一个文件（yyyyMMddHHmmss.txt），内容为一个GUID，总文件数超不过100个，保留最新的文件。
2. 数据库自动备份功能，每5个小时备份一次，可以删除备份文件。
3. 实现1+2+3+.....1000000，使用多线程分段累加后合并结果，并测试使用单线程与多线程所耗时的差别。

1、

~~~ java
package com.oukele.thread;

import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.sql.SQLOutput;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

public class OneTask {

    public static void main(String[] args) {
        //创建File 对象
        File file = new File("F:\\test");
        //如果文件夹不存在 ，则创建文件
        if (!file.exists()) {
            file.mkdir();
        }
        Thread thread = new Thread(new WriteContent(file), "线程A");
        thread.start();

    }
}

class WriteContent implements Runnable {
    private File fileDis;

    public WriteContent(File dis) {
        this.fileDis = dis;
    }

    public void run() {
        int i = 1;
        //保持 最新记录
        File[] files = fileDis.listFiles();
        if (files.length > 0) {
            for (File file : files) {
                file.delete();
                System.out.println("删除[" + i + "]个文件");
                i++;
            }
        }

        i = 1;
        System.out.println("线程开始.....");
        while (true) {
            //写入文件
            try {
                //yyyyMMddHHmmss的文件夹格式
                SimpleDateFormat dateFormat = new SimpleDateFormat("yyyyMMddHHmmss");
                //创建文件
                FileWriter writer = new FileWriter(fileDis + File.separator + dateFormat.format(new Date()) + ".txt");
                //写入GUID 内容
                writer.write(String.valueOf(UUID.randomUUID()));
                //清空缓存
                writer.flush();
                //关闭输出流
                writer.close();
                System.out.println("成功写入 [" + i + "] 个文件");
                //每隔5秒休息一下
                Thread.sleep(5000);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            i++;
            //文件数量不能超过 100个
            if (i > 100) {
                System.out.println("线程结束....");
                break;
            }
        }
    }
}
~~~

2、

~~~ java
package com.oukele.thread;


import java.io.File;
import java.io.IOException;
import java.util.Timer;
import java.util.TimerTask;

public class TwoTask {

    public static void main(String[] args) throws Exception {

        //创建定时器
        Timer timer = new Timer("",false);
        //给定时器安排任务，延迟10毫秒执行，执行完后间隔5小时执行
        // 1s =1000ms 、 1分 = 60s 、1小时 = 60分
        // 5小时 =  1000 * 60 * 60 * 5
        timer.schedule(new DataBaseDumpTask("test"),10,1000 * 60 * 60 *5 );
        //结束定时器
        //timer.cancel();
    }

}
//创建定时任务
class DataBaseDumpTask extends TimerTask {
    private String sqlname;

    DataBaseDumpTask(String sqlname){
        this.sqlname = sqlname;
    }

    public void run() {
        try {
            dataBaseDump("127.0.0.1","oukele","oukele","shiro",sqlname);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //mysqldump -h端口号 -u用户 -p密码 数据库 > d:/test.sql --备份D盘
    //备份
    public static void dataBaseDump(String port,String username,String password,String databasename,String sqlname) throws Exception {
        File file = new File("F:\\test");
        if ( !file.exists() ){
            file.mkdir();
        }
        File datafile = new File(file+File.separator+sqlname+".sql");
        if( datafile.exists() ){
            System.out.println(sqlname+"文件名已存在，请更换");
            return ;
        }
        //拼接cmd命令
        Process exec = Runtime.getRuntime().exec("cmd /c mysqldump -h"+port+" -u "+username+" -p"+password+" "+databasename+" > "+datafile);
        if( exec.waitFor() == 0){
            System.out.println("数据库备份成功,备份路径为："+datafile);
        }
    }

    //还原
    //backup("127.0.0.1","oukele","oukele","shiro","test1");
    //mysql -h端口号 -u用户 -p密码 数据库 < d:/test.sql 恢复到数据库中
    public static void backup(String port,String username,String password,String databasename,String sqlname) throws Exception {
        File datafile = new File("F:\\test\\"+sqlname+".sql");
        if( !datafile.exists() ){
            System.out.println(sqlname+"文件不已存在，请检查");
            return ;
        }
        //拼接cmd命令
        Process exec = Runtime.getRuntime().exec("cmd /c mysql -h"+port+" -u "+username+" -p"+password+" "+databasename+" < "+datafile);
        if( exec.waitFor() == 0){
            System.out.println("数据库还原成功，还原的文件为："+datafile);
        }
    }

}

~~~

3、

~~~ java
单线程
	int n = 0;
    long start = System.currentTimeMillis();
    while (true) {
       if (n >= 1000000) {
              break;
        }
        System.out.println(Thread.currentThread().getName() + ":count:" + n);
         ++n;
    }
    System.out.println("结果：" + n);
    System.out.println("耗时：" + (System.currentTimeMillis() - start) / 1000 + " s");

	结果：1000000
	耗时:15s
~~~

~~~ java
多线程
package com.oukele.thread;

public class ThreeTask {
    int count = 0;

    public static void main(String[] args) {
        ThreeTask threeTask = new ThreeTask();

        MyRunnable myRunnable = new MyRunnable(threeTask);

        for (int i = 0; i < 5; i++) {
            new Thread(myRunnable).start();
        }
    }
}

class MyRunnable implements Runnable {

    private ThreeTask threeTask;

    public MyRunnable(ThreeTask threeTask) {
        this.threeTask = threeTask;
    }

    public void run() {
        long start = System.currentTimeMillis();
        while (true) {
            //锁住的是同一对象
            synchronized (threeTask) {
                if (threeTask.count >= 1000000) {
                    break;
                }
                ++threeTask.count;
                //当前运行线程释放处理器资源
                //Thread.yield();
                System.out.println(Thread.currentThread().getName() + ":count:" + (threeTask.count));
            }
        }
        System.out.println("耗时：" + (System.currentTimeMillis() - start) / 1000 + " s");
        System.out.println("结果：" + threeTask.count);
    }
}

结果：1000000
耗时：17s
~~~





