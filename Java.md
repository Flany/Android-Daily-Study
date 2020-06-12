

# 1、泛型

是一种参数化类型机制，编译时类型提前确认机制

## 泛型的好处

- 多种数据类型执行相同的代码；

- 在编码的时候指定数据类型，不需要强制类型转换，以及在插入错误类型的数据时，不至于在运行期抛出异常



## 泛型的原理

泛型时JDK1.5时才有的，虚拟机是不支持泛型的，为了向下兼容，采用泛型擦除来实现，是一种伪泛型机制，在编译后的字节码中是没有泛型信息的，其擦除为原始数据类型Object。



## **泛型擦除机制**

- 获取目标的泛型，获取限定范围；


- 如果泛型是<T>将其擦除为Object原始类型；如果泛型是<T extends Xclass>，会将其擦除为Xclass；如果有多个实现时<T extends Xclass&Yclass>，会将其擦除为第一个类或者接口为原始类；

- 在必要时插入类型转换以保持类型安全；


- 生成桥方法以在扩展时保持多态性。



## **泛型的局限性**

- 不能被实例化；


- 不能用在静态属性和静态方法中；


- 泛型不能用基础数据类型；


- 数组中不能使用泛型。




## **通配符**

- List 						没有泛型，什么类型都可以


- List<T> 				 有泛型，但是也是什么类型都可以，但是在初始化的时候需要指定类型


- List<Object> 	 有泛型，什么类型都可以	   


- List<?>                   有泛型，通配符，什么类型都可以，set/add都不行，可以get，但是get的对象为Object。


- List<? extends T> 限定通配符 只读  有泛型，T和T的子类都可以，set/add都不行，但是get可以，得到的是T和T的父类


- List<? super T>	 限定通配符 只写  只能写入 T和T的子类

```java
public void test() {
    List<? extends Test> list = new ArrayList<>();
    
    list.add(new TestParent());//不行
    list.add(new TestChild());//不行
    list.add(new Test());//不行

    Test test = list.get(0);//可以
    Object object = list.get(0);//可以
    TestParent parent = list.get(0);//可以
    TestChild child = list.get(0);//不行
    
    List<? super Test> list1 = new ArrayList<>();
    list1.add(new TestParent());//不行
    list1.add(new TestChild());//可以
    list1.add(new Test());//可以
    
    //按照下面的实例联想
    TestParent p0 = new Test();
    TestParent p1 = new TestChild();

    Object object1 = list1.get(0);
    Object object2 = new Test();
}
```

# 2、序列化

如网络传输数据，跨进程传输数据时，需要将对象或者数据结构进行序列化成二进制

## 方案

json, xml, protobuf

如何选择：通用性、强壮性、可调式性、可读性、可扩展性、安全性

## **Serializable**

```java
public class Person implements Serializable {
    
    //可以做一些版本控制，可以重写，序列化后，改变了，再反序列化回来就不行
    private static final long serialVersionUID = 1L;
    
    private String name;
    
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

## **Parcelable**

## Externalizable

```java
/**
 * @author Flany
 * @since 2020/6/10
 */
public class Student implements Externalizable {

    private static final long serialVersionUID = 2L;

    private String name;
    private int age;
    private String grade;

    //必须要有一个无参构造函数
    public Student() {
    }

