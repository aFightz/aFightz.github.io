---
title: 08 | 线程池的使用
date: 2019-01-19 12:23:59
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

在线程池的线程中不应该使用ThreadLocal在任务之间传递值。

在线程池中，如果任务依赖于其他任务，那么可能产生饥饿死锁。除非线程池足够大。

对于计算密集型的任务，在拥有N个处理器的系统上，当线程池的大小为N+1时，通常能实现最优的利用率。（即使当计算密集型的线程偶尔由于页缺失故障或者其他原因而暂停时，这个“额外”的线程也能确保CPU的时钟周期不会被浪费。）

线程数量 = CPU数量 * CPU利用率 * (1 + W/C)
W为线程等待时间，C为线程计算时间。
W+C即是一个线程任务实际运行时间。

int N_CPUS=Runtime.getRuntime().availableProcessors(); 


线程池构造方法
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    ...
}

线程池工作过程
keepAliveTime：线程存活时间。
Core Pool Size：线程池的基本大小，在没有任务执行时线程池的大小，并且只有在工作队列满了的情况下才会创建超出这个数量的线程。
Maximum Pool Size：最大大小。表示可同时活动的线程数量的上限。如果某个线程的空闲时间超过了存活时间，那么将被标记为可回收的，并且当线程池的当前大小超过了基本大小时，这个线程将被终止。

newFixedThreadPool工厂方法将线程池的基本大小和最大大小设置为参数中指定的值，而且创建的线程池不会超时。
newCachedThreadPool工厂方法将线程池的最大大小设置为Integer.MAX_VALUE，而将基本大小设置为零，并将超时设置为1分钟，这种方法创建出来的线程池可以被无限扩展，并且当需求降低时会自动收缩。

如果线程池中的线程数量等于线程池的基本大小，那么仅当在工作队列已满的情况下ThreadPoolExecutor才会创建新的线程。

默认的线程池饱和策略是“中止”：即抛出RejectedExecutionException。

可用Executors的unconfigurableExecutorService工厂方法包装ExecutorService，避免其被修改。

可继承ThreadPoolExecutor的beforeExecute、afterExecute和terminated以实现一些日志统计等功能。

等待任务集返回
//提交任务
exec.shutdown();
exec.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS);
//返回任务结果



@Immutable
public class PuzzleNode<P, M> {
    final P pos;
    final M move;
    final PuzzleNode<P, M> prev;
 
    public PuzzleNode(P pos, M move, PuzzleNode<P, M> prev) {
        this.pos = pos;
        this.move = move;
        this.prev = prev;
    }
 
    List<M> asMoveList() {
        List<M> solution = new LinkedList<M>();
        for (PuzzleNode<P, M> n = this; n.move != null; n = n.prev)
            solution.add(0, n.move);
        return solution;
    }
}

串行版本
/** 串行的谜题解答器*/
public class SequentialPuzzleSolver<P, M> {
    private final Puzzle<P, M> puzzle;
    private final Set<P> seen = new HashSet<P>();
 
    public SequentialPuzzleSolver(Puzzle<P, M> puzzle) {
        this.puzzle = puzzle;
    }
 
    public List<M> solve() {
        P pos = puzzle.initialPosition();
        return search(new PuzzleNode<P, M>(pos, null, null));
    }
 
    private List<M> search(PuzzleNode<P, M> node) {
        if (!seen.contains(node.pos)) {
            seen.add(node.pos);
            if (puzzle.isGoal(node.pos))
                return node.asMoveList();
            for (M move : puzzle.legalMoves(node.pos)) {
                P pos = puzzle.move(node.pos, move);
                PuzzleNode<P, M> child = new PuzzleNode<P, M>(pos, move, node);
                List<M> result = search(child);
                if (result != null)
                    return result;
            }
        }
        return null;
    }
}

并行版本
public class ConcurrentPuzzleSolver<P, M> {
    private final Puzzle<P, M> puzzle;
    private final ExecutorService exec;
    private final ConcurrentMap<P, Boolean> seen;
    protected final ValueLatch<PuzzleNode<P, M>> solution = new ValueLatch<PuzzleNode<P, M>>();
 
    public ConcurrentPuzzleSolver(Puzzle<P, M> puzzle) {
        this.puzzle = puzzle;
        this.exec = initThreadPool();
        this.seen = new ConcurrentHashMap<P, Boolean>();
        if (exec instanceof ThreadPoolExecutor) {
            ThreadPoolExecutor tpe = (ThreadPoolExecutor) exec;
            tpe.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy());
        }
    }
 
    private ExecutorService initThreadPool() {
        return Executors.newCachedThreadPool();
    }
 
    public List<M> solve() throws InterruptedException {
        try {
            P p = puzzle.initialPosition();
            exec.execute(newTask(p, null, null));
            // block until solution found
            PuzzleNode<P, M> solnPuzzleNode = solution.getValue();
            return (solnPuzzleNode == null) ? null : solnPuzzleNode.asMoveList();
        } finally {
            exec.shutdown();
        }
    }
 
    protected Runnable newTask(P p, M m, PuzzleNode<P, M> n) {
        return new SolverTask(p, m, n);
    }
 
    protected class SolverTask extends PuzzleNode<P, M> implements Runnable {
        SolverTask(P pos, M move, PuzzleNode<P, M> prev) {
            super(pos, move, prev);
        }
 
        public void run() {
            if (solution.isSet() || seen.putIfAbsent(pos, true) != null)
                return; // already solved or seen this position
            if (puzzle.isGoal(pos))
                solution.setValue(this);
            else
                for (M m : puzzle.legalMoves(pos))
                    exec.execute(newTask(puzzle.move(pos, m), m, this));
        }
    }
}

串行版本的程序执行深度优先搜索，因此搜索过程将受限于栈的大小。并发版本的程序执行广度优先搜索，因此不会受到栈大小的限制（但如果待搜索的或者已搜索的位置集合大小超过了可用的内存总量，那么仍可能耗尽内存）。