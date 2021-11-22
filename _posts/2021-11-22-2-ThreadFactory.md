






---
layout: post
title: "DefaultThreadFactory"
date: 2021-10-15
description: "DefaultThreadFactory"
tag: hexo
---   
## 介绍
DefaultThreadFactory






        static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;
        
                DefaultThreadFactory() {
                    SecurityManager s = System.getSecurityManager();
                    group = (s != null) ? s.getThreadGroup() :
                                          Thread.currentThread().getThreadGroup();
                    namePrefix = "pool-" +
                                  poolNumber.getAndIncrement() +
                                 "-thread-";
                }
        
                public Thread newThread(Runnable r) {
                    Thread t = new Thread(group, r,
                                          namePrefix + threadNumber.getAndIncrement(),
                                          0);
                    if (t.isDaemon())
                        t.setDaemon(false);
                    if (t.getPriority() != Thread.NORM_PRIORITY)
                        t.setPriority(Thread.NORM_PRIORITY);
                    return t;
                }
            }
