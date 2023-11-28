Fork/Join框架是Java7提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架

工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列

ForkJoinPool是ExecutorService的实现类，因此是一种特殊的线程池。ForkJoinPool提供了如下两个常用的构造器

```java
// 创建一个包含parallelism个并行线程的ForkJoinPool
public ForkJoinPool(int parallelism)

//以Runtime.getRuntime().availableProcessors()的返回值作为parallelism来创建ForkJoinPool
public ForkJoinPool() ：
```

创建ForkJoinPool实例后，可以钓鱼ForkJoinPool的submit(ForkJoinTask<T> task)或者invoke(ForkJoinTask<T> task)来执行指定任务。其中ForkJoinTask代表一个可以并行、合并的任务。ForkJoinTask是一个抽象类，它有两个抽象子类：RecursiveAction和RecursiveTask。

- RecursiveTask代表有返回值的任务
- RecursiveAction代表没有返回值的任务。

```java
public class RaskDemo extends RecursiveAction {
    /**
     *  每个"小任务"最多只打印20个数
      */
    private static final int MAX = 20;

    private int start;
    private int end;

    public RaskDemo(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected void compute() {
        //当end-start的值小于MAX时，开始打印
        if((end-start) < MAX) {
            for(int i= start; i<end;i++) {
                System.out.println(Thread.currentThread().getName()+"i的值"+i);
            }
        }else {
            // 将大任务分解成两个小任务
            int middle = (start + end) / 2;
            RaskDemo left = new RaskDemo(start, middle);
            RaskDemo right = new RaskDemo(middle, end);
            left.fork();
            right.fork();
        }
    }
}

public static void main(String[] args) throws Exception{
    // 创建包含Runtime.getRuntime().availableProcessors()返回值作为个数的并行线程的ForkJoinPool
    ForkJoinPool forkJoinPool = new ForkJoinPool();

    // 提交可分解的PrintTask任务
    forkJoinPool.submit(new RaskDemo(0, 1000));

    //阻塞当前线程直到 ForkJoinPool 中所有的任务都执行结束
    forkJoinPool.awaitTermination(2, TimeUnit.SECONDS);

    // 关闭线程池
    forkJoinPool.shutdown();
}
```

```java
public class RecursiveTaskDemo extends RecursiveTask<Integer> {

    /**
     *  每个"小任务"最多只打印70个数
     */
    private static final int MAX = 70;
    private int arr[];
    private int start;
    private int end;

    public RecursiveTaskDemo(int[] arr, int start, int end) {
        this.arr = arr;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        // 当end-start的值小于MAX时候，开始打印
        if((end - start) < MAX) {
            for (int i = start; i < end; i++) {
                sum += arr[i];
            }
            return sum;
        }else {
            System.err.println("=====任务分解======");
            // 将大任务分解成两个小任务
            int middle = (start + end) / 2;
            RecursiveTaskDemo left = new RecursiveTaskDemo(arr, start, middle);
            RecursiveTaskDemo right = new RecursiveTaskDemo(arr, middle, end);
            // 并行执行两个小任务
            left.fork();
            right.fork();
            // 把两个小任务累加的结果合并起来
            return left.join() + right.join();
        }
    }

}
@Test
    public void dfs() throws Exception{
        int arr[] = new int[1000];
        Random random = new Random();
        int total = 0;
        // 初始化100个数字元素
        for (int i = 0; i < arr.length; i++) {
            int temp = random.nextInt(100);
            // 对数组元素赋值,并将数组元素的值添加到total总和中
            total += (arr[i] = temp);
        }
        System.out.println("初始化时的总和=" + total);

        // 创建包含Runtime.getRuntime().availableProcessors()返回值作为个数的并行线程的ForkJoinPool
        ForkJoinPool forkJoinPool = new ForkJoinPool();

        // 提交可分解的PrintTask任务
//        Future<Integer> future = forkJoinPool.submit(new RecursiveTaskDemo(arr, 0, arr.length));
//        System.out.println("计算出来的总和="+future.get());

        Integer integer = forkJoinPool.invoke( new RecursiveTaskDemo(arr, 0, arr.length)  );
        System.out.println("计算出来的总和=" + integer);

        // 关闭线程池
        forkJoinPool.shutdown();
    }
```