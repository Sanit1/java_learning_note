#### ConcurrentHashMap

HashMap线程不安全，在jdk 1.7时，HashMap使用头插法插入链表，所以每次发生扩容的时候，进行链表复制时，都会翻转链表，所以并行put的并发生扩容的时候有可能导致HashMap的Entry链表形成环形数据结构，next永不为空，陷入死循环

```java
package com.company.concurrentclass.hashmapdeadend;

import java.util.HashMap;
import java.util.UUID;

public class DeadEndHashMap {
    public static void main(String[] args) throws InterruptedException {
        final HashMap<String, String> map = new HashMap<>(2);
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    new Thread(new Runnable() {
                        @Override
                        public void run() {
                            map.put(UUID.randomUUID().toString(), "");
                        }
                    }, "ftf" + i).start();
                }
            }
        }, "ftf");
        t.start();
        t.join();
    }
}
```

以上的bug得在jdk8前才能出现，jdk8后修复了hashmap死循环的问题，但是他依旧是线程不安全的。



ConcurrentHashMap使用了分段锁技术，一段数据配一把锁，其它段的数据不受影响（jdk1.8改为CAS锁，并引入红黑树）。HashMap key value可以为null 但是ConcurrentHashMap不允许，直接会判处异常



