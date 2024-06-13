
参考文章：
- 2.perfetto cpu监控  
https://www.jianshu.com/p/0a8dd1d866f5
- 3.cpu提频  
[Systrace 线程 CPU 运行状态分析技巧 - Runnable 篇](https://www.androidperformance.com/2022/01/21/android-systrace-cpu-state-runnable/#/Runnable-%E7%8A%B6%E6%80%81%E5%9C%A8-Trace-%E4%B8%AD%E7%9A%84%E6%98%BE%E7%A4%BA%E6%96%B9%E5%BC%8F)
[Android 高版本采集系统CPU使用率的方式](https://juejin.cn/post/7135034198158475300)  
[Android平台下的cpu利用率优化实现](https://juejin.cn/post/7243240618788388922)

- [io wait](https://www.percona.com/blog/understanding-linux-iowait/)
- BlockCanaryX 实践  
  添加cpu追踪信息。



### android源码：获取线程cpu使用时间
android.os.Debug#threadCpuTimeNanos
dalvik.system.VMDebug#threadCpuTimeNanos
art/runtime/native/dalvik_system_VMDebug.cc#VMDebug_threadCpuTimeNanos
art/libartbase/base/time_utils.cc#ThreadCpuNanoTime //线程cpu时间
art/libartbase/base/time_utils.cc#ProcessCpuNanoTime //进程cpu时间
```c++
uint64_t ThreadCpuNanoTime() {
#if defined(__linux__)
  timespec now;
  clock_gettime(CLOCK_THREAD_CPUTIME_ID, &now);
  return static_cast<uint64_t>(now.tv_sec) * UINT64_C(1000000000) + now.tv_nsec;
#else
  UNIMPLEMENTED(WARNING);
  return -1;
#endif
}

uint64_t ProcessCpuNanoTime() {
#if defined(__linux__)
  timespec now;
  clock_gettime(CLOCK_PROCESS_CPUTIME_ID, &now);
  return static_cast<uint64_t>(now.tv_sec) * UINT64_C(1000000000) + now.tv_nsec;
#else
  // We cannot use clock_gettime() here. Return the process wall clock time
  // (using art::NanoTime, which relies on gettimeofday()) as approximation of
  // the process CPU time instead.
  //
  // Note: clock_gettime() is available from macOS 10.12 (Darwin 16), but we try
  // to keep things simple here.
  return NanoTime();
#endif
}
```