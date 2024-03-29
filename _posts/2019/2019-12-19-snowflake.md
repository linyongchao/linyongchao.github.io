---
layout: post
title:  雪花算法
date:   2019-12-19 15:45:54
categories: Java
---

* content
{:toc}

参考[这里](https://www.cnblogs.com/relucent/p/4955340.html)和[这里](https://blog.csdn.net/u010398771/article/details/79765836)、[这里](https://blog.csdn.net/u010266988/article/details/87899533)

### 结构

Twitter 的雪花算法共计 64 位，恰好是一个 Long 类型，从左到右分四部分，分别是：
1. 第一部分，共 1 位，符号位，固定为 0
2. 第二部分，共 41 位，距自定义起始时间的时间戳（毫秒值），可用约69年
3. 第三部分，共 10 位，机器ID，可表示 2^10=1024台机器
4. 第四部分，共 12 位，毫秒内的序列号，2^12=4096

其结构如下图所示：  

![img](https://linyongchao.github.io/static/img/snowflake1.png) 

还有一种拆分方式，就是将第三部分的机器ID拆分成 5 位的数据中心ID和5位的机器ID，如下图所示：  

![img](https://linyongchao.github.io/static/img/snowflake2.png)

### 优势

雪花算法很好的满足了分布式主键的几个要求：
1. 全局唯一。因为有机器ID的原因，所以各个机器产生ID必然不同，而单机唯一很好保证
2. 趋势有序。因为以时间戳开头，所以是趋势有序的。但是，也正因为依赖了时间戳，所以机器时间绝对不允许回拨，一旦时间回拨，则无法保证单机唯一
3. 高性能。理论上单机情况下每秒支持生成 1000*2^12 = 409.6万ID，有人实测每秒可产生26万ID，已经足够应对大部分场景了

### 实现1

	/**
	 * 雪花算法
	 *
	 * @author
	 **/
	public class SnowflakeIdWorker {
	    /**
	     * 开始时间截 (2019-01-01 00:00:00)
	     */
	    private final long twepoch = 1546272000000L;
	
	    /**
	     * 机器id所占的位数
	     */
	    private final long workerIdBits = 5L;
	
	    /**
	     * 数据标识id所占的位数
	     */
	    private final long datacenterIdBits = 5L;
	
	    /**
	     * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
	     */
	    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);
	
	    /**
	     * 支持的最大数据标识id，结果是31
	     */
	    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
	
	    /**
	     * 序列在id中占的位数
	     */
	    private final long sequenceBits = 12L;
	
	    /**
	     * 机器ID向左移12位
	     */
	    private final long workerIdShift = sequenceBits;
	
	    /**
	     * 数据标识id向左移17位(12+5)
	     */
	    private final long datacenterIdShift = sequenceBits + workerIdBits;
	
	    /**
	     * 时间截向左移22位(5+5+12)
	     */
	    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
	
	    /**
	     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
	     */
	    private final long sequenceMask = -1L ^ (-1L << sequenceBits);
	
	    /**
	     * 工作机器ID(0~31)
	     */
	    private long workerId;
	
	    /**
	     * 数据中心ID(0~31)
	     */
	    private long datacenterId;
	
	    /**
	     * 毫秒内序列(0~4095)
	     */
	    private long sequence = 0L;
	
	    /**
	     * 上次生成ID的时间截
	     */
	    private long lastTimestamp = -1L;
	
	    /**
	     * 构造函数
	     *
	     * @param workerId     工作ID (0~31)
	     * @param datacenterId 数据中心ID (0~31)
	     */
	    public SnowflakeIdWorker(long workerId, long datacenterId) {
	        if (workerId > maxWorkerId || workerId < 0) {
	            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
	        }
	        if (datacenterId > maxDatacenterId || datacenterId < 0) {
	            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
	        }
	        this.workerId = workerId;
	        this.datacenterId = datacenterId;
	    }
	
	    /**
	     * 获得下一个ID (该方法是线程安全的)
	     *
	     * @return SnowflakeId
	     */
	    public synchronized long nextId() {
	        long timestamp = timeGen();
	
	        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
	        if (timestamp < lastTimestamp) {
	            throw new RuntimeException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
	        }
	
	        //如果是同一时间生成的，则进行毫秒内序列
	        if (lastTimestamp == timestamp) {
	            sequence = (sequence + 1) & sequenceMask;
	            //毫秒内序列溢出
	            if (sequence == 0) {
	                //阻塞到下一个毫秒,获得新的时间戳
	                timestamp = tilNextMillis(lastTimestamp);
	            }
	        } else {
	            //时间戳改变，毫秒内序列重置
	            sequence = 0L;
	        }
	
	        //上次生成ID的时间截
	        lastTimestamp = timestamp;
	
	        //移位并通过或运算拼到一起组成64位的ID
	        return ((timestamp - twepoch) << timestampLeftShift)
	                | (datacenterId << datacenterIdShift)
	                | (workerId << workerIdShift)
	                | sequence;
	    }
	
	    /**
	     * 阻塞到下一个毫秒，直到获得新的时间戳
	     *
	     * @param lastTimestamp 上次生成ID的时间截
	     * @return 当前时间戳
	     */
	    protected long tilNextMillis(long lastTimestamp) {
	        long timestamp = timeGen();
	        while (timestamp <= lastTimestamp) {
	            timestamp = timeGen();
	        }
	        return timestamp;
	    }
	
	    /**
	     * 返回以毫秒为单位的当前时间
	     *
	     * @return 当前时间(毫秒)
	     */
	    protected long timeGen() {
	        return System.currentTimeMillis();
	    }
	}

### 实现2

	/**
	 * 静态id生成器
	 *
	 * @author
	 */
	public class IdCenterUtil {
	    private volatile static IdCenterUtil instance = null;
	
	    private IdCenterUtil() {
	    }
	
	    public static IdCenterUtil getInstance() {
	        if (instance == null) {
	            synchronized (IdCenterUtil.class) {
	                if (instance == null) {
	                    instance = new IdCenterUtil();
	                    instance.initParam();
	                }
	            }
	        }
	        return instance;
	    }
	
	    /**
	     * 节点 ID 默认取1
	     */
	    private long workerId = 1;
	    /**
	     * 数据中心的ID 默认取1
	     */
	    private long datacenterId = 1;
	    /**
	     * 序列id 默认取1
	     */
	    private long sequence = 1;
	    /**
	     * 起始纪元
	     */
	    private long idepoch = 1546272000000L;
	    /**
	     * 机器标识位数
	     */
	    private final long workerIdBits = 5L;
	    /**
	     * 数据中心标识位数
	     */
	    private final long datacenterIdBits = 5L;
	    /**
	     * 机器ID最大值
	     */
	    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);
	    /**
	     * 数据中心ID最大值
	     */
	    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
	    /**
	     * 毫秒内自增位
	     */
	    private final long sequenceBits = 12L;
	    /**
	     * 机器ID偏左移12位
	     */
	    private final long workerIdShift = sequenceBits;
	    /**
	     * 数据中心ID左移17位
	     */
	    private final long datacenterIdShift = sequenceBits + workerIdBits;
	    /**
	     * 时间毫秒左移22位
	     */
	    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
	
	    private final long sequenceMask = -1L ^ (-1L << sequenceBits);
	
	    private long lastTimestamp = -1L;
	
	    private void initParam() {
	        if (workerId < 0 || workerId > maxWorkerId) {
	            throw new IllegalArgumentException("workerId is illegal: " + workerId);
	        }
	        if (datacenterId < 0 || datacenterId > maxDatacenterId) {
	            throw new IllegalArgumentException("datacenterId is illegal: " + workerId);
	        }
	        workerId = getIpFlag(4);
	        if (workerId > maxWorkerId) {
	            workerId = workerId % maxWorkerId;
	        }
	
	        datacenterId = getIpFlag(3);
	        if (datacenterId > maxDatacenterId) {
	            datacenterId = datacenterId % maxDatacenterId;
	        }
	    }
	
	    /**
	     * 获取ip第几位 从1开始
	     *
	     * @param flag
	     * @return
	     */
	    private Long getIpFlag(int flag) {
	        String ip = IpUtil.getLocalIp();
	        String[] ips = ip.split("[.]");
	        return Long.valueOf(ips[flag - 1]);
	    }
	
	    /**
	     * 获取id 线程安全
	     *
	     * @return
	     */
	    public synchronized long nextId() {
	        long timestamp = timeGen();
	        // 时间错误
	        if (timestamp < lastTimestamp) {
	            throw new IllegalStateException("Clock moved backwards.");
	        }
	        // 当前毫秒内，则+1
	        if (lastTimestamp == timestamp) {
	            // 当前毫秒内计数满了，则等待下一秒
	            sequence = (sequence + 1) & sequenceMask;
	            if (sequence == 0) {
	                timestamp = tilNextMillis(lastTimestamp);
	            }
	        } else {
	            sequence = 0;
	        }
	        lastTimestamp = timestamp;
	        // 偏移组合生成最终的ID，并返回
	        long id = ((timestamp - idepoch) << timestampLeftShift) | (datacenterId << datacenterIdShift) | (workerId << workerIdShift) | sequence;
	        return id;
	    }
	
	    /**
	     * 等待下一个毫秒的到来
	     *
	     * @param lastTimestamp
	     * @return
	     */
	    private long tilNextMillis(long lastTimestamp) {
	        long timestamp = timeGen();
	        while (timestamp <= lastTimestamp) {
	            timestamp = timeGen();
	        }
	        return timestamp;
	    }
	
	    private long timeGen() {
	        return System.currentTimeMillis();
	    }
	
	}

需要依赖获取本地 IP 的工具类	
	
	/**
	 * IpUtil
	 *
	 * @author
	 * @return
	 **/
	public class IpUtil {
	
	    private static final Logger log = LoggerFactory.getLogger(IpUtil.class);
	
	    /**
	     * Windows判断标准
	     **/
	    private static final String WIN = "windows";
	
	    /**
	     * 判断当前操作是否Windows.
	     *
	     * @return true---是Windows操作系统
	     */
	    public static boolean isWindows() {
	        boolean isWindows = false;
	        String osName = System.getProperty("os.name");
	        if (osName.toLowerCase().indexOf(WIN) > -1) {
	            isWindows = true;
	        }
	        return isWindows;
	    }
	
	    /**
	     * 获取本机IP地址，并自动区分Windows还是Linux操作系统
	     *
	     * @return String
	     */
	    public static String getLocalIp() {
	        InetAddress ip;
	        List<InetAddress> innerIpList = new ArrayList<>();
	        try {
	            // 如果是Windows操作系统
	            if (isWindows()) {
	                ip = InetAddress.getLocalHost();
	                innerIpList.add(ip);
	            } else {
	                // 如果是Linux操作系统
	                Enumeration<NetworkInterface> netInterfaces = NetworkInterface.getNetworkInterfaces();
	                while (netInterfaces.hasMoreElements()) {
	                    NetworkInterface ni = netInterfaces.nextElement();
	                    // 遍历所有ip
	                    Enumeration<InetAddress> ips = ni.getInetAddresses();
	                    while (ips.hasMoreElements()) {
	                        ip = ips.nextElement();
	                        if (ip.isSiteLocalAddress() && !ip.isLoopbackAddress()
	                                && ip.getHostAddress().indexOf(":") == -1) {
	                            innerIpList.add(ip);
	                            log.info("Register Service Ip=" + ip);
	                        } else {
	                            log.info("Skip Service Ip=" + ip);
	                        }
	                    }
	                }
	            }
	        } catch (Exception e) {
	            log.error("get local ip error:", e);
	        }
	        return innerIpList.size() > 0 ? innerIpList.get(0).getHostAddress() : "";
	    }
	}

### 测试

    public static void main(String[] args) {
        int num = 100;
        SnowflakeIdWorker idWorker = new SnowflakeIdWorker(0, 0);
        for (int i = 0; i < num; i++) {
            long id = idWorker.nextId();
            System.out.println(id);
        }
        System.out.println("---------");
        for (int i = 0; i < num; i++) {
            long id = IdCenterUtil.getInstance().nextId();
            System.out.println(id);
        }
    }

