# Java多线程复习笔记md版




# Java 多线程笔记

> 大连交通大学 信息学院 刘嘉宁
>
> 笔记摘自 西部开源 秦疆



## 多线程基础

#### 进程与线程

- 进程（Process）：进程就是程序的一次执行过程，是系统分配的单位，操作系统在执行时里面有很多进程
- 线程（Thread）：一个进程中可以有一至多个线程，是 CPU 调度和执行的单位，真正的线程是多个核心每个核心处理一个线程

#### Thread 类常用方法

- `Thread.currentThread()` 获取当前线程
- `Thread.currentThread().getName()` 获取当前线程 name
- `new Thread(t1, "张三")` 创建线程对象时，为线程设置 name 
- `Thread.sleep(200);` 设置阻塞，单位 ms 

#### 线程的五大步骤

1. 新建
2. 就绪
3. 运行
4. 阻塞
5. 终止

<img src="Java多线程复习笔记md版.assets/image-20220325205456137.png" alt="image-20220325205456137" style="zoom: 33%;" />

#### 线程的相关方法

- `final void setPriority(int newPriority)` 更改线程的优先级
- `static void sleep(long millis)` 【休眠】
- `void join()` 等待该线程终止，【插队】
- `static void yield()` 暂停当前正在执行的线程对象，【礼让】
- `void interrupt()` 中断线程（勿用）
- `boolean isAlive()` 测试线程是否处于活动状态
- `Thread.state getState()` 获取线程的状态（NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED）

#### 线程的停止

- `stop()`、`destory()` 已经被标识过时不建议使用
- 让线程自己停下来【推荐】
- 设置一个标记位，当其值为 flase 时，终止其循环

```java
class MyThread implements Runnable{

    boolean flag = true;

    @Override
    public void run() {
        int i = 1;
        while (flag){
            System.out.println(i++);
        }
    }

    public void stop(){
        this.flag = false;
    }
}
```

#### 线程的阻塞

1. `Thread.sleep(1000ms)` 【静态方法】
    - 每个线程都有一把锁，sleep 不会释放锁
2. `wait()` 
3. 同步锁定时

#### 线程的礼让

- `Thread.yield()`【静态方法】
- 让当前线程从运行状态变为就绪状态，暂停一下
- cpu 会重新调度线程，不一定会礼让成功

#### 线程的插队

- `t.join()` 
- 让 t 线程强制执行，其它线程阻塞，执行结束后其他线程才可继续

#### 线程的优先级

- `getPriority().setPriority(5)` 
- 线程的优先级由 1 ~ 10 表示，优先级越高权重就越大，但调度器未必完全按优先级来执行

#### 守护线程 daemon 线程

- 线程分为**用户线程**和**守护线程**
- JVM 不会等待守护线程执行完毕
- `setDaemon(true)` 设置一个线程为守护线程



## 线程的实现

- run 方法：线程执行的内容
- start 方法：开启线程的方法，线程开启不一定立即执行，由 CPU 调度执行

#### 继承 Thead 类

> OOP 单继承局限性

- 创建一个类，继承 Thread 类 `class 线程对象 extends Thread`
- 重写 run 方法
- 调用其 start 方法开启线程

#### 实现 Runnable 接口

> 避免单继承的局限性，灵活方便，方便同一个对象被多个线程使用

- 创建一个类，实现 Runnable 接口 `class 线程对象 implements Runnable`
- 重写 run 方法
- 创建 Thread **代理类**实例 `Thread 代理对象 = new Thread(线程对象)`
- 调用代理类 srart 方法开启线程

#### 实现 Callable 接口

> 可以定义返回值，可以抛出异常，实现较为复杂

- 创建一个类，实现 Callable 接口 `class 线程对象 implements Callable<返回值类型>`

- 重写 call 方法（需要抛出异常）`public Boolean call() throws Exception`

- 创建线程对象

- 创建执行服务

    ```java
    //													线程池数量（也就是想要创建几个线程）
    ExecutorService service = Executors.newFixedThreadPool(1);
    ```

- 提交执行

    ```java
    //想要创建几个线程就写多少行
    Future<Boolean> r = service.submit(线程对象);
    ```

- 获取结果

    ```java
    //创建了几个线程就写多少行
    返回值类型 res = r.get();
    ```

- 关闭服务 `service.shutdown();`

#### 小问题

- 使用 Runnable 实现多线程的时候，为什么说 Thread 是代理类？
    - 因为此时的 Thread 充当的就是**静态代理**中的代理类
    - 在 `new Thread(线程对象)` 后，Thread 类会将线程对象作为其成员变量 `private Runnable target;`
    - 在启动线程后，调用 Thread 类的 run 方法实际上调用的是 `target.run();` 并对其进行**访问控制**、**功能增强** 



## 线程同步

