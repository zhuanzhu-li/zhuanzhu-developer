Java NIO 第一版

## 2 缓冲区



~~~mermaid
graph TB
	start[BUFFER] --> CharBuffer
	start[BUFFER] --> IntBuffer
	start[BUFFER] --> DoubleBuffer
	start[BUFFER] --> ShortBuffer
	start[BUFFER] --> LongBuffer
	start[BUFFER] --> FloatBuffer
	start[BUFFER] --> ByteBuffer
	


~~~

* 基本概念

  缓存区是包在一个对象内的基本数据元素数据

* 属性

  * 容量
  * 上界
  * 位置
  * 标记

* Api

  * public final int capacity()
  * public final int position()
  * public final Buffer position( int newPosition)
  * public final int limit()
  * public final Buffer limit(int newLinit)
  * public final Buffer mark()
  * public final Buffer reset()
  * public final Buffer clear()
  * public final Buffer flip()
  * public final Buffer rewind()
  * public final int remaining()
  * public final boolean hasRemaining()
  * public final boolean isReadOnly()