    public Student(String name, int age, String grade) {
        this.name = name;
        this.age = age;
        this.grade = grade;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getGrade() {
        return grade;
    }

    public void setGrade(String grade) {
        this.grade = grade;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject("姓名：" + name);
        out.writeInt(age + 100);
        out.writeObject("年级：" + grade);
    }

    @Override
    public void readExternal(ObjectInput in) throws ClassNotFoundException, IOException {
        name = (String) in.readObject();
        age = in.readInt();
        grade = (String) in.readObject();
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", grade='" + grade + '\'' +
                '}';
    }
}
```

# 3、注解、反射和动态代理

## 注解

- **SOURCE**

  源码级别保留，apt技术（annotation processor tool注解处理器）	例如：@Nonull等做一些IDE语法检查

- **CLASS**		

  保留在class文件中，但是会被虚拟机忽略			字节码插桩

- **RUNTIME**

  保留至运行期，反射

##   APT

在javac编译的时候，会采集所有的注解信息，javac主动调用注解处理器程序来处理，一般用于生成额外的辅助类

```java
创建一个compiler的java工程

创建一个文件    
G:\android-code\CompilerTest\compiler\src\main\resources\META-INF\services\javax.annotation.processing.Processor
    
文件中写入：com.flany.compiler.FlanyProcessor    
/**
 * 指定处理那个注解
 */
@SupportedAnnotationTypes("com.flany.compiler.annotion.Flany")
public class FlanyProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        Messager messager = processingEnv.getMessager();
        messager.printMessage(Diagnostic.Kind.NOTE, "========================测试");
        return false;
    }
}
```

## 静态代理

- 代理模式

  ```java
  public class ProxyDemo {
  
      static interface ProxyInterface {
          void doWork();
      }
  
      static class RealObject implements ProxyInterface {
  
          @Override
          public void doWork() {
              System.out.println("我是真实要做事情的对象");
          }
      }
  
      /**
       * 代理对象也实现了代理的接口
       */
      static class Agent implements ProxyInterface {
  
          //代理对象持有真实对象的实例
          private RealObject realObject;
  
          public Agent(RealObject realObject) {
              this.realObject = realObject;
          }
  
          /**
           * 代理接口的方法的实现实际上是调用真实对象的方法
           */
          @Override
          public void doWork() {
              realObject.doWork();
          }
      }
  }
  ```

- **坏处**

  1. 如果有多个代理接口，就需要多个代理对象；
  2. 需要些很多重复的代码。

## 动态代理

一般的class流程 ->Java源文件--编译-->.class字节码--类加载-->class对象--实例化-->对象   在硬盘中

动态代理  动态生成一个class类的字节码在内存中，然后生成一个Class对象，在这个动态生成的Calss类中，在其中实现了代理接口的方法，实现的方法中的实际实现时调用InvocationHandler的invoke方法。

```java
//动态代理
Proxy.newProxyInstance(getClassLoader(), new Class[]{接口...}, new InvocationHandler() {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return null;
    }
});
```

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Onclick {
    @IdRes int[] value();
}
```

```java
//实现对View.OnClickListener的动态代理
public static void proxy(final Activity activity) {
    Method[] declaredMethods = activity.getClass().getDeclaredMethods();
    for (final Method declaredMethod : declaredMethods) {
        if (declaredMethod.isAnnotationPresent(Onclick.class)) {
            Onclick annotation = declaredMethod.getAnnotation(Onclick.class);
            if (annotation == null) {
                return;
            }
            int[] ids = annotation.value();
            for (int id : ids) {
                View view = activity.findViewById(id);
                View.OnClickListener listener = (View.OnClickListener) Proxy.newProxyInstance(ProxyOnclick.class.getClassLoader(), new Class[]{View.OnClickListener.class}, new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        declaredMethod.setAccessible(true);
                        return declaredMethod.invoke(activity, args);
                    }
                });
                view.setOnClickListener(listener);
            }
        }
    }
}
```



## 注解的注解

