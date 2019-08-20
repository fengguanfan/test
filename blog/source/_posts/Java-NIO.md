---
title: Java_NIO
date: 2019-07-24 22:30:53
categories: 
    - Java
tags: 
    - NIO/IO
mp3: /musics/心如止水.mp3
cover: /images/NIO/封面.jpg
---
# NIO初识
1.IO是阻塞类型 文件的读写操作只能被程序执行一种，是一种单向操作。
2.NIO是非阻塞类型 Channels and Buffers (通道和缓存区) 组成，是一种双向操作。
3.NIO的文件和程序之间存在通道，通道中有缓存区，文件内容先到缓存区，再到程序。
## NIO的缓冲区
1.缓冲区有大小，用allocate设置
2.position   当前指针位置
3.limit      当前缓冲区已有的数据大小(首先要打开缓存区)
4.capacity   缓存区最大容量
5.mark(标记)和reset(重置到标记处)
# ByteBuffer操作实例

		import java.nio.ByteBuffer;
		class NIOBufferDemo {
			public test(){
				ByteBuffer buf = ByteBuffer.allocate(1024);
				
				System.out.println("初始化后信息：");
				System.out.println("position ="+buf.position()); --0
				System.out.println("limit = "+buf.limit()); --1024
				System.out.println("capacity = "+buf.capacity()); --1024
				
				--长度13
				buf.put("feizuisedeNIO".getBytes());
				System.out.println("输入内容后：");
				System.out.println("position ="+buf.position()); --13
				System.out.println("limit = "+buf.limit()); --1024
				System.out.println("capacity = "+buf.capacity()); --1024
				
				System.out.println("开启读之后：");
				buf.flip();
				System.out.println("position ="+buf.position()); --0
				System.out.println("limit = "+buf.limit()); --13
				System.out.println("capacity = "+buf.capacity()); --1024
				
				byte[] bytes = new byte[buf.limit()];
				buf.get(bytes);
				System.out.println("获取到的值"+new String(bytes,0,bytes.length));
				System.out.println("获取值之后：");
				System.out.println("position ="+buf.position()); --13
				System.out.println("limit = "+buf.limit()); --13
				System.out.println("capacity = "+buf.capacity()); --1024
				
				--重复读 指针重新跳到0
				buf.rewind()；
				buf.flip();
				
				--清空缓存区 注意clear方法并没有将缓存区的内容清空，只是将所有信息复原而已
				buf.clear();
				
				--标记和重置
				buf.get(bytes,0,2);
				buf.mark(); --标记指针为2
				buf.get(bytes,2,3);
				buf.reset(); --重置指针为2
				
			}
		} 

# NIO的直接缓存区和非直接缓存区
![非直接缓存区](/images/NIO/非直接缓存区.jpg)
直接缓存区是添加物理内存映射文件直接找到内容
![直接缓存区](/images/NIO/直接缓存区.jpg)

# 文件通道的使用(FileChannel)

		class FileChannelDemo{
			public test() throws IOException{
			
				--非直接缓存区
				FileChannel inChannel = FileChannel.open(Paths.get("D:\\1.png"),StandardOpenOption.READ);
				FileChannel outChannel = FileChannel.open(Paths.get("D:\\2.png"),StandardOpenOption.READ,StandardOpenOption.WRITE,StandardOpenOption.CREATE);
				--直接缓存区
				MappendByteBuffer inMap = inChannel.map(MapMode.READ_ONLY,0,inChannel.size());
				MappendByteBuffer outMap = outChannel.map(MapMode.READ_WRITE,0,inChannel.size());
				
				byte[] bytes = new byte[inMap.limit()];
				inMap.get(bytes);
				outMap.put(bytes);
				outMap.clear();
			}
		}

# 非直接缓存区和直接缓存区的区别
非直接缓存区

		while(inChannel.read(buf) != -1){
			buf.flip();
			outChannel.write(buf);
			buf.clear()
		}

直接缓存区

		inMap.get(bytes);
		outMap.put(bytes);
		outMap.clear();

# 分散读取和聚合写入操作
![分散读取和聚合写入](/images/NIO/分散读取和聚合写入.jpg)


 

