---
layout: post
title: "OutputStream转InputStream的两种方案"
date: 2014-11-29
categories: java
tags: jdk io piped 管道流
---

项目中需要将动态数据填充到模板中后生成的文件与其他用户上传的文件打包供审批人员下载，思路简单。但问题来了，利用freemaker将动态数据填充玩后会将流写入一个给定的OutputStream中，如果将此文件流存入物理硬盘，打包的时候再去下载。对物理硬盘的IO性能差不说，还占资源，果断否决。

### 方案一： ByteArrayOutputStream

此方案简单粗爆，如果遇到OutputStream写入的数据非常大的时间，就苦逼了，有内存溢出的风险，所以，勿必慎用。

<pre>
    public void testFrByteArray() throws Exception {
		// 业务对象,此处仅给空对象做示例,Freemarker将此model解析替换模板中的标签,生成业务文件.
		Object model = new Object();

		// 新建字节输出流,Freemarker操作此输出流写入生成的业务文件.
		ByteArrayOutputStream out = new ByteArrayOutputStream();

		// 自定义工具类,封装Freemarker操作细节
		FreemarkerUtils.renderTemplate(out, model, "template.xml");

		// 将outputstream转成inputstream
		ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray());

		FileOutputStream fos = new FileOutputStream("D:\\test\result.rar");

		ZipOutputStream zip = new ZipOutputStream(fos);

		// 生成压缩文件
		ZipUtils.add(zip, in, "template-res.doc", true);
	}
</pre>

### 方案二：PipedInputStream与PipedOutputStream

此方案有点高大上，但操作和理解起来有点难度。先看一下官方API的说明:

PipedInputStream，管道输入流应该连接到管道输出流；管道输入流提供要写入管道输出流的所有数据字节。通常，数据由某个线程从PipedInputStream 对象读取， 并由其他线程将其写入到相应的
PipedOutputStream。`不建议对这两个对象尝试使用单个线程，因为这样可能死锁线程。`管道输入流包含一个缓冲区，可在缓冲区限定的 范 围内将读操作和写操作分离开。如果向连接管道输出流提供数据字节的线程不再存在，则认为该管道已损坏。

红色字体部分很重要，因为很不巧笔者在这上面摔跟头了。

PipedOutputStream，可以将管道输出流连接到管道输入流来创建通信管道。管道输出流是管道的发送端。`通常，数据由某个线程写入PipedOutputStream 对象，并由其他线程从连接的PipedInputStream 读取。`不建议对这两个对象尝试使用单个线程，因为这样可能会造成该线程死锁。如果某个线程正从连接的管道输入流中读取数据字节，但该线程不再处于活动状态，则该管道被视为处于毁坏状态。

通过官方API的两段解释，大概能摸到这种实现方式的设计思路了：PipedOutputStream(生产者)生产数据，PipedInputStream(消费者)读取数据，二者同步进行。但谁也不能先挂掉，一旦挂掉，管道便处于损坏状态。

<pre>
	    public void testFrPiped() throws IOException {
    		// 业务对象,此处仅给空对象做示例,Freemarker将此model解析替换模板中的标签,生成业务文件.
    		final Object model = new Object();

    		FileOutputStream fos = new FileOutputStream("D:\\test\result.rar");

    		final ZipOutputStream zip = new ZipOutputStream(fos);

    		final PipedInputStream pis = new PipedInputStream();

    		final PipedOutputStream pos = new PipedOutputStream();

    		//将管道输出流连接到管道输入流来创建通信管道
    		pos.connect(pis);

    		//创建生产者线程来为管道输出流写入数据.
    		new Thread(new Runnable() {
    			public void run() {
    				try {
    					FreemarkerUtils.renderTemplate(pos, model,
    							"testtemplate.xml");
    				} catch (IOException e) {
    				} catch (TemplateException e) {
    				}
    			}
    		}).start();

    		//创建消费者线程来从 PipedInputStream对象读取数据
    		Thread customer = new Thread(new Runnable() {
    			public void run() {
    				try {
    					ZipUtils.add(zip, pis, "testtemplate.doc", true);
    				} catch (IOException e) {
    				}
    			}
    		});
    		customer.start();
    		try {
    			//勿必等所有数据读完才继续后面的操作,否则可能造成数据仍未读完便GameOver了.
    			customer.join();
    		} catch (InterruptedException e) {
    		}
    	}
</pre>