```java
//定义一个用在注解上面的注解
@Target(ElementType.ANNOTATION_TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface EventType {
    Class listenerType();
    String listenerSetter();
}
```

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@EventType(listenerType = View.OnClickListener.class, listenerSetter = "setOnClickListener")
public @interface OnClick {
    int[] value();
}
```

```java
public static void injectEvent(Activity activity) {
    Class<? extends Activity> activityClass = activity.getClass();
    Method[] declaredMethods = activityClass.getDeclaredMethods();

    for (Method method : declaredMethods) {
        //获得方法上所有注解
        Annotation[] annotations = method.getAnnotations();

        for (Annotation annotation : annotations) {
            //注解类型
            Class<? extends Annotation> annotationType = annotation.annotationType();
            if (annotationType.isAnnotationPresent(EventType.class)) {
                EventType eventType = annotationType.getAnnotation(EventType.class);
                // OnClickListener.class
                Class listenerType = eventType.listenerType();
                //setOnClickListener
                String listenerSetter = eventType.listenerSetter();

                try {
                    // 不需要关心到底是OnClick 还是 OnLongClick
                    Method valueMethod = annotationType.getDeclaredMethod("value");
                    int[] viewIds = (int[]) valueMethod.invoke(annotation);

                    method.setAccessible(true);
                    ListenerInvocationHandler<Activity> handler = new ListenerInvocationHandler(activity, method);
                    Object listenerProxy = Proxy.newProxyInstance(listenerType.getClassLoader(),
                            new Class[]{listenerType}, handler);
                    // 遍历注解的值
                    for (int viewId : viewIds) {
                        // 获得当前activity的view（赋值）
                        View view = activity.findViewById(viewId);
                        // 获取指定的方法(不需要判断是Click还是LongClick)
                        // 如获得：setOnClickLisnter方法，参数为OnClickListener
                        // 获得 setOnLongClickLisnter，则参数为OnLongClickLisnter
                        Method setter = view.getClass().getMethod(listenerSetter, listenerType);
                        // 执行方法
                        setter.invoke(view, listenerProxy); //执行setOnclickListener里面的回调 onclick方法
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

    }
}
```

# 4、线程和进程

## 概念

1. 进程是程序运行资源分配的最小单位；
2. 线程是CPU调度的最小单位,必须依赖于进程而存在

## 并行和并发

我们举个例子,如果有条高速公路A上面并排有8条车道,那么最大的并行车辆就是8辆此条高速公路A同时并排行走的车辆小于等于8辆的时候,车辆就可以并行运行。CPU也是这个原理,一个CPU相当于一个高速公路A,核心数或者线程数就相当于并排可以通行的车道;而多个CPU就相当于并排有多条高速公路,而每个高速公路并排有多个车道。 

当谈论并发的时候一定要加个单位时间,也就是说单位时间内并发量是多少?离开了单位时间其实是没有意义的

## **线程开启**

1.继承一个Thread，调用start方法；

```java
/**
 * 方式1，这个是一个线程
 */
static class Thread1 extends Thread {
    @Override
    public void run() {
        super.run();
        System.out.println("Thread1 is running");
    }
}
```

2.实现Runnable接口，然后交给Thread运行

```java
/**
 * 方式2，这个不是一个线程
 */
static class Thread2Runnable implements Runnable {

    @Override
    public void run() {
        System.out.println("Thread2 is running");
    }
}
Thread2Runnable thread2Runnable = new Thread2Runnable();
Thread thread2 = new Thread(thread2Runnable);
thread2.start();
```

3.实现Callable接口，AsyncTask的实现就是通过这个方法

```java
/**
 * 方式3，这个不是一个线程，这个并不是，FutureTask是实现了Runnable
 */
static class Thread3Callable implements Callable<String> {

    @Override
    public String call() throws Exception {
        System.out.println("Thread3 is running");
        return "Thread3 success";
    }
}
Thread3Callable thread3Callable = new Thread3Callable();
FutureTask<String> futureTask = new FutureTask<>(thread3Callable);
Thread thread3 = new Thread(futureTask);
thread3.start();
String result = futureTask.get();
```

## **线程的关闭**

- 线程的关闭是一种协作机制，发出信号，线程得知信号然后自己来终止；

- 线程要和谐的停止，不能粗暴停止；


- stop方法，太粗暴了，会造成很多问题，比如资源未释放等等，尽量不要使用；
- interrupt和isInterrupt方法，interrupt发出信号，告知线程可以停止了，线程通过isInterrupt方法得知信息，最后让run方法走完，自动停止；
- 如果run中调用Thread.sleep()，会让取消interrupt发出的信号，并抛出一个异常，在try catch 可以调用Thread.currentThread().interrupt();再次发出中断信号。

## 线程的顺序

使用join方法，让该线程先执行完。

```java
public class JoinTestThread {

    static class JoinThread1 extends Thread {

        private Thread thread;

        public JoinThread1(String name, Thread thread) {
            super(name);
            this.thread = thread;
        }

        @Override
        public void run() {
            super.run();
            System.out.println(getName() + " 开始运行。。。");
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(thread.getName() + " 运行完毕。。。");

            System.out.println(getName() + " 继续运行。。。");
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName() + " is running");
            }
        }
    }

    static class JoinThread2 extends Thread {

        private Thread thread;

        public JoinThread2(String name, Thread thread) {
            super(name);
            this.thread = thread;
        }

        @Override
        public void run() {
            super.run();
            System.out.println(getName() + " 开始运行。。。");
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(thread.getName() + " 运行完毕。。。");

            System.out.println(getName() + " 继续运行。。。");
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName() + " is running");
            }
        }
    }

    static class JoinThread3 extends Thread {

        public JoinThread3(String name) {
            super(name);
        }

        @Override
        public void run() {
            super.run();
            System.out.println(getName() + " 开始运行。。。");
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName() + " is running");
            }
        }
    }

    /**
     * joinThread3 joinThread2 joinThread1顺序执行
     */
    public static void main(String[] args) {
        JoinThread3 joinThread3 = new JoinThread3("==张三==");
        JoinThread2 joinThread2 = new JoinThread2("**李四**", joinThread3);
        JoinThread1 joinThread1 = new JoinThread1("--王五--", joinThread2);
        joinThread1.start();
        joinThread2.start();
        joinThread3.start();
    }
}
```

## 守护线程



## 锁的分类

- 悲观锁：synchronized ，先拿到锁，总有其他线程来改我的数值


- 乐观锁：类似CAS机制，保持好的心态，总以为没有线程来修改我的数值

  

- 阻塞锁：synchronized 

- 非阻塞锁：类似CAS机制，自旋尝试拿锁

  

- 重入锁：synchronized，AQS等实现的重入锁，递归或者多个同步方法相互调用

- 不可重入锁：

  

- 公平锁：先排队，在链表的最后面排队，不停地自旋，看链表的前一个节点是否释放锁

- 非公平锁：先尝试去拿锁，拿不到再去排队

  

- 共享锁：读写锁中的读锁是共享的

- 排它锁：写锁，和synchronized都是排他的

  

- 偏向锁：如果只有一个线程去拿锁或者总是同一个线程拿到锁，不用做CAS操作（第一次需要），直接获取锁

- 自旋锁：防止上下文切换导致的资源浪费，采用自旋去尝试拿锁

- 适应性自旋锁：自旋的次数有限制，自旋的时间不能超过上下文切换的时间

- 轻量级锁：采用CAS操作来获取锁

- 重量级锁：如果加锁的内容执行时间比较久，

  

- **类锁**

  ```java
  static class SingleClass {
  
      private static SingleClass singleClass;
  
      private SingleClass() {
      }
  
      /**
       * 会有一些性能的损耗
       *
       * @return SingleClass
       */
      public synchronized static SingleClass getInstance() {
          if (singleClass == null) {
              singleClass = new SingleClass();
          }
          return singleClass;
      }
  
  
      /**
       * DCL:Double Check Lock 双重检查锁定
       *
       * @return SingleClass
       */
      public static SingleClass getInstance2() {
          if (singleClass == null) {
              synchronized (SingleClass.class) {
                  if (singleClass == null) {
                      singleClass = new SingleClass();
                  }
              }
          }
          return singleClass;
      }
  }
  ```

- **对象锁**

  非静态方法上加synchronized关键字和方法内synchronized(this)

  ```java
  static class ObjectSync {
      
      public synchronized void test() {
      }
      
      public void test2() {
          synchronized (this) {
          }
      }
  }
  ```

- **显示锁**

  ```java
  static class LockObject {
  
      //可重入锁，synchronized也是可重入锁
      private Lock lock = new ReentrantLock();
  
      public void test() {
          lock.lock();
          ...
          lock.unlock();
      }
  }
  ```

## ThreadLocal

原理：Thread内部持有自己的ThreadLocalMap的数组对象，数组对象存放的是该TheadLocal对象和TheadLocal设置的属性值。(Handler内部有用的TheadLocal)

- 自己实现的TheadLocal

```java
//其他线程如果拿到当前线程的实例，是可以获取或者修改当前线程保存的数据
//如果get和set方法不设置synchronized关键字，会导致set的值还没有设置成功，然后get为空
public class MyThreadLocal<T> {

