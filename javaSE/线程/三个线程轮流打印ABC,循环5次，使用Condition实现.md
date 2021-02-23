````java
public class ConditionDemo {
    ShareData shareData = new ShareData();
    public static void main( String[] args ) {
        ConditionDemo demo = new ConditionDemo();
        demo.getThread("A",0,5);
        demo.getThread("B",1,5);
        demo.getThread("C",2,5);
    }

    public void getThread(String threadName,Integer num,Integer interval){
         new Thread(() -> {
            for (int i = 0; i < interval; i++) {
                shareData.print(num);
            }
        },threadName).start();
    }
}

class ShareData {
    private int num = 0; //A:1 B:2 C:3

    private final int NUMBER = 3;

    private Lock lock = new ReentrantLock();

    private Map<Integer, Condition> map = new HashMap<>();

    ShareData() {
        for (int i = 0; i < NUMBER; i++) {
            map.put(i, lock.newCondition());
        }
    }

    public void print( int number) {
        try {
            lock.lock();
            while (number != num) {
                map.get(number).await();
            }
            System.out.println(Thread.currentThread().getName());
            num = (++num) % NUMBER;
            map.get(num).signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
````

