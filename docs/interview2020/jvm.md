## gc
    新生代
    
    
## synchronized
    synchronized保证了原子性、有序性、可见性
    首先是原子性，synchronized是加锁操作，所有其他线程无法同时运行该代码，通过串行保证了原子性；
    有序性，synchronized块内代码无法保证有序，但是能保证方法块和前后代码有序，从而实现有序性；
    可见性，synchronized在方法执行前会从新获取主缓存数据，并且自执行完成后，会把数据提交到主缓存；
    
## volatile
    volatile保证了可见性和有序性；
    volatile的变量会强制刷新到主缓存；
    volatile修饰的变量不允许重排序；
    