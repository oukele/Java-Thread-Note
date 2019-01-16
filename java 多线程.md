~~~ java
进程
指的是：程序（任务）的执行过程。持有资源（共享内存，共享文件）和线程。

线程
线程是系统中最小的执行单元，同一进程中有多个线程。线程共享进程的资源。

线程的交互
互斥、同步
互斥 --> 考试的资料
同步 --> 大合唱
~~~

Thread常用方法

~~~ java
 线程的创建
 Thread()
 Thread(String name)
 Thread(Runnable target)
 Thread(Runnable taget,String name)
 
 线程的方法
 void start();//启动线程
 static void sleep(long millis)//线程休眠
 static void sleep(long millis , int nanos)//线程休眠
 void join()//使其他线程等待当前线程终止
 void join(long millis)//使其他线程等待当前线程终止
 void join(long millis ,int nanos)//使其他线程等待当前线程终止
 static void yield()//当前运行线程释放处理器资源
 
 获取线程引用
 static Thread currentThread()//返回当前运行的线程引用
~~~

代码模拟：

~~~ java
//军队线程
//模拟作战双方的行为
public class ArmyRunnable implements Runnable{
    
}
//军队
ArmyRunnable armyTaskOfSuiDynasty = new ArmyRunnable();
//农民军队
ArmyRunnable armyTaskOfrmyOfRevolt = new ArmyRunnable();

//使用Runnable 接口 创建线程
Thread armyOfSuiDynasty = new Thread(armyTaskOfSuiDynasty,"隋军");
Thread armyOfRevolt = new Thread(armyTaskOfrmyOfRevolt,"农民起义军");


//关键人物线程
public class KeyPersonThread extends Thread{
    public KeyPersonThread(String name){
        super(name);
    }
    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName + "开始战斗了！");
        for(int i=0 ; i < 10 ; i++){
            System.out.println(Thread.currentThread().getName() + "左突右杀，攻击隋军....");
        }   
        System.out.println(Thread.currentThread().getName() + "结束了战斗");
    }
}

// 战斗舞台
public class Stage extends Thread{
    
}
//下面的代码可以运行
~~~