    private Map<Thread, T> myThreadLocalMap = new HashMap<>();

    public void set(Thread thread, T t) {
        myThreadLocalMap.put(thread, t);
    }

    public T get(Thread thread) {
        return myThreadLocalMap.get(thread);   
    }
}
```

- 在线程中设置的属性，只在该线程内有用
- 初始化时的属性，所有的线程可以共享使用

```java
public class ThreadLocalTest {
	//初始化
    static ThreadLocal<String> threadLocal = new ThreadLocal<String>() {
        @Nullable
        @Override
        protected String initialValue() {
            return "初始化参数";
        }
    };
	
    //线程一
    static class ThreadStudent extends Thread {
        @Override
        public void run() {
            super.run();
            System.out.println(Thread.currentThread().getName() + "，第一次获取：" + threadLocal.get());
            threadLocal.set("学生");
            System.out.println(Thread.currentThread().getName() + "，重新赋值之后获取：" + threadLocal.get());
        }
    }

    //线程二
    static class ThreadTeacher extends Thread {
        @Override
        public void run() {
            super.run();
            System.out.println(Thread.currentThread().getName() + "，第一次获取：" + threadLocal.get());
            threadLocal.set("老师");
            System.out.println(Thread.currentThread().getName() + "，重新赋值之后获取：" + threadLocal.get());
        }
    }

