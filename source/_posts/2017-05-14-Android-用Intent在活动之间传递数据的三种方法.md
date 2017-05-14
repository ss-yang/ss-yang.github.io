---
title: Android-用Intent在活动之间传递数据的三种方法
date: 2017-05-14 17:16:18
tags:
- Android
categories:
- Android
---


### Intent

> intent n. 意图；目的；

Intent最常见的用法是启动活动、服务等，实现Activity间跳转。

那么问题来了，在跳转的时候，怎样给下一个Activity捎一些数据呢？

### 三种方法

- putExtra
- serializable
- parcelable

### - putExtra()

Intent类提供了putExtra()方法，可以向intent对象添加一些常见类型的附加数据。只要在startActivity之前调用即可。

    intent.putExtra("string_data","hello");

然后在另一边用getStringExtra()接收：

    getIntent.getStringExtra("string_data");

这种方法很简单，简单到只能传基本数据类型的数据，要传自定义的对象就无能为力了。

### - serializable

这是一个接口，serializable即序列化，能将对象转化成可以直接传输的数据，****可以理解为****将对象简单粗暴的用一串二进制表示，然后就可传输了。

用法非常简单，只要在要传输的类定义里面实现Serializable接口。

    public class Article implements Serializable{ …… }

putExtra()跟上面的方法一样，只是把put的对象换成自定义的对象了

    intent.putExtra("article",article);

再到下一个Activity取出传过来的序列化对象，就完成了。

    article = (Article) getIntent.getSerializbleExtra("article");

是不是超级简单粗暴！

但由于是直接传递整个对象，不可避免的会降低效率，尤其是传递体量较大的对象时。这个时候推荐用parcelable方法来实现。

### - parcelable

parcelable即包裹化，原理是先将对象拆开，再把真正有用的数据打包带走。

相比序列化的方法，parcelable实现了精准传递，很多时候我只需要对象的某些数据，并不是所有数据。也因为这种机制，这种方式实现起来较为复杂（拆开和打包需要自己动手）。

#### 1. 实现接口

        public class Article implements Parcelable{
        private int id;
        private String title;
        private String author;
        private String content;
        private String time;
        private String last_time;

#### 2. 先看看源码（Parcelable.java）

    public interface Parcelable {
        
        public static final int PARCELABLE_WRITE_RETURN_VALUE = 0x0001;

        public static final int PARCELABLE_ELIDE_DUPLICATES = 0x0002;

        public static final int CONTENTS_FILE_DESCRIPTOR = 0x0001;

        public int describeContents();
        
        public void writeToParcel(Parcel dest, int flags);

        public interface Creator<T> {        
            public T createFromParcel(Parcel source);
            public T[] newArray(int size);
        }

        public interface ClassLoaderCreator<T> extends Creator<T> {
            public T createFromParcel(Parcel source, ClassLoader loader);
        }
    }

这里只需要重写describeContents、writeToParcel和Creator

#### 3. 重写describeContents

通常，这里什么都不用做，直接返回0.

    @Override
    public int describeContents(){
        return 0;
    }

#### 4. 重写writeToParcel

通过writeXXX()方法写出对象的属性。XXX可以是各种受支持的数据类型。

    @Override
    public void writeToParcel(Parcel dest, int flag){
        dest.writeInt(id);
        dest.writeString(title);
        dest.writeString(author);
        dest.writeString(content);
        dest.writeString(time);
        dest.writeString(last_time);
    }

#### 5. Creator

从源码可以看出，public interface Creator<T> 是一个泛型接口，里面需要重写两个方法。

createFromParcel方法将打包的数据读取，重新封装成泛型指示的对象。

newArray方法返回一个对象数组。

    public static final Parcelable.Creator<Article> CREATOR = new Creator<Article>(){
        @Override
        public Article createFromParcel(Parcel source){
            Article article = new Article();
            article.setId(source.readInt());
            article.setTitle(source.readString());
            article.setAuthor(source.readString());
            article.setContent(source.readString());
            article.setTime(source.readString());
            article.setLast_time(source.readString());
            return article;
        }

        @Override
        public Article[] newArray(int size){
            return new Article[size];
        }
    };

注意这里读取数据的顺序应该和上一步写数据的****顺序一样****。

#### 6. 在下一个活动中接收数据

接受数据和前两种方法相差无几。用的是getParcelableExtra()方法。

---

### 总结

三种传数据的方法的思路都是一样的，都是A把数据put给intent，B从intent里面读取。不同的是实现的细节，第一种只能put一些基本数据类型，后两种方法进行了扩展，可以支持自定义类型。其中serializable直接将序列化后的对象传递，parcelable将对象的属性写入包裹，再从包裹重建对象，比前者效率高，但实现起来比前者复杂。

---

> 附：Intent putExtra支持的所有类型

    Intent 	putExtra(String name, String[] value)
    Intent 	putExtra(String name, long value)
    Intent 	putExtra(String name, boolean value)
    Intent 	putExtra(String name, double value)
    Intent 	putExtra(String name, Parcelable[] value)
    Intent 	putExtra(String name, char value)
    Intent 	putExtra(String name, int[] value)
    Intent 	putExtra(String name, int value)
    Intent 	putExtra(String name, double[] value)	 	 	 	
    Intent 	putExtra(String name, short value) 	 	 	 	 	
    Intent 	putExtra(String name, long[] value) 	 	 	 	 	
    Intent 	putExtra(String name, boolean[] value) 	 	 	 	 	
    Intent 	putExtra(String name, short[] value) 	 	 	 	 	
    Intent 	putExtra(String name, String value) 	 	 	 	 	
    Intent 	putExtra(String name, float[] value) 	 	 	 	 	
    Intent 	putExtra(String name, Bundle value) 	 	 	 	 	
    Intent 	putExtra(String name, byte[] value) 	 	 	 	 	
    Intent 	putExtra(String name, CharSequence value) 	 	 	 	 	
    Intent 	putExtra(String name, char[] value) 	 	 	 	 	
    Intent 	putExtra(String name, byte value) 	 	 	 	 	
    Intent 	putExtras(Intent src) 	 	 	 	 	
    Intent 	putExtras(Bundle extras)
    Intent 	putExtra(String name, Serializable value) 	 	 	 	 	
    Intent 	putExtra(String name, Parcelable value)