- 线程同步就是一种等待机制，多个需要同时访问此对象的线程进入**线程的等待池**形成**队列**，等待前一个线程执行完毕，后面的才能执行

- 锁机制，为当前线程设置排它锁，独占资源

    - synchronized 

    - lock 

        - 使用 Lock 对象充当同步锁，Lock 接口的实现类 ReentrantLock 【可重入锁】可以显式的加锁、释放锁
        - 是显式锁，需要手动关闭锁
        - 只有代码块锁，没有方法锁
        - 使用 Lock 锁减少 JVM 调度线程的时间，性能更好

    - 优先使用：Lock  > 同步代码块 > 同步方法

        ```java
        class MyThread1 implements Runnable {
        
            private ReentrantLock lock = new ReentrantLock();
        
            @Override
            public void run() {
                try {
                    lock.lock();
                    ...
                //}catch (Exception e){
                }finally {
                    lock.unlock();
                }
            }
        }
        ```

- 锁机制会存在的问题：

    - 当一个线程持有锁，其他线程都要挂起等待
    - 多线程竞争时，加锁、释放锁，会导致较多的**上下文切换**、**调度延时**，引起性能问题
    - 如果一个优先级高的线程在等待优先级低的线程释放锁，导致**性能倒置** 

1. 为方法加锁
    - 锁的是**同步监视器** 也就是 this

```java
class Blank{
    int money;
    
    synchronized void buy(){
		...
    }
}
```

2. 添加锁方法块
    - 锁的**同步监视器**是括号中的对象
        - this：锁 this 就是给当前这个对象整个锁起来了，相等于给银行锁起来，但是人们还在里面取钱
        - 变量：锁 变化的量 ，锁的是进行业务操作的对象，也就是账户这个类而不是银行这个类

```java
class Account{
    ...
}

class Blank{
    Account account;
    
    //如果此时将fun方法上锁，就是错误的锁
    fun(){
        sybchronized (account) {
    		...
		}	
    }
}
```



## 死锁

- 多个线程都在等待对方释放资源，都停止执行的情形
    - 互斥条件：一个资源每次只能被一个进程使用
    - 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放
    - 不剥夺条件：进程已获得的资源，在未使用完之前，不能强行剥夺
    - 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系

```java
class DLTest extends Thread {
    static Integer a = new Integer(0);
    static Integer b = new Integer(1);

    int flag;

    public DLTest(int flag) {
        this.flag = flag;
    }

    @Override
    public void run() {
        makeup();
    }

    private void makeup() {
        if (flag == 0){
            synchronized (a){
                System.out.println(Thread.currentThread().getName()+" 拿到了a");
                synchronized (b){
                    System.out.println(Thread.currentThread().getName()+" 拿到了b");
                }
            }
        }else {
            synchronized (b){
                System.out.println(Thread.currentThread().getName()+" 拿到了b");
                synchronized (a){
                    System.out.println(Thread.currentThread().getName()+" 拿到了a");
                }
            }
        }
    }
    
    public static void main(String[] args) {
        DLTest dl1 = new DLTest(0);
        DLTest dl2 = new DLTest(1);

        dl1.start();
        dl2.start();
    }
}
```

> 使用命令行查看是否死锁：
>
> ```
> jps //查看当前用户下的java进程的pid及基本信息
> jstack 进程号 //查看指定进程（pid）的堆栈信息，用以分析线程情况
> ```



## 生产者和消费者问题

- 生产者和消费者问题

    - 假设仓库中只能存放一件产品, 生产者将生产出来的产品放入仓库, 消费者将仓库中产品取走消费.
    - 如果仓库中没有产品, 则生产者将产品放入仓库, 否则停止生产并等待, 直到仓库中的产品被消费者取走为止.
    - 如果仓库中放有产品, 则消费者可以将产品取走消费, 否则停止消费并等待, 直到仓库中再次放入产品为止.

- 只能在同步代码块中使用

    - `wait()` 让线程处于等待状态，直到其他线程通知【会释放锁】

    - `notify()` 唤醒一个处于等待状态的线程

    - `notityAll()` 唤醒一个对象上所有调用 wait 的线程



## 线程池

- 经常创建、销毁线程对性能影响比较大
- 使用多线程可以
    - 提高相应速度
    - 降低资源消耗
    - 便于线程管理
- `ExecutorService` 线程池接口
- `Executors` 工具类、工厂类，用于创建并返回不同类型的线程池

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolTest {
    public static void main(String[] args) {
        //创建一个线程池，池大小为 10
        ExecutorService service = Executors.newFixedThreadPool(10);
        //执行
        service.execute(new MyThread2());
        service.execute(new MyThread2());
        service.execute(new MyThread2());
        service.execute(new MyThread2());
        service.execute(new MyThread2());
		//关闭线程池
        service.shutdown();
    }
}

class MyThread2 implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
    
}
```

- Callable 就是使用的线程池创建的线程












