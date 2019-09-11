---
title: Java多线程（五）之线程的通信 
date: 2019-08-09 22:46:38 
tags: 
    - Java 
    - 多线程
---

1.wait() 与 notify() 和 notifyAll()

wait()：令当前线程挂起并放弃CPU、同步资源并等待，使别的线程可访问并修改共享资源，而当前线程排队等候其他线程调用notify()或notifyAll()方法唤醒，唤醒后等待重新获得对监视器的所有权后才能继续执行

notify()：唤醒正在排队等待同步资源的线程中优先级最高者结束等待

notifyAll ()：唤醒正在排队等待资源的所有线程结束等待.

注意：
  --->这三个方法只有在synchronized方法或synchronized代码块中才能使用，否则会报
java.lang.IllegalMonitorStateException异常

  --->因为这三个方法必须有锁对象调用，而任意对象都可以作为synchronized的同步锁，
因此这三个方法只能在Object类中声明。

<!--more-->

2.wait() 方法

 --->  在当前线程中调用方法： 对象名.wait()

 --->  使当前线程进入等待（某对象）状态 ，直到另一线程对该对象发出 notify (或notifyAll) 为止。

 --->  调用方法的必要条件：当前线程必须具有对该对象的监控权（加锁）

 --->  调用此方法后，当前线程将释放对象监控权 ，然后进入等待

 --->  在当前线程被notify后，要重新获得监控权，然后从断点处继续代码的执行。


3.notify()/notifyAll()

 --->  在当前线程中调用方法： 对象名.notify()

 --->  功能：唤醒等待该对象监控权的一个/所有线程。

 --->  调用方法的必要条件：当前线程必须具有对该对象的监控权（加锁）


4. 经典例题：生产者/消费者问题

  生产者(Productor)将产品交给店员(Clerk)，而消费者(Customer)从店员处取走产品，店员一次只能持有固定数量的产品(比如:20），如果生产者试图生产更多的产品，店员会叫生产者停一下，如果店中有空位放产品了再通知生产者继续生产；如果店中没有产品了，店员会告诉消费者等一下，如果店中有产品了再通知消费者来取走产品。


这里可能出现两个问题：

--->生产者比消费者快时，消费者会漏掉一些数据没有取到。

--->消费者比生产者快时，消费者会取相同的数据。

    class Clerk { // 售货员
    private int product = 0;

    public synchronized void addProduct() {
        if (product >= 20) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } else {
            product++;
            System.out.println("生产者生产了第" + product + " 个产品");
                    notifyAll();
        }
    }

    public synchronized void getProduct() {
        if (this.product <= 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("消费者取走了第" +
                    product + "个产品");
            product--;
            notifyAll();
        }
     }
    }



    class Productor implements Runnable { // 生产者
    Clerk clerk;

    public Productor(Clerk clerk) {
        this.clerk = clerk;
    }

    public void run() {
        System.out.println("生产者开始生产产品");
        while (true) {
            try {
                Thread.sleep((int) Math.random() * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            clerk.addProduct();
        }
     }
    }


	class Consumer implements Runnable { // 消费者
	    Clerk clerk;
	    public Consumer(Clerk clerk) {
	        this.clerk = clerk;
	    }
	    public void run() {
	        System.out.println("消费者开始取走产品");
	        while (true) {
	            try {
	                Thread.sleep((int) Math.random() * 1000);
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	            clerk.getProduct();
	        }
	    }
	}


	public class ProductTest {
	    public static void main(String[] args) {
	        Clerk clerk = new Clerk();
	        Thread productorThread = new Thread(new Productor(clerk));
	        Thread consumerThread = new Thread(new Consumer(clerk));
	        productorThread.start();
	        consumerThread.start();
	    }
	}

