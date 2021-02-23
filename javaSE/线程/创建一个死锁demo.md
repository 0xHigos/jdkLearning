### 通过两个资源和两个线程，互相访问对方的资源造成死锁现象

````java
public class DeadLockDemo {
    private static final String resource_a ="A";
    private static final String resource_b = "B";

    private static void deadLock(){
        Thread thread1 = new Thread(() -> {
            synchronized (resource_a){
                System.out.println("get resource a");
                try {
                    TimeUnit.SECONDS.sleep(3);
                    synchronized (resource_b){
                        System.out.println("get resource b");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        Thread thread2= new Thread(()->{
            synchronized (resource_b){
                System.out.println("get resource b");
                synchronized (resource_a){
                    System.out.println("get resource a");
                }
            }
        });
        thread1.start();
        thread2.start();
    }

    public static void main(String[] args){
        deadLock();
    }
}
````



