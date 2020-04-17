---
title: 苦命管家之巧解草莓风波
tags:
  - Algorithm
  - 分布式
categories:
  - normal
date: 2017-02-19 18:24:50
---


阿哲是村东边的大户，他有一个特别厉害的管家，能说会道，智商超人，说到草莓风波...

<!-- more -->


事起
===

老爷平日没啥爱好，年前下苏杭时候碰巧尝了一次草莓，至此每日念念不忘，招来管家...

原来是老爷嘴馋了
---
  嘴馋也有讲究，不是胡吃海喝，咱们要持续性发展...
  草莓这物件，这年代也没冰箱什么的，最新鲜的就是每日采摘，老爷做的长远打算，每日一醒就对草莓心心念念，所以啊，就想越早越好，晚来一点也没关系，但是必须是今天摘的才行，但是一天都没吃到好草莓，我可是要发飙哦！
  还有，这草莓虽好，但是也不能贪杯哦...啊呸，不能多吃，每日一袋是为最好，之后来的，就把他们退掉吧！

老爷想要的是什么
---
对于老爷的想法，管家自然是摸得透彻，稍作盘算，就知道老爷想的是啥

1. 每天都需要消耗一批草莓(尽最大努力)
2. 每天仅吃一批草莓
3. 只吃当日最新鲜的、质量过关的草莓，至于是谁提供的并不重要
4. 如果有多余的或者不合格的草莓，则需要把他们处理掉(退货)


召集商家
---

任务吩咐下去，让管家一手操办，老板提出了需求，那管家就只能跑跑腿了～ 要拿到草莓，那得首先联系一下草莓供应商是个什么情况

几日之后，很快就有了消息，这几家供应商，因为草莓属于现摘，每日的情况都不一样，供应商每天送来都货物质量也都参差不齐，需要管家一一验过
如果货物质量不行，那么就只能退货啦，又或者我们老爷当日只能吃过草莓了，那么抱歉，你这批货我只能退啦，要是你来的时候，管家还在验上家的货，那你可以碰碰运气，万一上家的货物不过关呢，是吧～

简而言之，市场竞争真是激烈呀！

那么提炼一下：
1. 不保证每日都有货物
2. 不保证货物的质量
3. 不保证到货时间
4. 接受退货
5. 供应商在货物送到之前，并不知晓老爷今日是否已经享用了草莓


嗯，任性的供应商提供的服务还真是不怎么可靠啊，但是这总难不住聪明的管家，咋一看，虽然每一家货物提供商并不能提供完全可靠的服务，但是我多喊几家一起上，那么老爷的需求还是可以尽量满足的嘛，想到这里，管家就被自己的聪明才智锁折服了，心里还有点小激动呢 (ಡωಡ)hiahiahia

心中一阵盘算之后，通知供应商们召集到院中开了一次集体会议....

一个月黑风高的夜晚
====

众商家，今日将大家召集到此，是为了通知各商家之后咱们的统一供货方式，一方面尽量保证老爷的要求，一方面呢，也是和大家打个招呼，咱们以后就按照这个法子来，互通有无嘛

之后每日，若各位货物到后，可将货物先置放于院中，然后由知晓我，待我来对货物进行验收，验收通过之后，向你发放当日款项。若通知我时，我已找到一批新鲜的货物，那么你货的新鲜程度不如上家，自然是输了，那么我就无法给你款项，只能将货退还给你。若我还在验货，那你可以再等等，若之前的货物不合格，那么你可以等等，若上家的货物不符合老爷的要求，那么我就可以来验你的货啦～

各商家见管家提出如此妥善的注意，事少，也不用和别家伤和气，一切买卖全凭本事，一个一个也摩拳擦掌，纷纷表示赞同

单机版
===

一段时日之后，老爷如愿吃上了想要的草莓🍓，管家也将此方法记录了下来，以便改进与学习 :)

商家p1、p2属于集合P`(Provider)`，商家提供的服务都是同质的，可以归为一类，分别为
1. 送货 deliver
2. 验货 validate
3. 退货 takeBack

而管家g`(governor)`的状态分为几种
1. 尚未验货 available
2. 正在验货 validating
3. 验货结束 unavailable

那么商家所执行的步骤可以用如下伪代码表示：

```java
public class ProviderDeliver {

  public void dailyWork(Provider p, Governor g){}

  p.deliver();

  if (g.unavailable()) {
    p.takeBack();
    return ;
  }

  while (g.validating()) {
    // simply wait
  }

  if (g.available()) {
    p.validate();
  } else {
    p.takeBack();
  }
}
```

分布式版
===