    public static void main(String[] args) {
        new ThreadTeacher().start();
        new ThreadStudent().start();

        //主线程
        System.out.println(Thread.currentThread().getName() + "，第一次获取：" + threadLocal.get());
        threadLocal.set("学校");
        System.out.println(Thread.currentThread().getName() + "，重新赋值之后获取：" + threadLocal.get());

        //主线程 子线程 设置属性 相互不影响，只有初始化的时候的属性共享
    }
}
```

## 线程的状态/线程的生命周期

- 初始化（new）：新建了一个线程，但是还没有调用start方法；

- 运行（RUNNABLE）:java将运行状态分为就绪（reday）和运行中（running）；

  新建一个线程后，调用start方法后，该线程就进入可运行线程池中，等待被线程调度选中，获取CPU的执行权，此时线程就处于就绪状态。就绪状态的线程在获取CPU时间片后变为运行中状态。

- 阻塞（blocked）：线程阻塞于锁，线程想要获取锁，但是没有获取到，就处于阻塞状态，等待其他线程的唤醒；

- 等待（waiting）：进入该状态的线程需要等待其他线程做出一些特定的动作（通知或者中断），才能回到运行状态；

- 等待超时（timed_waiting）：该状态不同于等待状态（waiting），它可以在指定的时间后自行返回运行状态。

- 终止（terminated）:表示该线程已经执行完毕。

  

​				等待(wait, join)  唤醒（notify, notifyAll）



初始(new)  -->运行（start，CPU调度）-->就绪(yield()方法让出CPU执行权，内部或外部因素)----------终止



​				等待超时（Thread.sleep(long time), wait(long time)，join(long time)）	唤醒（notify, notifyAll）

​					

​						阻塞（有且仅有synchronized可实现）Lock没有获取到锁的时候，是进入到等待或者等待超时状态

​						阻塞会造成上下文切换，3-5ms  一个CPU指令0.6ns



阻塞是被动进入；

等待是主动进入，我没有获取到资源，主动进入等待状态。

## 死锁

两个或者两个以上的线程在执行过程中，由于竞争资源或者彼此之间相互通信，而造成的阻塞现象，若无外力作用，他们讲无法进行下去，此时称系统处于死锁状态或者系统产生了死锁。

**产生死锁的条件：**

- 多个线程M>=2   争夺N>=2个资源 M>=N
- 争夺资源的顺序不对
- 未拿到全部资源，之前的资源不释放

**专业术语：**

互斥条件：资源拿到后，只能独享

请求保持：在请求其他资源的时候，没拿到，一直在请求，也不释放之前拿到的资源  

不剥夺：资源不能被剥夺 

环路等待：1.拿到A  想拿B;	2.拿到B，想拿A

**解决方案：**

关键是保证拿锁的顺序一致

1、内部通过比较，确定拿锁的顺序；

2、采用尝试拿锁的机制；

## 其他多线程问题

- 活锁：两个线程在尝试拿锁的机制中，发生多个线程之间的谦让，不断发生同一个线程拿到同一把锁，在尝试那另外一把锁而拿不到，从而释放之前已经拿到的锁的过程。
- 线程饥饿：优先级比较低的线程，总是拿不到锁。也不是说优先级比较低，有可能就有一个线程一直拿不到锁。

## 可见性和原子性

- 可见性--当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他的线程能立刻看到修改的值；
- 原子性--执行一个操作或多个操作的时候，要么全部都不执行，要么全部都执行，并且在执行的过程中不会被打断。

## CAS

compare and swap  比较和交换   自旋   先计算 和预期的比较 如果和预期一样  就交换，如果不一样，就重新再来一次

ABA问题 ：加入版本控制

开销问题： 如果实在是太大了，采用加锁方法

只能保证一个共享变量的原子操作： 将多个共享变量组合成一个对象来使用

## AQS

```java
/**
 * AQS，采用了模板设计模式
 * 类似ReentrantLock，但是不可重入锁
 *
 * @author Flany
 * @since 2020/6/1
 */
