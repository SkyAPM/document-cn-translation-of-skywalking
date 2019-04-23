## Spring注解插件
这个插件可以实现对被`@Bean`, `@Service`, `@Component` and `@Repository`注解标注的bean的所有方法的追踪。

- 为什么这个插件是可选的？  

追踪bean的所有方法会创建大量的span，这会导致耗费更多的CPU、内存和网络带宽。
当前如果你想追踪尽可能多的方法，请确保系统负载可以支撑你这么做。
