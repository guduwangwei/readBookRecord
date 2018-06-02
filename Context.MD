#### Context

 ![](https://img-blog.csdn.net/20151022212109519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* `Application`对应的`Context` 

  ​

  app启动 是通过 fork "zygote"进程来实现的 ， `SystemServer fork zygote` 

  SystemServer

  ````
   private void createSystemContext() {
          ActivityThread activityThread = ActivityThread.systemMain();//->1
          mSystemContext = activityThread.getSystemContext();
          mSystemContext.setTheme(DEFAULT_SYSTEM_THEME);

          final Context systemUiContext = activityThread.getSystemUiContext();
          systemUiContext.setTheme(DEFAULT_SYSTEM_THEME);
      }
  ````

  ->1 

  ​

  ​

  `ContextWrapper`是一个包装类 

  ```java
  //ContextWrapper所有的方法都是通过调用 mBase对应的方法
  public class ContextWrapper extends Context {
      Context mBase;

      public ContextWrapper(Context base) {
          mBase = base;
      }
  }
  ```

  ​