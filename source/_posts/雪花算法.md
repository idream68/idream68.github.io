---
title: 雪花算法
date: 2021-04-30 17:05:00
categories: 
  - java
tags:
  - 算法
  - java
typora-root-url: ..
---

使用一个 64 bit 的 long 型的数字作为全局唯一 id。在分布式系统中的应用十分广泛，且ID 引入了时间戳，基本上保持自增的，后面的代码中有详细的注解。

 

这 64 个 bit 中，其中 1 个 bit 是不用的，然后用其中的 41 bit 作为毫秒数，用 10 bit 作为工作机器 id，12 bit 作为序列号。

 ![image](/images/雪花算法.png)

雪花算法简单描述：
+ 最高位是符号位，始终为0，不可用。
+ 41位的时间序列，精确到毫秒级，41位的长度可以使用69年。时间位还有一个很重要的作用是可以根据时间进行排序。
+ 10位的机器标识，10位的长度最多支持部署1024个节点。
+ 12位的计数序列号，序列号即一系列的自增id，可以支持同一节点同一毫秒生成多个ID序号，12位的计数序列号支持每个节点每毫秒产生4096个ID序号。

看的出来，这个算法很简洁也很简单，但依旧是一个很好的ID生成策略。其中，10位器标识符一般是5位IDC+5位machine编号，唯一确定一台机器。

```java
public class SnowFlakeGenerator {

    public static class Factory {
        /**
         * 每一部分占用位数的默认值
         */
        private final static int DEFAULT_MACHINE_BIT_NUM = 5;   //机器标识占用的位数
        private final static int DEFAULT_IDC_BIT_NUM = 5;//数据中心占用的位数

        private int machineBitNum;
        private int idcBitNum;

        public Factory() {
            this.idcBitNum = DEFAULT_IDC_BIT_NUM;
            this.machineBitNum = DEFAULT_MACHINE_BIT_NUM;
        }

        public Factory(int machineBitNum, int idcBitNum) {
            this.idcBitNum = idcBitNum;
            this.machineBitNum = machineBitNum;
        }

        public SnowFlakeGenerator create(long idcId, long machineId) {
            return new SnowFlakeGenerator(this.idcBitNum, this.machineBitNum, idcId, machineId);
        }
    }

    /**
     * 起始的时间戳
     * 作者写代码时的时间戳
     */
    private final static long START_STAMP = 1508143349995L;

    /**
     * 可分配的位数
     */
    private final static int REMAIN_BIT_NUM = 22;

    /**
     * idc编号
     */
    private long idcId;

    /**
     * 机器编号
     */
    private long machineId;

    /**
     * 当前序列号
     */
    private long sequence = 0L;

    /**
     * 上次最新时间戳
     */
    private long lastStamp = -1L;

    /**
     * idc偏移量：一次计算出，避免重复计算
     */
    private int idcBitLeftOffset;

    /**
     * 机器id偏移量：一次计算出，避免重复计算
     */
    private int machineBitLeftOffset;

    /**
     * 时间戳偏移量：一次计算出，避免重复计算
     */
    private int timestampBitLeftOffset;

    /**
     * 最大序列值：一次计算出，避免重复计算
     */
    private int maxSequenceValue;

    private SnowFlakeGenerator(int idcBitNum, int machineBitNum, long idcId, long machineId) {
        int sequenceBitNum = REMAIN_BIT_NUM - idcBitNum - machineBitNum;

        if (idcBitNum <= 0 || machineBitNum <= 0 || sequenceBitNum <= 0) {
            throw new IllegalArgumentException("error bit number");
        }

        this.maxSequenceValue = ~(-1 << sequenceBitNum);

        machineBitLeftOffset = sequenceBitNum;
        idcBitLeftOffset = idcBitNum + sequenceBitNum;
        timestampBitLeftOffset = idcBitNum + machineBitNum + sequenceBitNum;

        this.idcId = idcId;
        this.machineId = machineId;
    }

    /**
     * 产生下一个ID
     */
    public synchronized long nextId() {
        long currentStamp = getTimeMill();
        if (currentStamp < lastStamp) {
            throw new RuntimeException(String.format("Clock moved backwards. Refusing to generate id for %d milliseconds", lastStamp - currentStamp));
        }

        //新的毫秒，序列从0开始，否则序列自增
        if (currentStamp == lastStamp) {
            sequence = (sequence + 1) & this.maxSequenceValue;
            if (sequence == 0L) {
                //Twitter源代码中的逻辑是循环，直到下一个毫秒
                lastStamp = tilNextMillis();
//                throw new IllegalStateException("sequence over flow");
            }
        } else {
            sequence = 0L;
        }

        lastStamp = currentStamp;

        return (currentStamp - START_STAMP) << timestampBitLeftOffset | idcId << idcBitLeftOffset | machineId << machineBitLeftOffset | sequence;
    }

    private long getTimeMill() {
        return System.currentTimeMillis();
    }

    private long tilNextMillis() {
        long timestamp = getTimeMill();
        while (timestamp <= lastStamp) {
            timestamp = getTimeMill();
        }
        return timestamp;
    }
}
```





参考：

[雪花算法(snowflake)](https://blog.csdn.net/u011499747/article/details/78254990)
[雪花算法的原理和实现Java](https://blog.csdn.net/lq18050010830/article/details/89845790)