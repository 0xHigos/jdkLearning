#### 写一个程序，让用户来决定windows 任务管理器的cpu占用率程序越简越好，实现下面三种情况：

+ cpu的占用率固定在50%，为一条直线
+ cpu的占用率为一条直线，但是具体占用率由命令行参数决定（参数为1~100）
+ cpu的占用率状态是一个正弦曲线



### cpu的占用率固定在50%，为一条直线(单核)

#### 思路：cpu的使用曲线，是cpu在一段时间内跑busy 和idle两个不同的循环，通过不同的时间比例，来调节cpu的使用率

````java
public class Test1 {
    public static void main( String[] args ) {
        int busy = 5,idle = 5;
        long startTime = 0;
        while(true){
            startTime = System.currentTimeMillis();
            //busy time
            while(System.currentTimeMillis() - startTime <= busy);
            //idle time
            try {
                Thread.sleep(idle);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}
````

![img](https://raw.githubusercontent.com/0xHigos/jdkLearning/master/image/image-20210317215356215.png)