public class AQSReentrantLock implements Lock {

    private class AQSSync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire(int arg) {
            //原子操作，设置同步的状态
            if (compareAndSetState(0, 1)) {
                //设置当前线程为拿到锁的线程
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            if (getState() == 0) {
                throw new RuntimeException("");
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
    }

    private AQSSync aqsSync = new AQSSync();

    @Override
    public void lock() {
        System.out.println(Thread.currentThread().getName() + " --> 尝试拿锁");
        //内部调用tryAcquire方法
        aqsSync.acquire(1);
        System.out.println(Thread.currentThread().getName() + " --> 已经拿到锁了");
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {

    }

    @Override
    public boolean tryLock() {
        return false;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return false;
    }

    @Override
    public void unlock() {
        System.out.println(Thread.currentThread().getName() + " --> 尝试释放锁");
        //内部调用tryRelease方法
        aqsSync.release(1);
        System.out.println(Thread.currentThread().getName() + " --> 已经释放锁了");
    }

    @Override
    public Condition newCondition() {
        return null;
    }
}
```

## CLH

- 公平锁  链表 QNode(指向前一个节点，当前是否获取到锁的状态，Thread)   按顺序排队   自旋是有次数限制的，超过一定的次数后，将其放到阻塞队列中

## Volatile

**原理：**

有volatile变量修饰的共享变量进行写操作的时候会使用CPU提供的Lock前缀指令。

1、将当前处理器缓存行的数据写到系统内存中；

2、这个写操作会使其他CPU里缓存了改内存地址的数据无效，所以在使用的时候，会重新去获取数据。

- 保证可见性，抑制重排序（CPU指令，可能会重排序，在多线程中会出现问题，加上volatile关键字，可以预防这个问题）

- 读和写是可以保证原子性的（get/set），但是做一些复合操作（count++），就不能保证原子性了

- 使用场景：一个线程写，多个线程读

## synchronized和Lock原理

- synchronized：在jvm虚拟机中是通过monitorenter和monitorexit两个指令，来获取或者释放monitor对象的锁，synchronized不管是加static还是不加，都是通过锁对象，只是static锁的对象是类的class对象，不加static是锁的类的对象的实例。
- Lock：