~~~ java
public class Actor extends Thread{
    public void run(){
        System.out.println(getName() + "是一个演员");
        int count = 0;
        boolean flag = true;
        while(flag){
        	System.out.println(getName() + "登台演出：" + (++count));
            if ( count == 100){
                flag = false;
            }
            
            if( count % 10 ==0 ){
                try{
                	Thread.sleep(1000);
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
            
        }
        System.out.println(getName() + "的演出结束了");
    }
    
    public static void main(String[] agrs){
        Thread actor = new Actor();
        actor.setName("Mr.Thread");//设置线程名称
        actor.start();//开始线程
        
        Thread actressThread = new Thread(new Actress(),"Ms Runnable");
        actressThread.start();//开始线程
        
    }
}
// Runnable 接口之中 没有getName方法 可以使用Thread的静态方法
//currentThreat().getName(）获取该对象的名字。
class Actress implements Runnable{
    @Override
    public void run(){
        System.out.println(Thread.currentThread.getName() + "是一个演员");
        int count = 0;
        boolean flag = true;
        while(flag){
        	System.out.println(Thread.currentThread.getName() + "登台演出：" + (++count));
            if ( count == 100){
                flag = false;
            }
            
            if( count % 10 ==0 ){
                try{
                	Thread.sleep(1000);
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
            
        }
        System.out.println(Thread.currentThread.getName() + "的演出结束了");
    }
}
//下面的代码可运行
~~~

#### 大作战

~~~ java
//军队线程
//模拟作战双方的行为
public class ArmyRunnable implements Runnable{
    
    //volatitle 保证了线程可以正确的读取其他线程写入的值
    // 可见性 
    // 这个值 改变时 JMM 会把该线程对应的本地内存中的变量强制刷新到主内存中去。
    // 比如：线程A 将其 修改为 false 时，线程B可以立刻得知。
    volatile boolean keepRunnable = true;
    
    @Override
    public void run(){
        while(keepRunnable){
            //发动 5连击
            for( int i =0 ; i < 5; i++){
                System.out.println(Thread.currentThread().getName() + "进攻对方["+ i +"]" );
                //让出了处理器时间，下次该谁进攻未知
                Thread.yield();
            }
        }
        System.out.println(Thread.currentThread().getName() +"结束了战斗");
    }
}
~~~
####  战场
~~~ java
public class Stage extends Thread{
    
    public void run(){
        //创建对象
        ArmyRunnable armyTaskOfSuiDynasty = new ArmyRunnable();
        ArmyRunnable armyTaskOfRevolt = new ArmyRunnable();
        //创建线程
        Thread armyOfSuiDynasty = new Thread(armyTaskOfSuiDynasty,"隋军");
        Thread armyOfRevolt = new Thread(armyTaskOfSuiDynasty,"农民军");
        //开始线程,让军队开始作战
        armyOfSuiDynasty.start();
        armyOfRevolt.start();
        //战场线程休眠，大家专心观看军队厮杀
        try{
        	Thread.sleep(50);
        }catch(InterruptedExecption e){
            e.printStackTrace();
        }
        
        armyTaskOfSuiDynasty.keepRunning = false;
        armyTaskOfRevolt.keepRunning = false;
        try{
            //使同级其他线程停下来，等待调用join方法的对象完成线程后，再进行其他线程
        	armyOfRevolt.join();
        }catch(InterruptedExecption e){
            e.printStackTrace();
        }
        
    }
    
    public static void main(String[] agrs){
        new Stage().start();//战场开始（线程）
    }
    
}
~~~

#### 关键人物

~~~ java
public class KeyPersonThread extends Thread{
    public void run(){
        System.out.println(Thread.currentThread().getName() +"开始了战斗");
        for( int i = 0 ; i < 10; i++){
            System.out.println(Thread.currentThread().getName() +"左突右杀，攻击隋军");
        }
        System.out.println(Thread.currentThread().getName() +"结束了战斗");
    }
}
~~~

#### 战场舞台

~~~ java
public class Stage extends Thread{
    
    public void run(){
        System.out.println("一场战争准备开始。。。。");
        //休眠5秒 开始大战
         try{
        	Thread.sleep(5000);
        }catch(InterruptedExecption e){
            e.printStackTrace();
        }
        System.out.println("战争开始了。。。。");
         try{
        	Thread.sleep(2000);
        }catch(InterruptedExecption e){
            e.printStackTrace();
        }
        
        //创建对象
        ArmyRunnable armyTaskOfSuiDynasty = new ArmyRunnable();
        ArmyRunnable armyTaskOfRevolt = new ArmyRunnable();
        //创建线程
        Thread armyOfSuiDynasty = new Thread(armyTaskOfSuiDynasty,"隋军");
        Thread armyOfRevolt = new Thread(armyTaskOfSuiDynasty,"农民军");
        //开始线程,让军队开始作战
        armyOfSuiDynasty.start();
        armyOfRevolt.start();
        //战场线程休眠，大家专心观看军队厮杀
        try{
        	Thread.sleep(50);
        }catch(InterruptedExecption e){
            e.printStackTrace();
        }
        
        System.out.println("正当双方激战的时候，半路杀出了个程咬金");
        Thread mrCheng = new KeyPersonThread();
        mrCheng.setName("程咬金");
        System.out.println("程咬金的理想就是结束战争，使百姓安居乐业");
        
        //停止军队作战
        //停止线程的方法
        armyTaskOfSuiDynasty.keepRunning = false;
        armyTaskOfRevolt.keepRunning = false;
        
        try{
        	Thread.sleep(2000);
        }catch(InterruptedExecption e){
            e.printStackTrace();
        }
        
        //关键人物出场
        mrCheng.start();
        // 所有线程等待程咬金完成历史使命
        try{
        	mrCheng.join();
        }catch(InterruptedExecption e){
            e.printStackTrace();
        }
        System.out.println("战争结束，人民安居乐业，程咬金实现了他的理想");
        System.out.println("战争结束了。。。。");
    }
    
    public static void main(String[] agrs){
        new Stage().start();//战场开始（线程）
    }
    
}
~~~

##### 如何正确的停止Java中的线程

~~~ java
// 不可取
stop() --> 戛然而止 ( 线程信息丢失 片段丢失 )

// 使用退出标志
示例中的 keepRunnign 变量

// 中断线程
interrupt()方法 初衷并不是用于停止线程。


~~~