哎呀，老爷一高兴，赏金给得特别高，各路商家接踵而至，管家要忙不过来了，于是吩咐下去几个家丁，分别接待商户，检查依然由管家亲自操作，但商户无需等待，仅需要将货物放在府中，退货由家丁完成，而商户也不直接与管家接触，由家丁负责商家和管家之间的沟通，管家通过府邸中的旗帜表示自己目前正在验货、休息、验货完毕

那么整体的流程作一下简单转换，对商户来说，流程更加简单

```java

/**
 *
 * Created by kevin on 20/02/2017.
 */
public class Strawberry {

    static AtomicInteger errorCount = new AtomicInteger(0);
    static AtomicInteger takeBackCount = new AtomicInteger(0);
    static AtomicInteger consumedCount = new AtomicInteger(0);
    static AtomicInteger acceptedCount = new AtomicInteger(0);
    static Random r = new Random();
    static class ProviderDeliver2 {

        private Provider p;

        ProviderDeliver2(String name) {
            this.p = new Provider(name);
        }

        void dailyWork(int day) {
            Merchandise m = p.deliver(); // 运送货物
            Worker worker = Worker.of(this, day); // 随便找个家丁，并把货物交给他
            worker.process(m); //  家丁来处理货物
        }

        void takeBack(Merchandise m) {
            int res = m.getTime().incrementAndGet();
            if (res == 1) {
                takeBackCount.incrementAndGet();
            } else if (res > 1) {
                System.out.println("fail! operate " + res );
                errorCount.incrementAndGet();
            }
        }
    }

    static class Worker {

        private Governor g;
        private ProviderDeliver2 pd2;

        Worker(ProviderDeliver2 pd2, Governor g) {
            this.pd2 = pd2;
            this.g = g;
        }

        static Worker of(ProviderDeliver2 pd2, int day) {
            return new Worker(pd2, Governor.getInstance(day));
        }

        void process(Merchandise m) {
            g.validate(this, m);
        }

        void takeBack(Merchandise m) {
            pd2.takeBack(m);
        }
    }

    static class Governor {

        private AtomicBoolean available = new AtomicBoolean(true);
        private AtomicBoolean validating = new AtomicBoolean(false);
        private Queue<Merchandise> queue = Queues.newConcurrentLinkedQueue();
        private int date;

        private static ConcurrentHashMap<Integer, Governor> MAP = new ConcurrentHashMap<>();

        static Governor getInstance(int day) {
            return MAP.computeIfAbsent(day, Governor::new);
        }

        boolean available() {
            return available.get();
        }

        private Governor(int date) {
            this.date = date;
        }

        void validate(Worker worker, Merchandise m) {
            try {
                if (!available()) {
                    worker.takeBack(m);
                    return;
                }

                this.addMerchandise(m);

                if (!validating.compareAndSet(false, true)) {
                    return;
                }

                if (!available()) {
                    cleanAll(worker);
                    return;
                }
                while (!queue.isEmpty()) {
                    Merchandise item = queue.poll();
                    if (available() && item.isGood()) {
                        available.set(false);
                        acceptedCount.incrementAndGet();
                    } else {
                        worker.takeBack(item);
                    }
                }
                validating.set(false);

            } finally {
                consumedCount.incrementAndGet();
            }
        }

        private void cleanAll(Worker worker) {
            while (!queue.isEmpty()) {
                Merchandise item = queue.poll();
                worker.takeBack(item);
            }
        }
        void addMerchandise(Merchandise m) {
            queue.add(m);
        }
    }

    public static class Provider {

        private String name;

        Provider(String name) {
            this.name = name;
        }

        Merchandise deliver() {
            return new Merchandise(this.name + System.nanoTime());
        }
    }

    static class Merchandise {

        private String name;
        private AtomicInteger time = new AtomicInteger(0);
        private long quality = r.nextInt(100);

        AtomicInteger getTime() {
            return time;
        }

        Merchandise(String name) {
            this.name = name;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;

            Merchandise that = (Merchandise) o;

            return name != null ? name.equals(that.name) : that.name == null;
        }

        @Override
        public int hashCode() {
            return name != null ? name.hashCode() : 0;
        }

        boolean isGood() {
//            return true;
            return quality < 80;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        // 模仿5个商家，连续提供500日的情况
        int k = 0;
        for (int day = 0; day < 500; day++) {
            int tmpDay = day;
            for (int i = 0; i < 5; i++) {
                new Thread(() -> new ProviderDeliver2(String.valueOf(tmpDay)).dailyWork(tmpDay)).start();
                k++;
            }
        }
        while (k != consumedCount.get()) {
            Thread.sleep(10);
        }
        System.out.println("done");
        System.out.println(errorCount.get()); //0
        System.out.println(takeBackCount.get()); //
        System.out.println(acceptedCount.get()); //
        System.out.println(consumedCount.get()); //
    }
}
```
