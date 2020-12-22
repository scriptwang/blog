这只是一篇我自己学习java的流水账笔记，包含了以下内容，全是基础代码示例，希望对正在学习java的小伙伴有帮助

- 基础类型转换: The convertion of basic data types
- 字符串统计: String counting functions
- 冒泡排序，选择排序，二分法查找：bubbleSort , selectionSort & binarySearch
- 打印文件树：Listing files tree of a path typed from command line
- java流的基本操作：Java Input/Output stream
- java线程：Java Thread；Thread pool
- java网络编程：TCP/UDP programing : Say hello to server!
- java GUI编程：Graphical User Interface programming
- java GUI贪吃蛇：[Daily] Snake
- java简单爬虫：Web Email Spider & Magnet Spider
- java正则表达式：Regular Expression
- 双向链表：[Daily]Two-way loopback linked list
- java编码：编码相关
- java常用代码：Java常用代码示例
- jdbc示例：JDBC编程示例
- java基本算法：基本算法
- java设计模式：设计模式


## The convertion of basic data types
### char to int
When converting a char into an int , you`ll get the ASCII value of a character.
- Convert character to ASCII
```java
int ascii = (int)char;
```

- Convert ASCII to character
ascii is an int below
```java
char char = (char)ascii;
```

- Example code 
```java
public class CharToAscii{
    public static void main(String[] args){
        System.out.println(charToAscii('a'));//97
        System.out.println(asciiToChar(97));//a
    }
     
    public static int charToAscii(char c){
        int ascii = c;
        return ascii;
    }
     
    public static char asciiToChar(int ascii){
        char c = (char)ascii;
        return c;
    }
}
```


## String counting functions
### First String counting
How to count a string : How many uppercase letters , lowercase letters and other symbols ? Like this:
```java
String str = "AqSwDeFrGtgHJhhhKgfgf!@#$%^7898754";/* How many uppercase letters 
 
lowercase letters and other symbols here ? */
```
Here are 3 ways to do it:
- Function_1:There`s a tip about basic data convertion char to int

```java
public static void fun_1(String s){
 
    int lCount = 0, uCount = 0, oCount = 0;
     
    for (int i = 0;i < s.length();i++){
     
        char c = s.charAt(i);
         
        if (c >= 'a' && c <= 'z'){//char to ascii
         
            lCount++;
             
        }else if(c >= 'A' && c <= 'Z'){
         
            uCount++;
             
        } else{
         
            oCount++;
             
        };
                     
    }
    System.out.println
    ("There are " + lCount + "uppercase letters!" );
     
    System.out.println
    ("There are " + uCount + "lowercase letters!" );
     
    System.out.println
    ("There are " + oCount + "other symbols!" );
     
     
}
```

- Function_2:Class String has the indexOf() methods
```java
public static void fun_2(String s){
 
    String lower = "abcdefghijklmnopqrstuvwxyz";
     
    String upper = "ABCDEFGHIJKLMNOPKQSTUVWXYZ";
     
    int lCount = 0, uCount = 0, oCount = 0;
     
    for (int i = 0;i < s.length();i++){
     
        char c = s.charAt(i);
         
        if ( lower.indexOf(c) != -1 ){
         
            lCount++;
             
        }else if(upper.indexOf(c) != -1){
         
            uCount++;
             
        } else{
         
            oCount++;
        };
                     
    }
    System.out.println
    ("There are " + lCount + "uppercase letters!" );
     
    System.out.println
    ("There are " + lCount + "lowercase letters!" );
     
    System.out.println
    ("There are " + lCount + "other symbols!" );
}
```

- Function_3:Using the charAt() , Character.isUpperCase() and Character.isLowerCase() method
```java
public static void fun_3(String s){
    int lCount = 0, uCount = 0, oCount = 0;
    for (int i = 0;i < s.length();i++){
        char c = s.charAt(i);
        if (Character.isLowerCase(c)){
            lCount++;
        }else if(Character.isUpperCase(c)){
            uCount++;
        } else{
            oCount++;
        };
                 
    }
    System.out.println
    ("There are " + lCount + "uppercase letters!" );
    System.out.println
    ("There are " + lCount + "lowercase letters!" );
    System.out.println
    ("There are " + lCount + "other symbols!" );
}  
```

### Second String counting
How to count a string : How many String pointed here? Like this:
```java
String str = "javartyjojavayyuhujavatyuuhueuhdjavajiiuhdsuhjava";
 
String s   ="java"// How many s in str ?
```
Here are 2 ways to do it:
- Function_1: 3 methods here : substring() , equals() , indexOf()
```java
public static void fun_1(String str , String s){
    int Count = 0, restore = 0,index = 0;
    String s_start = str.substring(0,s.length());
    if(s_start.equals(s)){Count = 1;}
    while(index != -1 ){
        index = str.indexOf(s , s.length() + restore);
        if( index >= 0 ){ Count++; }
        restore = index;
    }
    System.out.println
    ("\""+ s + "\"" + " appears "+ Count +"times");
}
```

- Function_2: 2 methods here : substring() ,  indexOf()
```java
public static void fun_2(String str , String s){
    int Count = 0,index = -1;
    while( (index = str.indexOf(s)) != -1){
        str = str.substring(index + s.length());
        Count ++;
    }
     
    System.out.println
    ("\""+ s + "\"" + " appears "+ Count +"times");
}
```

## bubbleSort , selectionSort & binarySearch
The source code here
```java
public class Test{
    public static void main(String[] args){
         
        Date[] d = new Date[4];
        d[0] = new Date(1994,8,20); 
        d[1] = new Date(1992,7,12); 
        d[2] = new Date(2016,8,20); 
        d[3] = new Date(1990,6,23);
        Date ds = new Date(2016,8,20);
        System.out.println("Original:");
        print(d);
        System.out.println("class Date serached:"+ds);
        System.out.println();
        System.out.println("selectionSort below:");
        selectionSort(d);
        print(d);
        System.out.println("index searched below:");
        System.out.println
        (binarySearch(d,ds));
        System.out.println("bubbleSort below:");
        bubbleSort(d);
        print(d);
        System.out.println("index searched below:");
        System.out.println
        (binarySearch(d,ds));
         
    }
     
     
    public static void print(Date[] date){
        for(int i = 0;i < date.length;i++){
            System.out.println(date[i]);
        }
    }
     
     
    public static void selectionSort(Date[] date){
        Date temp;
        for (int i = 0;i < date.length ; i++ ){
            for (int j = i;j < date.length; j++ ){
                if( date[i].compare(date[j]) > 0 ){
                    temp = date[i];
                    date[i] = date[j];
                    date[j] = temp;
                }
            }
        }
         
    }
     
     
    public static void bubbleSort(Date[] date){
        Date temp;
        for (int i = date.length - 1;i > 0 ;i-- ){
            for (int j = 0;j < i;j++){
                if( date[j].compare(date[j+1]) > 0 ){
                    temp = date[j];
                    date[j] = date[j+1];
                    date[j+1] = temp;
                }
            }
        }
    }
     
     
    public static int binarySearch(Date[] date,Date d){
        if (date.length == 0){return -1;}
        int startPos = 0;
        int endPos = date.length - 1;
        int middlePos = (startPos + endPos)/2;
        while (startPos <= endPos ){
            if (d.compare(date[middlePos]) == 0){
                return middlePos;
            }
            if (d.compare(date[middlePos]) > 0){
                startPos = middlePos + 1;
            }
            if (d.compare(date[middlePos]) < 0){
                endPos = middlePos - 1; 
            }
            middlePos = (startPos + endPos)/2;
        }
        return -1;
         
    }
     
     
     
}
 
 
class Date{
    private int day,month,year;
    Date(int y,int m,int d){
        this.year = y;
        this.month = m;
        this.day = d;
    }
     
    public int compare(Date date){//ternary operator
        return
         year > date.year ? 1
        :year < date.year ? -1
        :month > date.month ? 1
        :month < date.month ? -1
        :day > date.day ? 1
        :day < date.day ? -1 : 0;
    }
     
    public String toString(){//class Object method toString()
        String s = new String(year+"-"+ month+"-"+ day);
        return s;
    }
     
}
```
## Listing files tree of a path typed from command line
```java
import java.io.*;
public class ListFilesTree{
    public static void main(String[] args){//reading a path from command line
        String path = args[0].replace(File.separatorChar,'/'); 
        //converting separator
         
        File file = new File(path);
        fileTree(file,0);
    }
     
    public static void fileTree(File f,int level){//level denotes the depth of path
         
        String preStr = "";
        for (int i = 0;i < level;i++){
            preStr += "---|";
        }
         
        File[] flist = f.listFiles();
        for (int i = 0;i < flist.length;i++){
            System.out.println(preStr + flist[i]);
            if( flist[i].isDirectory() ){
                fileTree(flist[i],level + 1);/*calling the 
                fileTree itself is a recursion 
                level increase 1 when calling fileTree everytime 
                */
            }
        }
    }
}
```
## Java Input/Output stream
- FileInputStream & FileOutputStream
```java
import java.io.*;
public class TestFileOutputStream{
    public static void main(String[] args){
        int b = 0;
        FileInputStream in = null;
        FileOutputStream out = null;
        String inPath = 
        "D:\\Important files\\JAVA\\practice\\IO\\TestFileOutputStream.java";
        String outPath = 
        "D:\\Important files\\JAVA\\practice\\IO\\CopyTestFileOutputStream.java";
        try{
        in = new FileInputStream(inPath.replace(File.separatorChar,'/'));
        out = new FileOutputStream(outPath.replace(File.separatorChar,'/'));
        while ( (b = in.read()) != -1){
            out.write(b);
            System.out.print((char)b);
        }
        in.close();out.close();
        }catch(FileNotFoundException e){
            System.out.println("File not found!");
            System.exit(-1);
        }catch (IOException e1){
            System.out.println("io error!");
            System.exit(-1);
        }
        System.out.println("文件已经复制");
    }
}
```
- FileReader & FileWriter
```java
import java.io.*;
public class TestFileReader{
    public static void main(String[] args){
        int c = 0;
        FileReader fr = null;
        FileWriter fw = null;
        String frPath = 
        "D:\\Important files\\JAVA\\practice\\IO\\TestFileReader.java";
        String fwPath = 
        "D:\\Important files\\JAVA\\practice\\IO\\copyTestFileReader.java";
        try{
            fr = new FileReader(frPath.replace(File.separatorChar,'/'));
            fw = new FileWriter(fwPath.replace(File.separatorChar,'/'));
            while ((c = fr.read()) != -1 ){
                fw.write(c);
                System.out.print((char)c);
            }
            fr.close();
            fw.close();
        }catch(FileNotFoundException e){
                 
                System.out.println("文件找不到！");
                System.exit(-1);
        }catch(IOException e1){
                System.out.println("文件错误！");
                System.exit(-1);
        }
        System.out.println("文件已复制！");
         
    }
}
```
```java
import java.io.*;
public class TestFileWriter{
    public static void main(String[] args){
        FileWriter fw = null;
        try{
            fw = new FileWriter
            ("D:\\Important files\\JAVA\\practice\\IO\\Unicode.dat");
            for (int c = 0;c <= 50000;c++){
                fw.write(c);
            }
            fw.close();
        }catch(FileNotFoundException e){
            System.out.println("文件找不到！");
            System.exit(-1);
        }catch(IOException e1){
            System.out.println("IO错误！");
            System.exit(-1);
        }
         
    }
}
```
- Buffered Stream
```java
import java.io.*;
public class TestBuffered{
    public static void main(String[] args){
        try{
            FileInputStream in = new FileInputStream
            ("D:\\Important files\\JAVA\\practice\\IO\\TestBuffered.java");
            BufferedInputStream bis = new BufferedInputStream(in);
            System.out.println((char)bis.read());
            System.out.println((char)bis.read());
            int c = 0;
            for(int i = 0;i <= 100 && (c = bis.read()) != -1;i++ ){
                System.out.print((char)c+"");
            }
        }catch(IOException e){e.printStackTrace();}
         
    }
}
```
```java
import java.io.*;
public class TestBufferWriter{
    public static void main(String[] args){
        try{
            BufferedWriter bw = new BufferedWriter
            (new FileWriter("D:\\Important files\\JAVA\\practice\\IO\\Test\\1.txt"));
            BufferedReader br = new BufferedReader
            (new FileReader("D:\\Important files\\JAVA\\practice\\IO\\Test\\1.txt"));
            String s = null;
            for(int i = 0;i < 10;i++){
                s = String.valueOf(Math.random());
                bw.write(s);
                bw.newLine();
            }
            bw.flush();
            while( (s = br.readLine()) != null){
                System.out.println(s);
            }
            bw.close();br.close();
        }catch(IOException e){e.printStackTrace();}
    }
}
```
- Converting Stream
```java
import java.io.*;
public class Transform{
    public static void main(String[] args){
        try{
            String path = 
            "D:\\Important files\\JAVA\\practice\\IO\\Test\\transform.txt";
            OutputStreamWriter osw = new OutputStreamWriter
            (new FileOutputStream(path));
            osw.write("output stream writer transforms the file output stream!");
            System.out.println(osw.getEncoding());
            osw.close();
            osw = new OutputStreamWriter
            ((new FileOutputStream(path,true)),"ISO8859_1");
            osw.write("output stream writer transforms the file output stream!");
            System.out.println(osw.getEncoding());
            osw.close();
        }catch(IOException e){
            e.printStackTrace();
        }
         
    }
}
```
```java
import java.io.*;
public class Transform1{
    public static void main(String[] args){
        InputStreamReader isr = new InputStreamReader(System.in);//阻塞式的方法
        BufferedReader br = new BufferedReader(isr);
        String s = null;
        try {
            s = br.readLine();
            while(s != null){
                if (s.equalsIgnoreCase("exit")){System.exit(-1);}
                System.out.println(s.toUpperCase());
                s = br.readLine();
            } 
            br.close();
             
        }catch(IOException e){
            e.printStackTrace();
        }
    }
}
```
- Data Stream
```java
import java.io.*;
public class DataStream{
    public static void main(String[] args){
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        DataOutputStream dos = new DataOutputStream(baos);
        try {
            dos.writeDouble(Math.random());
            dos.writeBoolean(true);
            ByteArrayInputStream bais = new ByteArrayInputStream
            (baos.toByteArray());
            System.out.println(bais.available());
            DataInputStream dis = new DataInputStream(bais);
            System.out.println(dis.readDouble());
            System.out.println(dis.readBoolean());
             
        }catch (IOException e){e.printStackTrace();}
    }
}
```
- Printing Stream
```java
import java.io.*;
public class TestPrintStream1{
    public static void main(String[] args){
        String f = args[0];
        if (f != null){
            list(f,System.out);
        }
    }
     
    public static void list(String f,PrintStream ps){
        try{
            BufferedReader br = new BufferedReader
            (new FileReader(f));
            while( br.readLine()!= null){
                ps.println(br.readLine());
            }
            br.close();
        }catch (IOException e){e.printStackTrace();}
    }
}
```
```java
import java.io.*;
import java.util.*;
public class TestPrintStream2{
    public static void main(String[] args){
        BufferedReader br = new BufferedReader
        (new InputStreamReader(System.in));
        try{
            String s = null;
            FileWriter fw = new FileWriter
            ("D:\\Important files\\JAVA\\practice\\IO\\Test\\log.txt",true);
            PrintWriter log = new PrintWriter(fw);
            while( (s = br.readLine()) != null){
                if(s.equalsIgnoreCase("exit")) break;
                System.out.println(s.toUpperCase());
                log.println("-------");
                log.println(s.toUpperCase());
                log.flush();
            }
            log.println("===="+new Date()+"=====");
            log.flush();
            log.close();
        }catch(IOException e){e.printStackTrace();}
    }
}
```
- Object Stream
```java
import java.io.*;
public class TestObjectStream{
    public static void main(String[] args){
        T t = new T();
        t.i = 8;
        try{
            FileOutputStream fos = new FileOutputStream
            ("D:\\Important files\\JAVA\\practice\\IO\\Test\\object.dat");
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(t);
            oos.flush();
            oos.close();
            FileInputStream fis = new FileInputStream
            ("D:\\Important files\\JAVA\\practice\\IO\\Test\\object.dat");
            ObjectInputStream ois = new ObjectInputStream(fis);
            T tread = (T)ois.readObject();
            System.out.println
            (tread.i+" "+tread.j+" "+tread.d+" "+tread.k);
        }catch(IOException e ){e.printStackTrace();
        }catch(ClassNotFoundException e){e.printStackTrace();}
    }
}
class T implements Serializable{
    int i = 10;
    int j = 9;
    double d = 0.12324;
    transient int k = 15;
}
```
## Java Thread
### Creating thread
```java
public class TestThread{
    public static void main(String[] args){
        Thread1 a = new Thread1();
        Thread t = new Thread(a);
        t.start();
        for(int i= 1;i <= 1000;i++){
            System.out.println("main : "+i);
        }
        System.out.println(t.isAlive());
    }
}
class Thread1 implements Runnable{
    public void run(){
        for(int i= 1;i <= 1000;i++){
            try{
                System.out.println("Thread1 : "+i);
                 
            }catch
            (InterruptedException e){e.printStackTrace();}
 
        }
    }
}
```
### Testing join()
```java
public class TestJoin{
    public static void main(String[] args){
        Runner1 r = new Runner1();
        Thread t = new Thread(r);
        t.start();
        try{
            t.join();
        }catch(InterruptedException e){}
 
        for(int i = 0;i<100;i++){
            System.out.println("I am main");
        }
    }
}
class Runner1 implements Runnable{
    public void run(){
        for(int i = 0;i < 100;i++){
            System.out.println("I am Runner1");
        }
    }
}
```

### Testing priority()
```java
public class TestPriority{
    public static void main(String[] args){
        R1 r1 = new R1();
        R2 r2 = new R2();
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        //t2.setPriority(Thread.NORM_PRIORITY + 5);
        t2.setPriority(Thread.NORM_PRIORITY - 4);
        t1.start();t2.start();
         
    }
}
 
class R1 implements Runnable{
    public void run(){
        for(int i = 0;i < 1000;i++){
            System.out.println("=======I am R1");
        }
    }
}
 
class R2 implements Runnable{
    public void run(){
        for(int i =0;i < 1000;i++){
            System.out.println("I am R2");
        }
    }
     
}
```
### Testing yield()
```java
public class TestYield{
    public static void main(String[] args){
        Runner2 r1 = new Runner2("t1=======");
        Runner2 r2 = new Runner2("t2");
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();t2.start();
         
    }
}
 
class Runner2 implements Runnable{
    private String name; 
    Runner2(String name){
        this.name = name;
    }
    public void run(){
        for(int i = 0;i < 1000;i++){
            System.out.println("I am "+ name +" : "+i);
            if(i % 10 == 0){Thread.yield();}
        }
    }
}
```
### Testing sleep()
```java
import java.util.*;
public class TestSleep{
    public static void main(String[] args){
        Runner r = new Runner(); 
        Thread t = new Thread(r);
        t.start();
        try {
            Thread.sleep(10000);
        }catch(InterruptedException e){e.printStackTrace();}
        t.interrupt();
        r.shutdown();
    }
}
class Runner implements Runnable{
    private boolean flag = true;
    public void run(){
        while(flag){
            System.out.println("====="+new Date()+"=====");
            try{
                Thread.sleep(1000);
            }catch(InterruptedException e){
                System.out.println("interrupted");
                //return;
            }
        }
    }
     
    public void shutdown(){
        flag = false;
    }
}
```
### Testing synchronization
```java
/*
key words:synchronized
*/
public class TestSynch1 implements Runnable{
    Timer time = new Timer();
    public static void main(String[] args){
        TestSynch1 t = new TestSynch1();
        Thread t1 = new Thread(t,"t1");
        Thread t2 = new Thread(t,"t2");
        t1.start();
        t2.start();
    }
     
    public void run(){
        time.add(Thread.currentThread().getName());
    }
}
 
class Timer{
    private static int num = 0;
    public synchronized void add(String s){
        //synchronized (this){
        num++;
        try{
            Thread.sleep(1);//         
        }catch(InterruptedException e){}
        System.out.println
        (s+"是第"+num+"个调用该线程的！");
        //}
    }
}
public class TestSynch2 implements Runnable{
    int b = 100;
    public static void main(String[] args){
        TestSynch2 test = new TestSynch2();
        Thread t = new Thread(test);
        t.start();
        test.m2();
        try{
            Thread.sleep(100);
        }catch(InterruptedException e){}
         
        System.out.println(test.b);
    }
     
    public void run(){
        m1();  
    }
         
    public synchronized void m1(){
        b = 1000;
        try{
            Thread.sleep(200);
        }catch(InterruptedException e){}
         
        System.out.println("m1 "+ b);
    }
     
    public synchronized  void m2(){
        /*
        try{
            Thread.sleep();
        }catch(InterruptedException e){}*/
         b = 2000;
         System.out.println("m2 " +b);
    }
}

```

### Testing dead lock
```java
public class TestDeadLock implements Runnable{
    private int flag = 0;
    static Object o1 = new Object();
    static Object o2 = new Object();  
     
    public static void main(String[] args){
         
        TestDeadLock deadLock1 = new TestDeadLock();
        deadLock1.flag = 1;
        TestDeadLock deadLock2 = new TestDeadLock();
        deadLock2.flag = 2;
        Thread t1 = new Thread(deadLock1);
        Thread t2 = new Thread(deadLock2);
        t1.start();t2.start();
 
    }
     
    public void run(){
        System.out.println(flag);
        if (flag == 1){
            synchronized (o1) {
                try{
                    Thread.sleep(500);
                }catch(InterruptedException e){}
                synchronized (o2){
                    System.out.println("t1");
                }
            }
             
        }
         
        if(flag == 2){
            synchronized (o2) {
                try{
                    Thread.sleep(500);
                }catch(InterruptedException e){}
                synchronized (o1){
                    System.out.println("t2");
                }
            }
 
        }
    }
}
```

### Testing producer and consumer
```JAVA
public class Test1{
    public static void main(String[] args){
        /*
        SyncStack ss = new SyncStack();
        Wotou wt = new Wotou(0);
        ss.push(wt);
        Producer p = new Producer();
        p.add();
        Consumer c = new Consumer();
        c.subtract();*/
        SyncStack ss = new SyncStack();
        Producer p = new Producer(ss);
        Consumer c = new Consumer(ss);
        Thread t1 = new Thread(p);
        Thread t2 = new Thread(c);
        t1.start();t2.start();
    }
}
 
class Wotou{
    private int id;
    Wotou(int id ){
        this.id = id;
    }
    public String toString(){
        return "wotou : " + id;
    }
}
 
class SyncStack{
    private int index;
    Wotou[] arrwt = new Wotou[6];
     
    public synchronized void push(Wotou wt){
        while(index == arrwt.length) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.notifyAll();      
        if ( index == arrwt.length ){}
        arrwt[index] = wt;
        index++;
        System.out.println( "Producing " + index+" wt");
    }
     
    public synchronized Wotou pop(){
        while(index == 0) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.notifyAll();
        //arrwt[index] = wt;
        index--;
        System.out.println( "Consuming " + index+" wt");
        return arrwt[index];
    }
}
 
 
class Producer implements Runnable{
    SyncStack ss = null;
    Producer(SyncStack ss){
        this.ss = ss;
    }
    public void run(){
        for(int i=0; i<20; i++) {
            Wotou wt = new Wotou(i);
            ss.push(wt);
            try {
                Thread.sleep((int)(Math.random() * 200));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }          
             
        }
    }
}
 
class Consumer implements Runnable{
    SyncStack ss = null;
    Consumer(SyncStack ss){
        this.ss = ss;
    }
    public void run(){
        for(int i=0; i<20; i++) {
            Wotou wt = ss.pop();
             
            try {
                Thread.sleep((int)(Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }      
             
        }
    }
}
```

## TCP/UDP programing : Say hello to server!
### TCP Server
```java
import java.net.*;
public class TestServerSocket {
    public static void main(String[] args) throws Exception{
        ServerSocket ss = new ServerSocket(6666);
        while(true){
            Socket s = ss.accept();
            System.out.println("A client connect!");
        }
    }
}
```
### TCP Client
```java
import java.net.*;
import java.io.*;
public class TestSocket {
    public static void main(String[] args) throws Exception {
        Socket s = new Socket("45.32.18.35",6666);
        DataOutputStream dos = new DataOutputStream(s.getOutputStream());
        dos.writeUTF("Hello server");
        System.out.println(s.getLocalPort());
        dos.flush();dos.close();s.close();
    }
}
```

### TCP Server1
```java
import java.net.*;
import java.io.*;
public class TestServer{
    public static void main(String[] args){
      try{
         ServerSocket ss = new ServerSocket(6666);
         while (true){
            Socket s = ss.accept();
            DataOutputStream dos = new DataOutputStream
            ( s.getOutputStream());
            dos.writeUTF
            ("IP address: " + s.getInetAddress()+" "+"Port :" +s.getPort() );
            dos.flush();
            dos.close();
            s.close();
         }
        }catch(IOException e){System.out.println("A error has occured!");}
 
    }
 
}
```
### TCP Client1
```java
import java.io.*;
import java.net.*;
public class TestClient{
    public static void main(String[] args){
        try {
            Socket s = new Socket("45.32.18.35",6666);
            DataInputStream dis = new DataInputStream(s.getInputStream());
            System.out.println(dis.readUTF());
            dis.close();
            s.close();
        } catch (IOException e){
            System.out.println("服务器连接失败！");
        }
         
    }
}
```
### TCP Server2

```java
import java.io.*;
import java.net.*;
public class TestServer1{
    public static void main(String[] args){
        String str = null;
        DataInputStream dis = null;
        DataOutputStream dos = null;
        try{
            ServerSocket ss = new ServerSocket(6666);
            while(true) {
                Socket s = ss.accept();
                dis = new DataInputStream(s.getInputStream());
                if ((str = dis.readUTF()) != null) {
                    System.out.println(str);
                    System.out.println("from host :"+ s.getInetAddress());
                    System.out.println("port : " +s.getPort());
                    System.out.println();
                }
                dos = new DataOutputStream(s.getOutputStream());
                dos.writeUTF("hello client!");
                dos.flush();
                dos.close();
                dis.close();
                s.close();
            } 
        }catch (IOException e) {
                e.printStackTrace();
        }
    }
}
```
### TCP Client2

```java
import java.io.*;
import java.net.*;
public class TestSocket1{
    public static void main(String[] args){
        String str = null;
        DataOutputStream dos = null;
        DataInputStream dis = null;
        try {
            Socket s = new Socket("45.32.18.35",6666);
            dos = new DataOutputStream(s.getOutputStream());
            dos.writeUTF("hello server !");
            dos.flush();
             
            dis = new DataInputStream(s.getInputStream());
            if ( ( str = dis.readUTF() ) != null ) {
                System.out.println(str);
                System.out.println(s.getInetAddress());
                System.out.println(s.getPort());
                System.out.println();
            }
            dos.close();
            dis.close();
            s.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### A peer-2-peer talking instance TCP server
```java
import java.io.*;
import java.net.*;
public class TalkServer{
    public static void main(String[] args){
        String read = null;
        String inWrite = null;
        DataInputStream dis = null;
        DataOutputStream dos = null;
        InputStreamReader isr = new InputStreamReader(System.in);
        BufferedReader br = new BufferedReader(isr);
        try {
            ServerSocket ss = new ServerSocket(6666);
            Socket s = ss.accept();
            dis = new DataInputStream(s.getInputStream());
            dos = new DataOutputStream(s.getOutputStream());
            //read = dis.readUTF();
            //inWrite =  br.readLine();
            System.out.println("Client say : " + dis.readUTF());
            inWrite = br.readLine();
            while( !inWrite.equalsIgnoreCase("exit") ) {
                 
                dos.writeUTF(inWrite);
                dos.flush();
                System.out.println("Server say : " + inWrite);
                System.out.println("Client say : " + dis.readUTF());
                inWrite = br.readLine();
            }
            dis.close();
            dos.close();
            s.close();
            System.exit(-1);
        }catch (IOException e) {e.printStackTrace();}
    }
}
```
### A peer-2-peer talking instance TCP client
```java
import java.io.*;
import java.net.*;
public class TalkClient{
    public static void main(String[] args){
        String inWrite = null;
        String read = null;
        DataOutputStream dos = null;
        DataInputStream dis = null;
        InputStreamReader isr = new InputStreamReader(System.in);
        BufferedReader br = new BufferedReader(isr);
        try {
                Socket s = new Socket("127.0.0.1",6666);
                dos = new DataOutputStream(s.getOutputStream());
                dis = new DataInputStream(s.getInputStream());
                inWrite = br.readLine();
                while( !inWrite.equalsIgnoreCase("exit") ){
                    dos.writeUTF(inWrite);
                    dos.flush();
                    System.out.println("Client say : " +inWrite);
                    System.out.println("Server say : " + dis.readUTF());
                    inWrite = br.readLine();
                }
                dos.close();
                dis.close();
                s.close();
                System.exit(-1);
        }catch(IOException e){e.printStackTrace();}
    }
}
```
### UDP Server
```java
import java.net.*;
public class TestUDPServer1{
    public static void main(String[] args){
        byte[] buf = new byte[1024];
        DatagramPacket dp = new DatagramPacket(buf,buf.length);
         
        try{
            DatagramSocket ds = new DatagramSocket(6666); 
            while(true) {
                ds.receive(dp);
                System.out.println( new String(buf,0,dp.getLength()) );
            }
        }catch ( Exception e) {
            e.printStackTrace();
        }
    }
}
```
### UDP Client
```java
import java.net.*;
public class TestUDPClient1{
    public static void main(String[] args){
        byte[] buf = (new String("Hello Server!")).getBytes();
        DatagramPacket dp = new DatagramPacket
        (buf,buf.length,new InetSocketAddress("127.0.0.1",6666));
        try {
            DatagramSocket ds = new DatagramSocket(7777);
            ds.send(dp);
            ds.close();
        }catch ( Exception e) {
            e.printStackTrace();
        }
    }
}
```

### UDP Server1

```java
import java.io.*;
import java.net.*;
public class TestUDPServer2{
    public static void main(String[] args){
        byte[] buf = new byte[1024];
        DatagramPacket dp = new DatagramPacket
        (buf,buf.length);
        try{
            DatagramSocket ds = new DatagramSocket(6666);
            while(true){
                ds.receive(dp);
                ByteArrayInputStream bais = 
                new ByteArrayInputStream(buf);
                DataInputStream dis = 
                new DataInputStream(bais);
                System.out.println(dis.readLong());
            }
        }catch(Exception e){e.printStackTrace();}
    }
}

```

### UDP Client1
```java
import java.net.*;
import java.io.*;
public class TestUDPClient2{
    public static void main(String[] args){
        long n = 10000L;
        byte[] buf = null;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        DataOutputStream dos = new DataOutputStream(baos);
        //
        try{
            dos.writeLong(n);
        }catch (IOException e){e.printStackTrace();}
        buf =  baos.toByteArray();//important!
        DatagramPacket dp = new DatagramPacket
        (buf,buf.length,new InetSocketAddress("127.0.0.1",6666));
         
        try {
            DatagramSocket ds = new DatagramSocket(7777);
            ds.send(dp);
            ds.close();
        }catch (Exception e){e.printStackTrace();}
        try{
            dos.close();
            baos.close();
        }catch(IOException e){e.printStackTrace();}
    }
}
```

## classes InetAddress & NetworkInterface
### InetAddress
```java
import java.net.*;
import java.io.*;
public class Test {
    public static void main(String[] args) {
         
        getByHost();
        System.out.println();
        getHostByIP();
        System.out.println();
        getAllIP();
        System.out.println();
        getLocal();
        System.out.println();
        createAddress();
        System.out.println();
        getByteIP();
        System.out.println();
        Reachable();
        System.out.println();
        addressEquals();
    }
     
    public static void getByHost(){
        String hostName = "www.scriptwang.com";
        try{
            InetAddress address = InetAddress.getByName
            (hostName);
            System.out.println(address);
        } catch (UnknownHostException ex) {
            System.out.println("Can not find " + hostName);
        }
    }
     
    public static void getHostByIP(){
        String IP = "45.32.18.35";
        try {
            InetAddress address = InetAddress.getByName
            (IP);
            System.out.println(address.getHostName());
            System.out.println(address.getCanonicalHostName());
        } catch (UnknownHostException ex) {
            System.out.println
            ("Can not find host name by " + IP);
        }
    } 
     
    public static void getAllIP() {
        String hostName = "www.baidu.com";
        try{
            InetAddress[] addresses = 
                    InetAddress.getAllByName(hostName);
            for(InetAddress address : addresses) {
                System.out.println(address);
            } 
        }catch (UnknownHostException ex) {
                System.out.println("Can not find " + hostName);
        }
    }
     
    public static void getLocal() {
        try {
            InetAddress address = InetAddress.getLocalHost();
            String str = address.getHostAddress();
            System.out.println(address);
            System.out.println(str);
        } catch (UnknownHostException ex){
            System.out.println
            ("Can not find this computer`s address!");
        }
    }
     
    public static void createAddress(){
        byte[] IP = {107,23,(byte)216,(byte)196};
        try {
            InetAddress address1 = 
                    InetAddress.getByAddress(IP);
            InetAddress address2 = 
                    InetAddress.getByAddress("www.123.com",IP);
            System.out.println(address1);
            System.out.println(address2);
        } catch (UnknownHostException ex) {
            ex.printStackTrace();
        }
         
    }
     
    public static void getByteIP() {
        String host = "www.scriptwang.com";
        try {
            InetAddress address = InetAddress.getByName
            (host);
            byte[] IPArray = address.getAddress();
            //print IPArray
            for(byte ip : IPArray){
                System.out.println(ip);
            }
              
            //check IPv4 or IPv6
            if(IPArray.length == 4) 
                System.out.println("IPv4");
            if(IPArray.length == 16)
                System.out.println("IPv6");
             
        } catch (UnknownHostException ex ) {
            System.out.println();
        }
         
    }
     
    public static void Reachable(){
        String IP = "45.32.18.35";
        try{
            InetAddress address = 
                    InetAddress.getByName(IP);
            if(address.isReachable(500)){
                System.out.println
                (IP + " is reachable!");
            }else{
                System.out.println
                (IP + "  is not reachable!");
            }
        }catch(UnknownHostException ex){
            System.out.println
            ("Can not find " + IP);
        }catch ( IOException e){}
    }
     
    public static void addressEquals(){
        //equals returns true only the same IP! 
        String host1 = "www.scriptwang.com";
        String host2 = "img.scriptwang.com";
        try{
            InetAddress address1 =
                    InetAddress.getByName(host1);
            InetAddress address2 = 
                    InetAddress.getByName(host2);
            if(address1.equals(address2)){
                System.out.println
                (address1 + " is the same as " + address2);
            }else{
                System.out.println
                (address1 + " is the not same as " + address2);
            }
        }catch(UnknownHostException ex){
            System.out.println
            ("Host lookup failed!");
        }
          
         
    }
     
     
}


import java.net.*;
public class TestIP{
    public static void main(String[] args){
        try {
            InetAddress address =
                    InetAddress.getByName(args[0]);
            if(address.isAnyLocalAddress()){
                System.out.println
                (address + " is a wildcard address!");
            }
            if(address.isLoopbackAddress()){
                System.out.println
                (address + " is a loop back address!");
            }
            if(address.isLinkLocalAddress()){
                System.out.println
                (address + " is a link-local address!");
            }
            if(address.isSiteLocalAddress()){
                System.out.println
                (address + " is a site-local address!");
            }else{
                System.out.println
                (address + " is a global address!");
            }
             
            if(address.isMulticastAddress()){
                if(address.isMCGlobal()){
                    System.out.println
                    (address + " is a global multicast address!");
                }else if(address.isMCOrgLocal()){
                        System.out.println
                        (address + " is an organization multicast address!");
                }else if(address.isMCSiteLocal()){
                    System.out.println
                    (address + " is a site multicast address!");
                }else if(address.isMCLinkLocal()){
                    System.out.println
                    (address + " is a subnet multicast address!");
                }else if(address.isMCNodeLocal()){
                    System.out.println
                    (address + " is an interface-local multicast address!");
                }else{
                    System.out.println
                    (address + " is a unknown multicast address type!");
                }
                 
            }else{
                System.out.println
                    (address + " is a unicast address!");
            }
        } catch (UnknownHostException ex){
            System.out.println
                    ("Could not resolve " + args[0]);
        }
    }
}

```

### NetworkInterface

```java
import java.net.*;
import java.util.*;
public class TestNetworkInterface{
    public static void main(String[] args){
        test1();
        System.out.println();
        test2();
        System.out.println();
        //InterfaceLister();
        System.out.println();
        interfaceIPLister();
    }
     
    public static void test1(){
        try{
            NetworkInterface ni =
                    NetworkInterface.getByName
                    ("eth0");
            System.out.println
            (ni);
        }catch(SocketException e){
             
        }
    }
    public static void test2(){
        String IP = "127.0.0.1";
        try {
            InetAddress address = 
                    InetAddress.getByName(IP);
            NetworkInterface ni = 
                    NetworkInterface.getByInetAddress
                    (address);
            System.out.println(ni);
        }catch(SocketException e){
            System.err.println();
        }catch(UnknownHostException e1){
            System.err.println();
        }
    }
     
    public static void InterfaceLister(){
        try {
            Enumeration<NetworkInterface> interfaces = 
                    NetworkInterface.getNetworkInterfaces();
            while(interfaces.hasMoreElements()){
                NetworkInterface ni = 
                        interfaces.nextElement();
                System.out.println(ni);
            }
             
        }catch(SocketException e){
             
        }
    }
     
    public static void interfaceIPLister(){
        try{
            Enumeration<NetworkInterface> interfaces = 
                    NetworkInterface.getNetworkInterfaces();
            while(interfaces.hasMoreElements()){
                NetworkInterface ni =
                        interfaces.nextElement();
System.out.println(ni);
                Enumeration e = ni.getInetAddresses();
                while(e.hasMoreElements()){
                    System.out.println
                    (ni.getName()+" -->>InetAddress : "+e.nextElement());
                }
                System.out.println();
            }
        }catch(SocketException e){}
         
    }
     
}

```
### SpamCheck
```java

import java.net.*;
public class SpamCheck{
    public static final String BLACKHOLE = 
                                "sbl.spamhaus.org";
    public static void main(String[] args){
        for(String arg:args){
            if(isSpammer(arg)){
                System.out.println(arg + " is a spammer!");
            }else{
                System.out.println(arg + " is not a spammer!");
            }
        }
         
    }
    public static boolean isSpammer(String IP){
        try{
            InetAddress address = 
                    InetAddress.getByName(IP);
            byte[] ipArray = address.getAddress();
            String query = BLACKHOLE;
            for(byte b : ipArray){
                int unsignedByte = 
                    b < 0 ? b + 256 : b;
                query = unsignedByte + "." + query;
//System.out.println(query);
            }
            InetAddress.getByName(query);
            return true;
        }catch(UnknownHostException e){
            return false;
        }
    }
}
```
### WebLog 
```java
import java.io.*;
import java.net.*;
public class WebLog{
    public static void main(String[] args){
        try(FileInputStream fis = new FileInputStream(args[0]);
            Reader r = new InputStreamReader(fis);
            BufferedReader br = new BufferedReader(r);){
             
            for(String entry = br.readLine();
                entry!=null;entry = br.readLine()){
                    int index = entry.indexOf(" ");
                    String IP = entry.substring(0,index);
                    String rest = entry.substring(index);
                    try{
                            InetAddress address = 
                                    InetAddress.getByName(IP);
                            String host = address.getHostName();
                            System.out.println
                            (host +" "+rest);
                    }catch(UnknownHostException e){
                 
                    }
            }
        }catch(IOException e){
             
        }
    }
}
import java.io.*;
import java.net.*;
import java.util.*;
import java.util.concurrent.Callable;
import java.util.concurrent.*;
public class WebLogThread{
    private final static int NUM_THREADS = 4;
     
    public static void main(String[] args)throws IOException{
        ExecutorService executor = 
                Executors.newFixedThreadPool(NUM_THREADS);
        Queue<LogEntry> results = new LinkedList<LogEntry>();
 
        try( BufferedReader br = 
            new BufferedReader(
            new InputStreamReader( 
            new FileInputStream(args[0]), "UTF-8"))){
             
            for(String entry = br.readLine();
                entry != null;
                entry = br.readLine()){
                    LookupTask task = new LookupTask(entry);
                    Future<String> future = executor.submit(task);
                    LogEntry result = new LogEntry(entry,future);
                    results.add(result);
                }
        }
         
        for(LogEntry result : results){
            try{
                System.out.println(result.future.get());
            }catch( InterruptedException | ExecutionException ex ){
                System.out.println(result.original);
            }
        }
    }
     
    private static class LogEntry{
        String original;
        Future<String> future;
         
        LogEntry(String original,Future<String> future){
            this.original = original;
            this.future = future;
        }
    }
}
 
class LookupTask implements Callable<String>{
    private String line;
     
    public LookupTask(String line){
        this.line = line;
    }
     
    //@override
    public String call(){
        try{
            int index = line.indexOf(" ");
            String IP = line.substring(0,index);
            String rest = line.substring(index);
            String hostName = 
                    InetAddress.getByName(IP).getHostName();
            return hostName + " " + rest;
        }catch(Exception e){}
        return line;
    }
}
```
## Thread pool
```java
Extends class Thread for a thread

Extends class Thread for a thread or implements interface Runnable

import java.io.*;
import java.security.*;
import javax.xml.bind.*;
public class ExtendsThread{
    public static void main(String[] args){
        for(String filename : args){
            DigestThread dt = 
                    new DigestThread(filename);
            Thread t = new Thread(dt);
            t.start();
        }
    }
}
 
class DigestThread extends Thread{
    private String filename;
     
    DigestThread(String filename){
        this.filename = filename;
    }
     
    public void run(){
        try{
            FileInputStream fis = 
                    new FileInputStream(filename);
            MessageDigest sha = 
                    MessageDigest.getInstance("SHA-256");
            DigestInputStream dis = 
                    new DigestInputStream(fis,sha);
            while(dis.read() != -1);
            dis.close();
            byte[] digest = sha.digest();
            for(int i=0;i<digest.length;i++){
System.out.println(digest[i]);             
            }
 
            StringBuilder result = 
                    new StringBuilder(filename);
System.out.println(result);
            result.append(":");
System.out.println(result);
            result.append  
            (DatatypeConverter.printHexBinary(digest));
            System.out.println(result);
        }catch(IOException | NoSuchAlgorithmException  e){
             
        }
         
    }
     
}


```
### Call back functions for a thread including static call back and instance call back

```java

import java.io.*;
import java.security.*;
import javax.xml.bind.*;
public class CallbackDigestUserInterface{
    public static void main(String[] args){
        for(String filename : args){
            CallbackDigest cb = 
                    new CallbackDigest(filename);
            Thread t = new Thread(cb);
            t.start();
        }
         
    }
     
    public static void receiveDigest
    (byte[] digest,String name){
        StringBuilder result = 
                new StringBuilder(name);
        result.append(":");
        result.append
        (DatatypeConverter.printHexBinary());
        System.out.println(result);
    }
}
 
class CallbackDigest implements Runnable{
    private String filename;
     
    CallbackDigest(String filename){
        this.filename = filename;
    }
     
    public void run(){
        try {
            FileInputStream fis =
                    new FileInputStream(filename);
            MessageDigest sha = 
                    MessageDigest.getInstance("SHA-256");
            DigestInputStream dt = 
                    new DigestInputStream(fis,sha);
            while(dt.read != -1);
            dt.close();
            byte[] digest = sha.digest();
            CallbackDigestUserInterface.receiveDigest
            (digest,filename);
        }catch(IOException | NoSuchAlgorithmException e){
            System.err.printlln(e);
        }
    }
}
```
### Instance call back
```java
import java.io.*;
import java.security.*;
import javax.xml.bind.*;
public class InstanceCallbackDigestUserInterface{
    private String filename;
    private byte[] digest;
     
    InstanceCallbackDigestUserInterface(String filename){
        this.filename = filename;
    }
     
    public static void main(String[] args){
        for(String filename : args){
            InstanceCallbackDigestUserInterface d =
                new InstanceCallbackDigestUserInterface(filename);
            d.calculateDigest();
        }
    }
     
    public void calculateDigest(){
        InstanceCallbackDigest icd = 
                new InstanceCallbackDigest(filename,this);
        Thread t = new Thread(icd);
        t.start();
    }
     
    void receiveDigest(byte[] digest){
        this.digest = digest;
        System.out.println(this);
    }
     
    public String toString(){
        String result = filename + ":";
        if(digest != null){
            result += DatatypeConverter.printHexBinary(digest);
        }else{
            result += "digest not available!";
        }
        return result;
    }
     
     
}
 
class InstanceCallbackDigest implements Runnable{
    private String filename;
    private InstanceCallbackDigestUserInterface callback;
     
    InstanceCallbackDigest(String filename,
    InstanceCallbackDigestUserInterface callback){
        this.filename = filename;
        this.callback = callback;
    }
     
    public void run(){
        try(FileInputStream fis = 
        new FileInputStream(filename);
        ){
            MessageDigest sha = 
                    MessageDigest.getInstance("SHA-256");
            DigestInputStream dis = 
                    new DigestInputStream(fis,sha);
            while(dis.read() != -1);
            dis.close();
            byte[] digest = sha.digest();
            callback.receiveDigest(digest);
        }catch(IOException | NoSuchAlgorithmException e){
            System.err.println(e);
        }
    }
}
```
### A interesting instance
```java
interface DoWork{
    void doHomeWork(String homework,String anwser);
}
public class StudentCallback implements DoWork{
     
    public void doHomeWork(String homework,String anwser){
        System.out.println
        (homework+" the anwser is "+anwser+" thread线程");
    }
     
    public void goHome(){
        System.out.println
        ("I go home:主线程,不等thread线程出结果,无阻塞");
    }
     
    public void ask(String homework,RoomMate roommate){
        new Thread(new Runnable(){
            public void run(){
                roommate.getAnwser
                (homework,StudentCallback.this);
            }
        }).start();
         
        goHome();
    }
     
    public static void main(String[] args){
        StudentCallback student = new StudentCallback();
        String homework = "1+1=?";
        student.ask(homework,new RoomMate());
    }
}
 
class RoomMate{
    public void getAnwser(String homework,DoWork dowork){
        try{
            Thread.sleep(3000);
        }catch(InterruptedException e){
            System.err.println(e);
        }
        if(homework.equals("1+1=?")){
            String anwser = "2";
            dowork.doHomeWork(homework,anwser);
        }else{
            System.out.println("I do not know");
        }
         
    }
}
```
### Max finder
```java

import java.util.concurrent.*;
//import java.util.concurrent.Callable;
public class MultithreadedMaxFinder{
    public static void main(String[] args){
        /*
        System.out.println(Integer.MIN_VALUE);
        int[] i = {1,2,3,4,5};
        FindMaxTask m = new FindMaxTask(i,0,5);
        System.out.println(m.call());*/
        int[] data = new int[101];
        for(int i=0; i<100; i++){
            data[i] = i;
        }
         
        try{
            System.out.println
            (max(data));
        }catch(Exception e){}
         
         
         
    }
     
    public static int max(int[] data) throws InterruptedException,
    ExecutionException{
         
        if(data.length == 1){
            return data[0];
        }else if(data.length == 0){
            System.err.println();
        }
         
        FindMaxTask task1 = 
                new FindMaxTask(data,0,data.length/2);
        FindMaxTask task2 = 
                new FindMaxTask(data,data.length/2,data.length);
                 
        ExecutorService service = 
                Executors.newFixedThreadPool(2);
        Future<Integer> future1 = service.submit(task1);
        Future<Integer> future2 = service.submit(task2);
         
        return Math.max(future1.get(),future2.get());
             
    }
}
 
class FindMaxTask implements Callable<Integer>{
    private int[] data;
    private int start;
    private int end;
     
    FindMaxTask(int[] data,int start,int end){
        this.data = data;
        this.start = start;
        this.end = end;
    }
     
    public Integer call(){
        int max = Integer.MIN_VALUE;
        for(int i=start; i<end; i++){
            if(data[i] > max) max = data[i];
        }
        return max;
    }
}

```
### Gzip files
```java

import java.io.*;
import java.util.zip.*;
import java.util.concurrent.*;
public class GzipAllfiles{
     
    public final static int THREAD_COUNT = 4;
     
    public static void main(String[] args){
         
        ExecutorService pool = 
                Executors.newFixedThreadPool(THREAD_COUNT);
         
        for(String filename : args){
            File f = new File(filename);
            if(f.exists()){
                if(f.isDirectory()){
                    File[] files = f.listFiles();
                    for(int i=0; i<files.length; i++){
                        if(!files[i].isDirectory()){
                            Runnable task = 
                                    new GzipRunnable(files[i]);
                            pool.submit(task);
                        }
                    }
                }else{
                    Runnable task = 
                            new GzipRunnable(f);
                    pool.submit(task);
                }
            }
        }
        pool.shutdown();
    }
}
 
class GzipRunnable implements Runnable{
     
    private final File input;
     
    public GzipRunnable(File input){
        this.input = input;
    }
     
    public void run(){
        if(!input.getName().endsWith(".gz")){
            File output = new File
                (input.getParent(),input.getName() + ".gz");
            if(!output.exists()){
                try(InputStream in = 
                    new BufferedInputStream(
                    new FileInputStream(input));
                    OutputStream out = 
                    new BufferedOutputStream(
                    new GZIPOutputStream(
                    new FileOutputStream(output)))){
                        int b;
                        while( (b = in.read()) != -1 ){
                            out.write(b);
                        }
                        out.flush();
                }catch(IOException e){
                    System.out.println(e);
                }
            }
        }
    }
}
```
### A simple instance
```java
import java.util.concurrent.*;
import java.util.*;
public class ThreadPool{
    public static void main(String[] args){
        ExecutorService pool = 
                Executors.newFixedThreadPool(4);
         
        List<Future<String>> futures = 
                new ArrayList<Future<String>>(10);
        for(int i=0; i<10; i++){
            futures.add
            (pool.submit(new ThreadIns()));   
        }
        for(Future<String> future : futures){
            try{
                String result = future.get();
                System.out.println(result);
            }catch(InterruptedException | ExecutionException e){
                 
            }
             
        }
        pool.shutdown();
    }
}
 
class ThreadIns implements Callable<String>{
     
    public String call(){
        return "Run";
    }
}
```
## Graphical User Interface programming
### Frame and Panel
```java
import java.awt.*;
public class TestFrame2{
    public static void main(String[] args){
        frame f1 = new frame
        (100,100,100,100,Color.blue);
        frame f2 = new frame
        (200,100,100,100,Color.black);
        frame f3 = new frame
        (100,200,100,100,Color.yellow);
        frame f4 = new frame
        (200,200,100,100,Color.green);
    }
}
 
class frame extends Frame{
    static int id = 0;
    frame(int x,int y,int w,int h,Color c){
        super("Tittle"+(++id));
        setBounds(x,y,w,h);
        setBackground(c);
        setVisible(true);
    }
}
import java.awt.*;
public class TestFrame3{
    public static void main(String[] args){
        Frame f = new Frame("Test");
        Panel p = new Panel(null);
        f.setLayout(null);
        f.setBounds(50,100,300,300);
        f.setBackground(Color.blue);
        p.setBounds(50,50,200,200);
        p.setBackground(Color.yellow);
        f.add(p);
        f.setVisible(true);
    }
}
import java.awt.*;
public class TestFrame4{
    public static void main(String[] args){
        MyPanel p = new MyPanel
        ("test",800,30,400,400);
    }
}
 
class MyPanel extends Frame{
    private Panel p1,p2,p3,p4;
    public MyPanel(String n,int x,int y,int w,int h){
        super(n);
        setLayout(null);
        setBounds(x,y,w,h);
        p1 = new Panel(null);
        p2 = new Panel(null);
        p3 = new Panel(null);
        p4 = new Panel(null);
        p1.setBounds(0,0,w/2,h/2);
        p1.setBackground(Color.red);
        p2.setBounds(w/2,0,w/2,h/2);
        p2.setBackground(Color.blue);
        p3.setBounds(0,h/2,w/2,h/2);
        p3.setBackground(Color.yellow);
        p4.setBounds(w/2,h/2,w/2,h/2);
        p4.setBackground(Color.black);
        add(p1);add(p2);add(p3);add(p4);
        setVisible(true);
    }
}
import java.awt.*;
public class TestFrame5{
    public static void main(String[] args){
        MyFrame f = new MyFrame
        ("my panel",800,50,400,400);
    }
}
 
class MyFrame extends Frame{
    private Panel p1;
    public MyFrame(String n,int x,int y,int w, int h){
        super(n);
        setBounds(x,y,w,h);
        setBackground(Color.red);
        setLayout(null);
        p1 = new Panel(null);
        p1.setBounds(w/4,h/4,w/2,h/2);
        p1.setBackground(Color.yellow);
        add(p1);
        setVisible(true);
    }
}
```
### Layout Manager
```java

import java.awt.*;
public class TestFlowLayout{
    public static void main(String[] args){
        new Myframe().launchFrame();
    }
}
 
class Myframe extends Frame{
    public void launchFrame(){
        Button b1 = new Button("OK"); 
        Button b2 = new Button("Yes"); 
        Button b3 = new Button("cancle");
        setLayout(new FlowLayout(FlowLayout.LEFT));
        add(b1);
        add(b2);
        add(b3);
        setBounds(300,200,300,300);
        setVisible(true);
    }
}
import java.awt.*;
public class TestFlowLayout1{
    public static void main(String[] args){
        new Myframe().launchFrame();
    }
}
 
class Myframe extends Frame{
    public void launchFrame(){
        setBounds(100,100,300,300);
        FlowLayout layout = new FlowLayout
        (FlowLayout.CENTER,20,40);
        setLayout(layout);
        for(int i=0; i<7; i++){
            add(new Button("button"+(i+1)));
        }
        setVisible(true);
    }
}
import java.awt.*;
public class TestBorderLayout{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Border Layout");
    }
}
 
class MyFrame extends Frame{
 
    public void launchFrame(String name){
        this.setTitle(name);
        Button b1 = new Button("N");
        Button b2 = new Button("S");
        Button b3 = new Button("W");
        Button b4 = new Button("E");
        Button b5 = new Button("C");
        add(b1,BorderLayout.NORTH);
        add(b2,BorderLayout.SOUTH);
        add(b3,BorderLayout.WEST);
        add(b4,BorderLayout.EAST);
        add(b5,BorderLayout.CENTER);
        setBounds(100,200,400,400);
        setVisible(true);
    }
}
import java.awt.*;
public class TestGridLayout{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Grid Layout");
    }
}
 
class MyFrame extends Frame{
    public void launchFrame(String name){
        setTitle(name);
        for(int i=0; i<6; i++){
            add(new Button("button"+(i+1)));
        }
        setLayout(new GridLayout(3,2));
        pack();
        setVisible(true);
    }
}
import java.awt.*;
public class PracticeLayout{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Practice Layout");
    }
}
 
class MyFrame extends Frame{
    public void launchFrame(String name){
        setTitle(name);
        setLayout(new GridLayout(2,1));
        Panel p1 = new Panel(new BorderLayout());
        Panel p11 = new Panel(new GridLayout(2,1));
        add(p1);
        Button b1 = new Button("b1");
        Button b2 = new Button("b2");
        p1.add(b1,BorderLayout.WEST);
        p1.add(b2,BorderLayout.EAST);
        Button b3 = new Button("b3");
        Button b4 = new Button("b4");
        p11.add(b3);
        p11.add(b4);
        p1.add(p11,BorderLayout.CENTER);
         
        Panel p2 = new Panel(new BorderLayout());
        add(p2);
        Panel p22 = new Panel(new GridLayout(2,2));
        Button b5 = new Button("b5");
        Button b6 = new Button("b6");
        p2.add(b5,BorderLayout.WEST);
        p2.add(b6,BorderLayout.EAST);
        p2.add(p22,BorderLayout.CENTER);
        Button b7 = new Button("b7");
        Button b8 = new Button("b8");
        Button b9 = new Button("b9");
        Button b10 = new Button("b10");
        p22.add(b7);
        p22.add(b8);
        p22.add(b9);
        p22.add(b10);
        setBounds(0,0,500,500);
        //pack();
        setVisible(true);
    }
}

```
### Some event and inner class
```java
import java.awt.*;
import java.awt.event.*;
public class TestActionListener{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Action Event");
    }
}
 
class MyFrame extends Frame{
    void launchFrame(String name){
        setTitle(name);
        Button b = new Button("press me!");
        add(b);
        Monitor m = new Monitor();
        b.addActionListener(m);
        pack();
        setVisible(true);
    }
     
    class Monitor implements ActionListener{
        public void actionPerformed(ActionEvent e){
            System.out.println("A button has pressed!");
        }
    }
}
import java.awt.*;
import java.awt.event.*;
public class TestActionListener1{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Action Event");
    }
}
 
class MyFrame extends Frame{
    void launchFrame(String name){
        setTitle(name);
        Button b = new Button("press me!");
        add(b);
        b.addActionListener(new ActionListener(){
            public void actionPerformed(ActionEvent e){
                System.out.println("A button has pressed!");
            }
        });
        pack();
        setVisible(true);
    }
}
import java.awt.*;
import java.awt.event.*;
 
public class TestActionListener2{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Action Listener");
    }
}
 
class MyFrame extends Frame{
    void launchFrame(String name){
        setTitle(name);
        Button b1 = new Button("start");
        Button b2 = new Button("stop");
 
        Monitor m = new Monitor();
        b1.addActionListener(m);
        b2.addActionListener(m);
        //b2.setActionCommand("game over");
        add(b1,BorderLayout.NORTH);
        add(b2,BorderLayout.SOUTH);
        pack();
        setVisible(true);
    }
     
    class Monitor implements ActionListener{
        public void actionPerformed(ActionEvent e){
            System.out.println
            ("info : " + e.getActionCommand());
        }
    }
}
import java.awt.*;
import java.awt.event.*;
 
public class TestActionListener3{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Text Field");
    }
}
 
class MyFrame extends Frame{
    void launchFrame(String name){
        setTitle(name);
        TextField tf = new TextField();
        add(tf);
        Monitor m = new Monitor();
        tf.addActionListener(m);
        pack();
        setVisible(true);
    }
     
    class Monitor implements ActionListener{
        public void actionPerformed(ActionEvent e){
            TextField tf =(TextField)e.getSource();
            System.out.println
            (tf.getText());
            tf.setText(null);
        }
    }
}
import java.awt.*;
import java.awt.event.*;
 
public class TestActionListener4{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Text Field");
    }
}
 
class MyFrame extends Frame{
    void launchFrame(String name){
        setTitle(name);
        TextField tf = new TextField();
        add(tf);
        tf.addActionListener(new ActionListener(){
            public void actionPerformed(ActionEvent e){
                TextField tf =(TextField)e.getSource();
                System.out.println
                (tf.getText());
                tf.setText(null);
            }
        });
        pack();
        setVisible(true);
    }
}
import java.awt.*;
import java.awt.event.*;
 
public class TestActionListener5{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Text Field");
    }
}
 
class MyFrame extends Frame{
    void launchFrame(String name){
        setTitle(name);
        setLayout(new FlowLayout());
        TextField tf1 = new TextField();
        Label label = new Label("+");
        TextField tf2 = new TextField();
        Button b = new Button("=");
        TextField tf3 = new TextField();
        add(tf1);
        add(label);
        add(tf2);
        add(b);
        add(tf3);
        b.addActionListener(new ActionListener(){
            public void actionPerformed(ActionEvent e){
                int i1 = Integer.parseInt(tf1.getText());
                int i2 = Integer.parseInt(tf2.getText());
                tf3.setText(""+(i1+i2));
            }
        });
        pack();
        setVisible(true);
    }
}
import java.awt.*;
public class TestPaint{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Paint");
    }
}
 
class MyFrame extends Frame{
    public void launchFrame(String name){
        setTitle(name);
        setBounds(100,200,200,200);
        setVisible(true);
    }
     
    public void paint(Graphics g){
        Color c = g.getColor();
        g.setColor(Color.red);
        g.fillOval(50,50,40,40);
        g.setColor(Color.yellow);
        g.fillRect(20,40,60,80);
        g.setColor(c);
    }
}
import java.awt.*;
import java.awt.event.*;
import java.util.*;
public class TestMouseEvent{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Mouse Event");
    }
}
 
class MyFrame extends Frame{
    ArrayList<Point> points = null;
    public void launchFrame(String name){
        points = new ArrayList<Point>();
        setTitle(name);
        setBounds(100,200,400,400);
        setBackground(Color.blue);
        setVisible(true);
        addMouseListener(new Monitor());
    } 
     
    public void paint(Graphics g){
        g.setColor(Color.red);
        Iterator<Point> i = points.iterator();
        while(i.hasNext()){
            Point p = i.next();
            g.fillOval(p.x,p.y,10,10);
        }
    }
     
    public void addPoint(Point p){
        points.add(p);
    }
     
    class Monitor extends MouseAdapter{
        public void mousePressed(MouseEvent e){
            MyFrame f = (MyFrame)e.getSource();
            f.addPoint(new Point(e.getX(),e.getY()));
            f.repaint();
        }
    }
}
import java.awt.*;
import java.awt.event.*;
public class TestWindowEvent{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Window Event");
    }
}
 
class MyFrame extends Frame{
    public void launchFrame(String name){
        setTitle(name);
        setLayout(new GridLayout(2,1));
        Panel p1 = new Panel(new BorderLayout());
        Button b1 = new Button("top");
        Button b2 = new Button("bottom");
        Panel p2 = new Panel(new BorderLayout());
        p1.add(b1,BorderLayout.CENTER);
        p2.add(b2,BorderLayout.CENTER);
        add(p1);
        add(p2);
        pack();
        setVisible(true);
        addWindowListener(new WindowAdapter(){
            public void windowClosing(WindowEvent e){
                System.exit(-1);
            }
        });
    }
}
import java.awt.*;
import java.awt.event.*;
public class TestWindowEvent1{
    public static void main(String[] args){
        new MyFrame().lauchFrame
        ("Test Window Event");
    }
}
 
class MyFrame extends Frame{
    public void lauchFrame(String name){
        setTitle(name);
        Button b = new Button("Start");
        TextField tf = new TextField();
        add(b,BorderLayout.NORTH);
        add(tf,BorderLayout.SOUTH);
        pack();
        setVisible(true);
        b.addActionListener(new ActionListener(){
            private int i;
            public void actionPerformed(ActionEvent e){
                tf.setText(e.getActionCommand()+ ++i);
            }
        });
        addWindowListener(new WindowAdapter(){
            public void windowClosing(WindowEvent e){
                System.exit(-1);
            }
        });
    }
}
import java.awt.*;
import java.awt.event.*;
public class TestKeyEvent{
    public static void main(String[] args){
        new MyFrame().launchFrame
        ("Test Key Event");
    }
}
 
class MyFrame extends Frame{
    public void launchFrame(String name){
        setTitle(name);
        Button b = new Button
        ("How many times you press any key");
        TextField t = new TextField();
        add(b,BorderLayout.NORTH);
        add(t,BorderLayout.SOUTH);
        b.addKeyListener(new KeyAdapter(){
            private int i;
            public void keyPressed(KeyEvent e){
                t.setText(""+(++i));
            }
        });
        addWindowListener(new WindowAdapter(){
            public void windowClosing(WindowEvent e){
                System.exit(-1);
            }
        });
        pack();
        setVisible(true);
    }
}
```
## Web Email Spider & Magnet Spider
### EmailSpider
Run instance : java EmailSpider http://tieba.baidu.com/p/2314539885 D:\test.html
```java
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.Reader;
import java.io.Writer;
import java.net.URL;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
 
public class EmailSpider {
     
    private static String emailR = 
            "\\w[-\\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\\.)+[A-Za-z]{2,14}";
    public static Pattern p = Pattern.compile(emailR);
 
    public static void main(String[] args) {
        String url = args[0];
        String pathname = args[1].replace(File.separatorChar,'/'); 
        WriteHtml(url,pathname);
        ReadEmail(pathname);
    }
     
    public static void WriteHtml(String url,String pathname){
        int c;
        try(Reader r = new InputStreamReader(
            new BufferedInputStream(
            new URL(url).openStream()));
                 
            Writer w = new OutputStreamWriter(
            new BufferedOutputStream(
            new FileOutputStream(
            new File(pathname))))
            ){
            while( (c = r.read()) != -1){
                //System.out.println(c);
                w.write(c);
                w.flush();
            }
             
        }catch(IOException e){
            //System.out.println("Connection error!");
            e.printStackTrace();
        }
    }
     
    public static void ReadEmail(String pathname){
        try(BufferedReader br = new BufferedReader(
            new InputStreamReader(
            new FileInputStream(new File(pathname))))
            ){
                String line = "";
                while( (line = br.readLine()) != null){
                    Matcher m = p.matcher(line);
                    while(m.find()){
                        System.out.println(m.group());
                    }
                }
                 
            }catch(IOException e){
                System.out.println("No such file!");
            }
        }
     
}

```
### EmailSpiderModified
```java
import java.io.*;
import java.util.regex.*;
import java.net.*;
public class EmailSpiderModified{
    private static String emailRegExp =
        "\\w[-\\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\\.)+[A-Za-z]{2,14}";
    private static Pattern p = Pattern.compile(emailRegExp);
    private static int counter;
     
    public static void main(String[] args){
         
        String url = args[0];
        emailFromRAM(url);
    }
     
    public static void emailFromRAM(String url){
        String line = null;
        try(BufferedReader br = new BufferedReader(
            new InputStreamReader(
            new URL(url).openStream()))
            ){
                while((line = br.readLine()) != null){
                     
                    //System.out.println(line);
                     
                    Matcher m =   p.matcher(line);
                    while(m.find()){
                        readEmail(m.group());
                        counter++;
                    }
                }
                System.out.println
                ( "本页面共计" + counter + "个Email!");
             
        }catch(IOException e){
            e.printStackTrace();
        }
    }
     
    public static void readEmail(String email){
        System.out.println(email);
    }
 
}
```
### MagnetSpider
```java


import java.io.*;
import java.util.regex.*;
import java.net.*;
public class MagnetSpider{
    private static String magnetRegExp =
        "magnet:\\?xt=urn:btih:[\\w]*";
    private static Pattern p = Pattern.compile(magnetRegExp);
    private static int counter;
     
    public static void main(String[] args){
         
        String url = args[0];
        emailFromRAM(url);
    }
     
    public static void emailFromRAM(String url){
        String line = null;
        try(BufferedReader br = new BufferedReader(
            new InputStreamReader(
            new URL(url).openStream()))
            ){
                while((line = br.readLine()) != null){
                     
                    //System.out.println(line);
                     
                    Matcher m =   p.matcher(line);
                    while(m.find()){
                        readEmail(m.group());
                        counter++;
                    }
                }
                System.out.println
                ( "本页面共计" + counter + "个磁链!");
             
        }catch(IOException e){
            e.printStackTrace();
        }
    }
     
    public static void readEmail(String email){
        System.out.println(email);
    }
 
}
```
## Regular Expression
### Simple practice
```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
 
 
public class Test1 {
 
    public static void main(String[] args) {
        //初步认识Pattern & Matcher
        /*
        print("初步认识Pattern & Matcher");
         
        print("abc".matches("..."));
        print("a3456677a".replaceAll("\\d", "-"));
         
        Pattern p = Pattern.compile("[a-z]{3}");
        Matcher m = p.matcher("abc");
        print(m.matches());
         
        print("");
        */
        //初步认识 . * + ? 
        /*
        print("初步认识 . * + ?");
         
        print("TEST .");//一个任意字符
        print("aaa".matches("a.a"));
        print("ata".matches("a.a"));
 
        print("TEST *");//0个或多个
        print("".matches("a*"));
        print("a".matches("a*"));
 
        print("TEST +");//一个或多个
        print("".matches("a+"));//false
        print("a".matches("a+"));
 
        print("TEST ?");//0个或一个
        print("".matches("a?"));
        print("a".matches("a?"));
        print("aa".matches("a?"));//false
         
        print("TEST others");
        print("234567899".matches("\\d{3,100}"));
        print("192.168".matches("\\d{1,3}\\.\\d{1,3}"));
        print("192".matches("[0-2][0-9][0-5]"));
         
        print("");
        */
        //认识范围
        /*
        print("TEST []");
        print("a".matches("[abc]"));
        print("a".matches("[^abc]"));//取反false
         
        print("A".matches("[a-zA-Z]"));
        print("A".matches("[a-z]|[A-Z]"));
        print("A".matches("[a-z[A-Z]]"));
         
        print("R".matches("[A-Z&&[RGB]]"));//交集
        */
        //认识 \d \D \s \S \w \W  \
        /*
        print("认识 \\d \\D \\s \\S \\w \\W  \\");
        print("4".matches("\\d"));// \d  [0-9]
        print("4".matches("\\D"));// \D  [^0-9]
     
        print(" ".matches("\\s"));// \s 空白字符 [\t\n\x0B\f\r]
        print(" ".matches("\\S"));// \s 反空白字符 [^\s]
         
        print("a".matches("\\w"));// \w [a-zA-Z_0-9]
        print("a".matches("\\W"));// \w [^\w]
         
        print("\\".matches("\\\\"));// 正则"\\\\" 表示  \
        print("abc8881&".matches("[a-z]{1,3}\\d+[%^&]+"));
        */
        //boundary 边界匹配
        /*
        print("边界匹配");//^表示开头 $表示结束
        print("hello world".matches("^h.*d$"));
        print("hello world".matches(".*ld$"));
        print("hello world".matches("^h[a-z]{1,4}\\b.*"));
         */
 
        //practice
        /*
        print("practice");
         
        print("aaa 8888c".matches(".*\\b\\d{4}."));
        print("745998763@qq.com".matches("\\w+@\\w+\\.[A-Za-z]{2,14}"));
        print("a-".matches("\\w[-]"));
        print(".".matches("[\\w[.-]]"));
        print("+".matches("[+]+"));
        print("+q".matches("[+].+"));
        print("-".matches("[-A-Za-z0-9]"));
        */
        //matches find looking at
        /*
        print("matches find looking at");
         
        Pattern p = Pattern.compile("[a-z]{2,4}");
        Matcher m = p.matcher("ab-abd-abcf-OO");
        print(m.matches());
        m.reset();
        print(m.find());
        print(m.start() + "-" + m.end());
         
        print(m.find());
        print(m.start() + "-" + m.end());
         
        print(m.find());       
        print(m.start() + "-" + m.end());
         
        print(m.find());
        //print(m.start() + "-" + m.end());
 
        print(m.lookingAt());
        print(m.lookingAt());
        print(m.lookingAt());
        print(m.lookingAt());*/
         
        //replacement
        /*
        print("replacement");
        Pattern p = Pattern.compile("sakura",Pattern.CASE_INSENSITIVE);
        String str = "sakura SAKURA saKURA SAkura ILoveSakura YouLoveSakuraToo";
        Matcher m = p.matcher(str);
        //print(m.replaceAll("SAKURA"));
        int i = 1;
        StringBuffer buf = new StringBuffer();
        while(m.find()){
            if(i%2 == 0){
                m.appendReplacement(buf, "sakura");
            }else {
                m.appendReplacement(buf, "SAKURA");
            }
            i++;
        }
        m.appendTail(buf);
        print(buf);*/
         
        //group
        /*
        print("group");
        Pattern p = Pattern.compile("([a-z]{1,3})([0-9]{1,2})");
        Matcher m = p.matcher("qwe09 asd78 saj67");
        while(m.find()){
            print(m.group(1));
            print(m.group(2));
        }*/
         
        //
         
        //print("a".matches("[a-z&&[^bc]]"));//差集
         
        //Greedy Reluctant Possessive
 
        /*
        String s = "aahsjkhjdskka";
        //Matcher m = Pattern.compile("[a-z]+?").matcher(s);
        //Matcher m = Pattern.compile("[a-z]*?").matcher(s);
        //Matcher m = Pattern.compile("[a-z]??").matcher(s);
        //Matcher m = Pattern.compile("[a-z]{2,5}").matcher(s);
        Matcher m = Pattern.compile("[a-z]{2,5}?").matcher(s);
        while(m.find()){
            print(m.group());
            print(m.start()+"-"+m.end());
        }*/
         
        //logic operators
        //print("a".matches("[([a-z]|[0-9])[a-z][0-9]]+"));
        //print("0g9ag8".matches("(([a-z]|[0-9])[a-z][0-9])+"));
         
        /*
        Matcher m = Pattern.compile(".{3}(?=a)").matcher("444a66b");
        while(m.find()){
            print(m.group());
        }*/
         
        //back reference
        /*
        Matcher m = Pattern.compile("([a-z][a-z])\\1").matcher("ababcdef");
        while(m.find()){
            print(m.group());//abab 和第一个组相同
        }*/
         
        /*Matcher m = Pattern.compile("([a-z]([a-z]))\\2").matcher("abbcddefr");
        while(m.find()){
            print(m.group());//abb cdd和第二个组相同，第二个组被第一个组包含
        }*/
         
        print("JaVa".matches("(?i)(java)"));
         
         
         
         
         
         
    }
     
         
    public static void print(Object o){
        System.out.println(o);
    }
}
```
## [Daily]Two-way loopback linked list
```java

public class NodeLinkedList{
     
    public static void main(String[] args){
        List list = new List(500);
        print(list.length);
        Count3Quit(list);
         
    }
 
    public static void Count3Quit(List list){
        int countNum = 0;
        Node node = list.head;
        while(list.length > 0){
            countNum ++;
            if(countNum == 3){
                countNum = 0;
                list.deleteNode(node);
            }
            node = node.next;
        }
        print(list.head.id);
        ///print(list.tail.id);
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
     
}
 
class Node{
    int id;
    Node prev;
    Node next;
}
 
 
class List{
    int length = 0; 
    Node head;
    Node tail;
     
    List(int n){
        for(int i=0; i<n; i++){
            addNode();
        }
    }
     
    public void addNode(){
        Node node = new Node();
        if(length == 0){
            head = tail = node;
            print("The first Node added!");
        }else{
            head.prev = node;
            node.next = head;
            tail.next = node;
            node.prev = tail;
            tail = node;
            print("The " + (length + 1)  + "th Node added!");
        }
        length ++;
        node.id = length;
         
    }
     
    public void deleteNode(Node node){
         
        if (length == 1){
            //head = tail = null;
            print("----------Just one Node here");
        }else{
            node.next.prev = node.prev;
            node.prev.next = node.next;
            print("The " + node.id +"th deleted!");
            if(node == head){
                head = node.next;
                print( "----------" + head.id + " is the new head!");
            }else if(node == tail){
                tail = node.prev;
                print( "----------" + tail.id + " is the new tail!");
            }
        }
        length --;
    }
     
 
     
    public void print(Object o){
        System.out.println(o);
    }
 
}  
```
## [Daily] Snake
### Yard
```java
import java.awt.Color;
import java.awt.Font;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.Rectangle;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
 
public class Yard extends Frame {
 
 
 
    PaintThread paintThread = new PaintThread();
    public static final int ROWS = 50;
    public static final int COLS = 50;
    public static final int BLOCK_SIZE = 10;
    Snake snake = new Snake();
    Image offScreen = null;
    static Egg egg = new Egg();
    static boolean rePaint = true;
    static boolean gameOver = false;
    static Font font = new Font("微软雅黑",1 ,15);
    static int Score; 
 
     
    public static void main(String[] args) {
        new Yard().launch();
    }
     
     
 
 
    public void launch(){
        setTitle("贪吃蛇 GO!");
        setBounds(100, 100, COLS * BLOCK_SIZE, ROWS * BLOCK_SIZE);
        setBackground(new Color(175, 164, 255));
        setVisible(true);
        setResizable(false);
     
        addWindowListener(new WindowAdapter() {
 
            @Override
            public void windowClosing(WindowEvent e) {
                System.exit(0);
            }
             
        });
        addKeyListener(new KeyMonitor());
        new Thread(paintThread).start();
    }
     
 
     
    @Override
    public void paint(Graphics g) {
        /*
        g.setColor(Color.DARK_GRAY);
        //画出横线
        for(int i=1; i< ROWS+1; i++ ){
            g.drawLine
            (0, BLOCK_SIZE * i, COLS * BLOCK_SIZE, BLOCK_SIZE * i);
        }
        //画出竖线
        for(int i=1; i< COLS+1; i++ ){
            g.drawLine
            (BLOCK_SIZE * i, 0, BLOCK_SIZE * i, ROWS * BLOCK_SIZE,);
        }*/
 
        g.setFont(font);
        g.setColor(Color.white);
        g.drawString
        ("Snake Postion : [ " + 
            (snake.head.rows - 2)+" , "+snake.head.cols+" ]", 25, 50);
        g.drawString
        ("Egg Postion : [ " + 
            ( egg.rows -2 ) + " , " + egg.cols + " ]" , 25, 67);
        g.drawString("Score : " + Score , 25, 87);
        g.drawString("Moveing count : " + snake.length , 25, 104);
        g.drawString("Up count : " + snake.up , 25, 120);
        g.drawString("Down count : " + snake.down , 25, 134);
        g.drawString("Left count : " + snake.left , 25, 148);
        g.drawString("Right count : " + snake.right , 25, 164);
        g.drawString
        ("Key pressed count : "
             + (snake.left+snake.right+snake.up+snake.down) , 25, 180);
 
         
        snake.draw(g);
        egg.draw(g);
        snake.eat(egg);
        snake.checkDeath();
        if(gameOver){
            g.setColor(Color.yellow);
            g.setFont(new Font("微软雅黑", 0, 50));
            g.drawString("Game Over > <", 80 , 270);
        }  
    }
     
        @Overwrite
        public void update(Graphics g){
            if (offScreen == null){
                offScreen = createImage(COLS * BLOCK_SIZE,ROWS * BLOCK_SIZE);
            }
            Graphics sg = offScreen.getGraphics();
            sg.setColor(new Color(175, 164, 255));
            sg.fillRect(0,0,COLS * BLOCK_SIZE,ROWS * BLOCK_SIZE);
            paint(sg);
            g.drawImage(offScreen,0,0,null);
        }
 
     
    class KeyMonitor extends KeyAdapter{
 
        @Override
        public void keyPressed(KeyEvent e) {
            snake.keyPressed(e);
             
        }
         
    }
     
    public class PaintThread implements Runnable{
        @Override
        public void run() {
            while(rePaint){
                repaint();
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
             
        }
 
     
    }
 
 
}

```
### Snake
```java


import java.awt.Color;
import java.awt.Graphics;
import java.awt.Rectangle;
import java.awt.event.KeyEvent;
import java.util.Random;
 
public class Snake {
 
    Node head;
    Node tail;
    int length;
    int size = Yard.BLOCK_SIZE;
    int up,down,left,right;
 
 
    Random r = new Random();
    Node node = 
        new Node
            (r.nextInt(Yard.ROWS - 15)+9,
                 r.nextInt(Yard.COLS- 15)+9, Direction.randomDirection());
     
 
     
    public Snake() {
        this.head = node;
        this.tail = node;
        length = 1;
    }
     
    public void draw(Graphics g) {
        move();
        for(Node node = head; node != null; node =  node.next){
            node.draw(g);
        }
    }
     
    public void addToTail() {
        Node node = null;
        switch (tail.direction) {
        case Left:
            node = new Node(tail.rows, tail.cols + 1, Direction.Left);
            break;
        case Right:
            node = new Node(tail.rows, tail.cols - 1, Direction.Right);
            break;
        case Up:
            node = new Node(tail.rows + 1 , tail.cols, Direction.Up);
            break;
        case Down:
            node = new Node(tail.rows - 1, tail.cols, Direction.Down);
            break;
        default:
            break;
        }
        tail.next = node;
        node.prev = tail;
        tail = node;
        length ++;
    }
     
    public void addToHead() {
        Node node = null;
        switch (head.direction) {
        case Left:
            node = new Node(head.rows, head.cols - 1, Direction.Left);
            break;
        case Right:
            node = new Node(head.rows, head.cols + 1, Direction.Right);
            break;
        case Up:
            node = new Node(head.rows - 1 , head.cols, Direction.Up);
            break;
        case Down:
            node = new Node(head.rows + 1, head.cols, Direction.Down);
            break;
        default:
            break;
        }
        node.next = head;
        head.prev = node;
        head = node;
        length++;
    }
     
 
     
    public void deleteTail() {
        if(length == 0) return;
        tail = tail.prev;
        tail.next = null;
    }
     
    public void move() {
        addToHead();
        deleteTail();
    }
 
    public void eat(Egg egg) {
         
        if(this.getRectangle().equals(egg.getRectangle())){
            egg.reDraw();
            this.addToTail();
            Yard.Score += 1;
        }
     
    }
     
    public Rectangle getRectangle(){
         
        return new Rectangle
                (head.getCols() * size, head.getRows() * size, size, size);
    }
     
    public void checkDeath() {
        if(head.rows < 3 || 
            head.cols < 1 || head.rows > Yard.ROWS-2 
                    || head.cols > Yard.COLS-2){
             
            Yard.rePaint = false;
            Yard.gameOver = true;
        }else {
            for(Node node = head.next; node != null; node = node.next){
                if(head.rows == node.rows && head.cols == node.cols ){
                    Yard.rePaint = false;
                    Yard.gameOver = true;
                }
            }
        }
         
    }
     
    class Node{
         
        int width = Yard.BLOCK_SIZE;
        int height = Yard.BLOCK_SIZE;
        int rows,cols;
        Node next;
        Node prev;
        Direction direction = Direction.Left;
         
        public Node(int rows, int cols, Direction direction) {
            this.rows = rows;
            this.cols = cols;
            this.direction = direction;
        }
         
        public int getRows() {
            return rows;
        }
 
        public int getCols() {
            return cols;
        }
 
     
 
        public void draw(Graphics g){
            Color a = new Color(255, 213, 164);
 
            g.setColor(a);
            g.fillOval(cols * size, rows * size , width, height);
         
             
        }
         
    }
 
    public void keyPressed(KeyEvent e) {
         
        switch (e.getKeyCode()) {
        case KeyEvent.VK_LEFT:
            if(head.direction != Direction.Right){
                head.direction = Direction.Left;
                left++;
            }
            break;
        case KeyEvent.VK_RIGHT:
            if(head.direction != Direction.Left){
                head.direction = Direction.Right;
                right++;
            }
            break;
        case KeyEvent.VK_UP:
            if(head.direction != Direction.Down){
                head.direction = Direction.Up;
                up++;
            }
            break;
        case KeyEvent.VK_DOWN:
            if(head.direction != Direction.Up){
                head.direction = Direction.Down;
                down++;
            }
            break;
        default:
            break;
        }
         
         
         
    }
 
     
     
     
}

```
### Egg
```java


import java.awt.Color;
import java.awt.Graphics;
import java.awt.Rectangle;
import java.util.Random;
 
 
 
public class Egg {
 
    int rows,cols;
    int size = Yard.BLOCK_SIZE;
    static Random ran = new Random();
 
     
     
    public Egg(int rows, int cols) {
        this.rows = rows;
        this.cols = cols;
    }
 
    public Egg(){
         
        this(ran.nextInt(Yard.ROWS-5)+4, ran.nextInt(Yard.COLS-5)+4);
    }
 
 
    public void draw(Graphics g){
        g.setColor(Color.yellow);
        g.fillOval(cols * size, rows * size, size, size);
    }
 
 
 
    public Rectangle getRectangle() {
        return new Rectangle(cols * size, rows * size, size, size);
         
    }
 
    public int[] getPos() {
        int[] pos = new int[2];
        pos[0] = rows;
        pos[1] = cols;
        return pos;
         
    }
 
    public void reDraw() {
         
        rows = ran.nextInt(Yard.ROWS-5)+4;
        cols = ran.nextInt(Yard.COLS-5)+4;
    }
     
     
}


```
### Direction
```java

import java.util.Random;
 
public enum Direction {
    Left,Up,Right,Down;
     
    static Direction randomDirection(){
        int pick = new Random().nextInt(Direction.values().length);
        return Direction.values()[pick];
    }
}
```
## 编码相关
- 隐式类型转换:计算时自动将容量小的类型转换为容量大的类型:byte,short,char->int->long->float->double. important(整数默认是int ,定义byte应该这样定义: byte b = (byte)12 , 浮点数默认是double , 因此,定义float应该这样定义: float f = 12.4f),总之记得一点:大数转换成小数记得加上强制转换!
- 关于int强制转换为byte的问题(默认byte(1字节)的范围在-128~127,如果把int(4字节)的254强制转换成byte,会砍掉多余字节) , [关于原码,反码,补码的文章](https://www.zhihu.com/question/20159860/answer/71256667), [二进制补码百度百科](http://baike.baidu.com/view/1809209.htm#1), [二进制加法](http://jingyan.baidu.com/article/86112f135745432736978776.html) **[重说三 : 计算机内部用补码来表示数字,正数(最高位是0)原码=反码=补码 ,负数(最高位是1)三码需要转换]**

```java
public class Test{
    public static void main(String[] args){
        byte b1 = (byte)254;//强制转换成byte
        System.out.println(b1);//结果输出-2
         
        byte b2 = (byte)300;
        System.out.println(b2);//结果输出44,正数原码和补码相同
             
        byte b3 = (byte)(-1);
        System.out.println(b3);//打印-1
             
    }
}
/*
解释b1:
1:int的254表示为二进制(正数:原=反=补): 0000 0000 0000 0000 0000 0000 1111 1110(补)
2:强制转换成byte,丢掉前面三个字节,留下(最高位是1,表示负数): 1111 1110(补)
3:补码转反码(最高位符号位不变,其余取反): 1111 1110(补) -->> 1000 0001(反)
4:反码求原码: 1000 0001(反) + 0000 0001 = 1000 0010(原)
5:因此: 1000 0010表示-2
*/
/*
解释b2:
1:int的300表示为二进制(正数:原=反=补): 0000 0000 0000 0000 0000 0001 0010 1100(补)
2:强制转换成byte,丢掉前面三个字节,留下(最高位是0,表示正数): 0010 1100(补)
3:正数原码和补码相同,原码为:0010 1100(原)
4:因此: 0010 1100表示44
*/
/*
解释b3:
1:int的(-1)的原码为:1000 0000 0000 0000 0000 0000 0000 0001(原)
2:原码求反码:1111 1111 1111 1111 1111 1111 1111 1110(反)
3:反码求补码:1111 1111 1111 1111 1111 1111 1111 1111(补)
4:强制转换成byte,丢掉前面三个字节,留下(最高位是1,表示负数): 1111 1111(补)
5:补码求反码:1000 0000(反)
6:反码求原码:1000 0001(原)
7:因此:1000 0001表示-1
*/
```

## Java网络编程示例
### TCP多线程
```java
package com.scriptwang.test;
 
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
 
/**
 * Created by ScriptWang on 16/11/28.
 * Java TCP多线程编程模型;
 * 1 : MutiThreadNetwork 的主线程只负责监听请求(accept方法是阻塞的),当有客户端连接的时候交给新线程去处理
 * 2 : 当Socket连接上之后,可以通过getInput(Output)Strean当作普通的流对待
 *
 */
public class MutiThreadNetwork {
    public static final int PORT = 8888;
 
    public static void main(String[] args) {
        new MutiThreadNetwork().run();
    }
 
    public void run() {
        ServerSocket ss = null;
        Socket s = null;
        try {
            ss = new ServerSocket(PORT);//抛IOException
            print("ss ok !");
 
 
            /**
             * 重点:
             * ServerSocket的accept方法是阻塞式的,如果没有人连接,就会一直等待在那里
             * 并且其会抛出IOException
             */
            while ((s = ss.accept()) != null) {
                print("s is ok!");
                new Thread(new SocketThread(s)).start();
            }
 
        } catch (IOException e) {
            e.printStackTrace();
        }
 
    }
 
    class SocketThread implements Runnable {
        private Socket s;
 
        SocketThread(Socket s) {
            this.s = s;
        }
 
        @Override
        public void run() {
            OutputStream out = null;
            DataOutputStream dos = null;
            try {
                out = s.getOutputStream();//抛IOException
                dos = new DataOutputStream(out);
                dos.writeUTF("Test message!");//抛IOException
                out.close();
                s.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
 
    }
    public static void print(Object o) {
        System.out.println(o);
    }
}
 
/**
 * Client 类
 */
class Client {
    public static final int PORT = 8888;
    public static void main(String[] args){
        new Client().run();
    }
    public void run(){
        Socket s = null;
        DataInputStream dis = null;
        String value = null;
        try {
            s = new Socket("localhost", PORT);//抛IOexception
            dis = new DataInputStream(s.getInputStream());
            value = dis.readUTF();//抛IOException
            dis.close();
            s.close();
        }catch (IOException e){
            e.printStackTrace();
        }
 
        print(value);
 
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
}
```
### WEB服务器
```java
package com.scriptwang.test;
 
 
import java.io.IOException;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
 
/**
 * Created by ScriptWang on 16/11/28.
 * 模拟Web服务器,浏览器发起一个请求相当于建立了一个Socket
 * 当服务器accept到一个Socket之后交给对应的线程去处理
 * 新线程会拿到当前Socket并用PrintWriter println数据给浏览器
 *
 *
 * 在流编程中,flush和close很重要!!!!!!!!!!!!!!!!!!!!
 */
public class WEBServer {
    public static final int PORT = 8080;
    public static void main(String[] args){
        new WEBServer().run();
    }
    public void run(){
        ServerSocket ss = null;
        try {
            ss = new ServerSocket(PORT);
            print("ServerSocket is ready!");
 
            //循环来自客户端的请求
            for (Socket s = ss.accept(); s != null; s = ss.accept()){
                new Thread(new WebThread(s)).start();
                print("new Thread is start!");
            }
        }catch (IOException e){
            e.printStackTrace();
        }
 
 
    }
 
    /**
     * 新的线程类
     */
    class WebThread implements Runnable{
        Socket s;
        WebThread(Socket s){
            this.s = s;
        }
        @Override
        public void run(){
            PrintWriter pw = null;
            try{
                pw = new PrintWriter(s.getOutputStream());
                pw.println("<html>");
                pw.println("<h1>Test Message</h1>");
                pw.println("</html>");
 
                /**
                 * flush很重要,虽然PrintWriter有自动flush功能
                 */
                pw.flush();
 
 
                /**
                 * close很重要,如果不close,浏览器会一直转圈圈,
                 * 以为你还有东西要println给他,其实你已经println完了
                 * 浏览器还在傻傻的等待
                 */
                pw.close();
            }catch (IOException e){
                e.printStackTrace();
            }
 
        }
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
}
```
### UDP简单聊天应用
```java
package com.scriptwang.test;
 
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.*;
 
/**
 * Created by ScriptWang on 16/11/29.
 * UDP简单聊天应用,注意以下几点
 *
 * 1: 基本思路:发送和接收消息各用一个线程,互不干扰
 *
 * 2: System.in本身不阻塞,当用BufferedReader怼到
 *    System.in上的时候,BufferedReader的readLine
 *    会阻塞当读取不到数据的时候,readLine会傻傻等待
 *    直到读取到数据等的时候才会继续向下执行
 *
 * 3: DatagramSocket的receive(DataPacket dp)方法
 *    会阻塞,当读取不到数据肚饿时候会傻傻的等待在那里
 *
 */
public class BasicUDP {
 
    public static void main(String[] args) {
        //监听端口,对方ip,对方接受端口
        new Thread(new BasicUDP().new SendThread(8080,"127.0.0.1",9125)).start();
 
        //我方接收端口
        new Thread(new BasicUDP().new ReceiveThread(7777)).start();
    }
 
 
    class SendThread implements Runnable{
 
        public final int SENDPORT;
        public final int YOURPORT;
        public final String IP;
        DatagramSocket ds = null;
        DatagramPacket dp = null;
 
        SendThread(int port0,String ip,int port1){
            this.SENDPORT = port0;
            this.IP = ip;
            this.YOURPORT = port1;
            try {
                ds = new DatagramSocket(SENDPORT);
            }catch (SocketException e){
                e.printStackTrace();
            }
        }
 
        @Override
        public void run(){
 
            //System.in本身不会阻塞,当和br结合的时候,br的readLine读取不到命令行输入的数据时会阻塞
            //当br结合FileInputStream从文本文件读取的时候不会阻塞,因为文本文件始终有数据
 
            //总结:br的readLine读取不到数据的时候会阻塞
            //比如:readLine和System.in结合会阻塞
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
 
            try{
 
                /**
                 * 执行for循环第一句话的时候,readLine会阻塞等待键盘输入
                 * 输入后执行for 循环体
                 * 返回for检查条件语句,发现line并不是null(line的值为上一次谁人的数据)
                 * 执行for的第三句话,这个时候又会阻塞(等待从命令行读入数据)
                 * 如此循环.......loop loop loop.....
                 */
                for ( String line = br.readLine() ; line != null ; line = br.readLine()){
                    dp = new DatagramPacket(line.getBytes(),0,line.getBytes().length,InetAddress.getByName(IP),YOURPORT);
                    ds.send(dp);
                    print(line + " has sended !");
                }
 
            }catch (IOException e){
                e.printStackTrace();
            }
 
        }
    }
 
 
    class ReceiveThread implements Runnable{
        public final int RECEIVEPORT;
 
        DatagramSocket ds = null;
        DatagramPacket dp = null;
        byte[] packet = null;
 
        ReceiveThread(int port){
            this.RECEIVEPORT = port;
            packet = new byte[256];
            try{
                ds = new DatagramSocket(RECEIVEPORT);
            }catch (SocketException e){
                e.printStackTrace();
            }
        }
 
        @Override
        public void run(){
            dp = new DatagramPacket(packet,packet.length);
            try{
 
                /**
                 * ds的receive方法会阻塞,当读取不到DatagramPacket
                 * 的时候会傻傻地等待!
                 */
                while(true){
                    ds.receive(dp);
                    String value = new String(dp.getData(),0,dp.getLength());
                    print(value);
                }
 
            }catch (IOException e){
                e.printStackTrace();
            }
 
        }
    }
    public static void print(Object o){
        System.out.println(o);
    }
}
```

## Java常用代码示例
### BigDecimal工具类封装
```java
package com.scriptwang.Test;
 
import java.math.BigDecimal;
 
/**
 * Created by ScriptWang on 16/11/26.
 *
 *  BigDecimal可用于精!确!计!算!
 */
public class BigDecimalUtil {
 
 
    public static double add(double a1,double a2){
        //将第一个参数转化成字符串构造BigDecimal对象
        BigDecimal a = new BigDecimal(String.valueOf(a1));
 
        //将第二个参数转化成字符串构造BigDecimal对象
        // (注意两次转化字符串的过程不同)
        BigDecimal b = new BigDecimal(Double.toString(a2));
 
        //调用BigDecimal的add方法,在调用doubleValue转换成double类型
        return a.add(b).doubleValue();
    }
 
 
    public static double substract(double a1,double a2){
        BigDecimal a = new BigDecimal(String.valueOf(a1));
        BigDecimal b = new BigDecimal(String.valueOf(a2));
        return a.subtract(b).doubleValue();
    }
 
    public static double multiply(double a1,double a2){
        BigDecimal a = new BigDecimal(String.valueOf(a1));
        BigDecimal b = new BigDecimal(String.valueOf(a2));
        return a.multiply(b).doubleValue();
    }
 
    public static double divide(double a1,double a2){
        BigDecimal a = new BigDecimal(String.valueOf(a1));
        BigDecimal b = new BigDecimal(String.valueOf(a2));
        return a.divide(b).doubleValue();
    }
 
}

```
### 对象池的使用示例（多例模式）
```java
package com.scriptwang.Test;
 
import java.util.HashSet;
import java.util.Set;
 
/**
 * Created by ScriptWang on 16/11/26.
 *
 * 对象池的使用示例
 */
public class Dog {
    //加载类的时候初始化一个Set对象池,static的
    private static Set<Dog> dogPool = new HashSet<>();
 
    private String name;
    private int age;
 
    //构造方法私有,不允许别人new对象,只允许自己new对象
    private Dog(String name,int age){
        this.name = name;
        this.age = age;
    }
 
    /**
     *
     * @param name
     * @param age
     * @return Dog
     * 工厂方法,在构造对象的时候,如果对象池里有,就返回已有的对象,不再new
     * 如果没有,则将new的对象加入到对象池中,并将其返回
     */
    public static Dog newInstance(String name,int age){
        //循环对象池
        for (Dog dog : dogPool){
            //比较name和age,如果相同,则返回对象池里已有的对象
            if (dog.getName().equals(name) && dog.getAge() == age){
                return dog;
            }
        }
 
        //如果对象池里没有该对象,则new出新对象加入对象池并返回
        Dog dog = new Dog(name, age);
        dogPool.add(dog);
        return dog;
    }
 
 
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
 
 
    @Override
    public String toString(){
        return "Dog : " + name + " : " + age;
    }
}
```
### 字符串反转工具类
```java
package com.scriptwang.Test;
 
/**
 * Created by ScriptWang on 16/11/26.
 *
 *  字符串反转工具类
 */
public class StringReverseUtil {
 
    /**
     * @param s
     * @return
     * 递归,用到length和subString方法
     * 从字符串中间掰开,左右互换,一直循环,知道字符串的长度为1返回
     */
    public static String reverse1(String s){
        int len = s.length();
        if (len == 1) return s;
        String left = s.substring(0,len/2);
        String right = s.substring(len/2,len);
        return reverse1(right) + reverse1(left);
    }
 
 
    /**
     *
     * @param s
     * @return
     * 反向循环
     */
    public static String reverse2(String s){
        String ref = "";
        for (int i=s.length() - 1;i >= 0;i--){
            //反向循环字符串连接方法
            ref += s.charAt(i);
        }
        return ref;
    }
 
    /**
     *
     * @param s
     * @return
     * 正向循环
     */
    public static String reverse3(String s){
        char[] chars = s.toCharArray();
        String ref = "";
        for (int i=0;i<chars.length;i++){
            //正向循环字符串连接方法
            ref = chars[i] + ref;
        }
        return ref;
    }
 
    /**
     *
     * @param s
     * @return
     * 用StringBuffer或StrinBuilder的reverse方法
     */
    public static String reverse4(String s){
        StringBuffer sb = new StringBuffer(s);
        return sb.reverse().toString();
    }
}


```
### Set、List和Map遍历方法
```java
package com.scriptwang.Test;
 
import java.util.*;
 
/**
 * Created by ScriptWang on 16/11/26.
 * 循环遍历Set,List和Map的方法:
 * Set和List都可以用Iterator和foreach遍历,List比Set多一种size和get遍历
 * Map有三种遍历方法,分别是keySey ; values ; Map.Entry和entrySet遍历,都使用foreach循环
 */
public class CollectionLoop {
    public static void main(String[] args){
        SetLoop();
        print("=======split line======");
        ListLoop();
        print("========split line=====");
        MapLoop();
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
 
    /**
     * 两种循环方法:foreach循环 ; iterator循环
     */
    public static void SetLoop(){
        Set<String> set = new HashSet<String>();
        for (int i=0;i<4;i++) set.add(""+i);
 
        //foreach loop
        for(String value : set) print(value);
 
        //iterator loop
        for (Iterator<String> i = set.iterator();i.hasNext();){
            String value = i.next();
            print(value);
        }
 
    }
 
 
    /**
     * 三种循环方法:foreach循环 ; iterator循环 ; size get for循环
     */
    public static void ListLoop(){
        List<String> list = new ArrayList<String>();
        for (int i=0;i<4;++i) list.add(""+i);
 
        //foreach loop
        for (String value : list) print(value);
 
        //iterator loop
        for (Iterator<String> i = list.iterator();i.hasNext();){
            String value = i.next();
            print(value);
        }
 
        //size get for loop
        for (int i=0;i<list.size();i++){
            String value = list.get(i);
            print(value);
        }
    }
 
 
    /**
     * 全是用foreach循环
     * 三种循环方法:keySet和get 循环 ; values循环 ; Map.Entry 和entrySet 循环
     */
    public static void MapLoop(){
        Map<String,Integer> map = new TreeMap<String,Integer>();
        for (int i=0;i<4;i++) map.put("key:"+i,i);
 
        //keySet loop
        for (String key : map.keySet()){
            Integer value = map.get(key);
            print(key);
            print(value);
        }
 
        //values loop 缺点:不能循环出key,只能循环出value
        for (Integer value : map.values()){
            print(value);
        }
 
        //Map.Entry和entrySet循环
        for(Map.Entry<String,Integer> entry : map.entrySet()){
            String key = entry.getKey();
            Integer value = entry.getValue();
            print(key);
            print(value);
        }
    }
 
 
}
```
### 文件复制
```java
package com.scriptwang.Test;
 
import java.io.*;
 
/**
 * Created by ScriptWang on 16/11/26.
 * 
 */
public class CopyFiles {
    public static void main(String[] args){
        copy("/Volumes/Swap/所有PSD归档.zip","/Volumes/Swap/所有PSD归档1.zip");
    }
 
    public static void copy(String path,String pathout){
 
        InputStream in = null;
        OutputStream out = null;
        byte[] buffer = new byte[256];//准备一个缓冲区
 
        try{
            in = new FileInputStream(path);
            out = new FileOutputStream(pathout);
        }catch (FileNotFoundException e){
            System.out.println("File not found,Please check it!");
        }
        try{
            //将文件读取到buffer里面,并返回读取到字节数!!
            // len是指读取到的字节数
            for (int len = in.read(buffer);len > 0;len = in.read(buffer)){
                System.out.println(len);
                 
                //将buffer从0到len写出去!
                out.write(buffer,0,len);
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
 
}
```
### 序列化与反序列化一个对象
```java
package com.scriptwang.Test;
 
import java.io.Serializable;
 
/**
 * Created by ScriptWang on 16/11/26.
 * 
 * 被序列化的类要实现Serializable接口
 */
public class Student implements Serializable{
    String name;
    int age;
 
    Student(String name,int age){
        this.name = name;
        this.age = age;
    }
 
 
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
 
 
 
package com.scriptwang.Test;
 
import java.io.*;
 
/**
 * Created by ScriptWang on 16/11/26.
 *
 * 序列化与反序列化一个对象
 */
public class Test4 {
    public static void main(String[] args){
 
        Student s = new Student("A",12);
        serialize("/Volumes/Swap/data.dat",s);
        Student o = (Student) deserialization("/Volumes/Swap/data.dat");
        System.out.println(o);
    }
 
    /**
     *
     * @param path
     * @param obj
     * 序列化一个类
     */
    public static void serialize(String path,Object obj){
        ObjectOutputStream oos = null;
        try{
            //初始化oos
            oos = new ObjectOutputStream(new FileOutputStream(path));
 
            //将obj写到文件
            oos.writeObject(obj);
        }catch(IOException e){
            e.printStackTrace();
        }
 
    }
 
     
    /**
     *
     * @param path
     * @return
     * 反序列化
     */
    public static Object deserialization(String path){
        ObjectInputStream ois = null;
        Object obj = null;
        try{
            //初始化ois
            ois = new ObjectInputStream(new FileInputStream(path));
 
            //从文件读取对象,会抛出ClassNotFoundException
            obj = ois.readObject();
        }catch(IOException | ClassNotFoundException e){
            e.printStackTrace();
        }
 
        return obj;
    }
}
```
### 生产者和消费者模型
```java
package com.scriptwang.Test;
 
/**
 * Created by ScriptWang on 16/11/26.
 */
public class ProducerAndCostumer {
    public static void main(String[] agrs){
 
        Stack s = new Stack(4);
 
        //将同一个Stack给Producer和Consumer
        new Thread(new Producer(s)).start();
        //可以同时用两个Producer线程,只要他们总的add和pop的个数相同就不会出现死锁
        new Thread(new Producer(s)).start();
 
        new Thread(new Consumer(s)).start();
    }
}
 
 
class Stack{
    String[] datas;
    int index;
 
    //初始化
    Stack(int capacity){
        datas = new String[capacity];
        index = 0;
    }
 
    /**
     *
     * @param s
     * 添加方法
     */
    public synchronized void add(String s){
        /**
         * 当指针指到栈顶的时候表示Stack已经满了,不能在加了
         * 因此告诉Producer线程们该wait了
         * 等待Consumer来叫醒Producer线程
         */
        while (index == datas.length){
            try{
                this.wait();//等待
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
 
        this.notifyAll();//叫醒正在等待的Comsumer线程
        datas[index] = s;
        index ++;
    }
 
    /**
     *
     * @return
     * 移除方法
     */
    public synchronized String pop(){
        /**
         * 当指针指到栈底时表示Stack已经空了,没有东西可以在pop了
         * 这时需要让Consumer线程wait
         * 让Producer线程叫醒Consumer线程
         */
        while(index == 0){
            try{
                this.wait();//Consumer线程等待
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
 
        this.notifyAll();//叫醒Producer线程
        index --;
        return datas[index];
    }
}
 
class Producer implements Runnable{
    private Stack s;
    Producer(Stack s) {
        this.s = s;
    }
 
    @Override
    public void run(){
        for (int i=1;i<=5;i++){//两个Producer线程时,则每个添加五个
            s.add(""+i);
            System.out.println(i + " has add! ^_^ ");
 
        }
    }
}
 
class Consumer implements Runnable{
    private Stack s;
    Consumer(Stack s){
        this.s = s;
    }
 
    @Override
    public void run(){
        for (int i=1;i<=10;++i){//pop10个
            String pop = s.pop();
            System.out.println(pop + " has pop!");
 
            /**
             * pop一个数据后就等待100ms
             * 说明Producer比Consumer快!
             */
            try{
                Thread.sleep(100);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}

```
### 死锁
```java
package com.scriptwang.Test;
 
/**
 * Created by ScriptWang on 16/11/26.
 */
public class DeadLock implements Runnable{
    //flag用于执行不同的代码块
    private boolean flag;
 
    /**
     * 声明o1和o2为static,保证每个DeadLock对象都共享同一份儿o1和o2
     *
     */
    private static Object o1;
    private static Object o2;
 
    // 初始化
    DeadLock(boolean flag){
        this.flag = flag;
 
        //初始化o1和o2
        o1 = new Object();
        o2 = new Object();
    }
 
    public static void main(String[] args){
        new Thread(new DeadLock(false)).start();
        new Thread(new DeadLock(true)).start();
 
    }
 
    @Override
    public void run(){
        //根据flag执行不同的代码块
        if (flag){
            /**
             * 先锁定o1再锁定o2
             */
            synchronized (o1){
                System.out.println("====1");
                try{
                    Thread.sleep(200);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                synchronized (o2){
                    try{
                        Thread.sleep(200);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                    System.out.println("====2");
                }
            }
        } else {
            /**
             * 先锁定o2再锁定o1
             */
            synchronized (o2){
                System.out.println("====3");
                try{
                    Thread.sleep(200);
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
                synchronized (o1){
                    try{
                        Thread.sleep(200);
                    }catch(InterruptedException e){
                        e.printStackTrace();
                    }
                    System.out.println("====4");
                }
            }
        }
    }
 
}
```
### 反射（创建对象、访问Field、调用方法、重写toString）
```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
 
/**
 * Created by Script Wang on 2016/11/27.
 */
 
class Student {
    private String name;
    private int age;
    Student (){
        name = "";
        age = 0;
    }
    Student (String name,int age){
        this.name = name;
        this.age = age;
    }
 
    public void m(){
        System.out.println("I am invoked ===== !!!");
    }
 
 
    /**
     *
     * @return
     * 利用反射重写toString
     */
    @Override
    public String toString(){
        StringBuilder sb = new StringBuilder();
 
        Field[] fields = this.getClass().getDeclaredFields();//拿到所有成员变量
 
        //循环成员变量，将变量名值添加到sb中
        for (int i=0;i<fields.length;i++){
            sb.append(fields[i].getName());//拿到当前Field的名字
            sb.append(" = ");//添加等号
            try{
                sb.append(fields[i].get(this));//拿到当前Field的值
            }catch (IllegalAccessException e){
                e.printStackTrace();
            }
 
            //如果遇到最后一个变量，则末尾不添加分号
            if (i != fields.length - 1){
                sb.append(" ; ");
            }
        }
 
        //返回sb的toString
        return sb.toString();
    }
 
}
 
public class TestReflect {
    public static void main(String[] args){
        test1();//打印示例：name = SW ; age = 21
        test2();//打印示例：name = SW ; age = 22
        test3();//打印示例：I am invoked ===== !!!
    }
 
    /**
     * 用反射中的Constructor构建对象
     */
    public static void test1(){
        Class clazz = null;
        Constructor con = null;
        Student s = null;
        try{
            clazz = Class.forName("Student");//抛ClassNotFoundException
            con = clazz.getDeclaredConstructor(String.class,int.class);//抛NoSuchMethodException
            s = (Student) con.newInstance("SW",21);//抛剩下的三个Exception
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (NoSuchMethodException e){
            e.printStackTrace();
        }catch (InstantiationException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }catch (InvocationTargetException e){
            e.printStackTrace();
        }
 
        System.out.println(s);
    }
 
 
 
    /**
     * 用反射为成员变量赋值构建对象（需要被构建对象的类有空的构造方法）
     * 需要知道构造函数需要哪些类型的参数
     *
     */
    public static void test2(){
        Class clazz = null;
        Object obj = null;
        Field[] fields = null;
 
        try{
            clazz = Class.forName("Student");//抛ClassNotFoundException
            obj = clazz.newInstance();//抛剩下的两个Exception
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (InstantiationException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
 
        //拿到所有Field并循环
        fields = clazz.getDeclaredFields();
        for (Field f : fields){
            f.setAccessible(true);//这句相当重要，可以访问private的变量
            try {
                //get方法会抛IllegalAccessException
                if (f.get(obj) instanceof String){
                    f.set(obj,"SW");//为当前obj 赋值
                } else if (f.get(obj) instanceof Integer){
                    f.set(obj,22);
                }
            }catch (IllegalAccessException e){
                e.printStackTrace();
            }
        }
 
        System.out.println( (Student)obj );
 
    }
 
 
    /**
     * 利用反射调用方法
     */
    public static void test3(){
        Class clazz = null;
        Object obj = null;
        Method[] methods = null;
 
        try{
            clazz = Class.forName("Student");//抛ClassNotFoundException
            obj = clazz.newInstance();//抛剩下的两个Exception
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (InstantiationException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
 
        //拿到所有的Method对象并循环
        methods =clazz.getDeclaredMethods();
        for (Method m : methods){
            if (m.getName().equals("m")){
                m.setAccessible(true);
                try {
                    m.invoke(obj);//调用obj的方法，抛出以下两个Exception
                }catch (IllegalAccessException e){
                    e.printStackTrace();
                } catch (InvocationTargetException e){
                    e.printStackTrace();
                }
 
            }
        }
 
    }
 
 
}
```
###  i++ 和 ++i 的区别
**i++ 先取值再做自加运算; ++i 先做自加运算在取值.**
```java
public class Test{
    public static void main(String[] args){
        int i = 10;
        int i1 = 1 + i++;
        System.out.println(i1);// 11
        System.out.println(i);// 11
        i = 10;
        int i2 = 1 + ++i;
        System.out.println(i2);// 12
        System.out.println(i);// 11
    }
}
```
## JDBC编程示例

```java
package com.scriptwang.test;
 
 
import java.sql.*;
 
/**
 * Created by ScriptWang on 16/11/30.
 * JDBC的基本使用方法
 */
public class TestJDBC {
    public static void main(String[] args){
        Connection c = null;
        Statement s = null;
        PreparedStatement ps = null;
        ResultSet r = null;
        try{
            //1:new驱动程序
            Class.forName("com.mysql.jdbc.Driver");
 
            //2:通过url,username和pwd拿到Connection
            c = DriverManager.getConnection("jdbc:mysql://**","*","*");
 
            //3:通过Conenction创建Statement
            s = c.createStatement();
 
            //4:通过Statement执行sql初始化ResultSet
            r = s.executeQuery("select * from tablename");
 
            //5:对ResultSet进行操作
            while(r.next()){
                String value = r.getString(1);
            }
 
            /**
             * JDBC中比较常用的PreparedStatement的用法
             */
            //通过Connection初始化ps,ps种通过用问号作为占位符
            ps = c.prepareStatement("delete from tablename where id > ?");
            ps.setInt(1,2);//将sql语句种第一个问号设置为2
            int row = ps.executeUpdate();//执行executeUpdate返回受影响的行数
             
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (SQLException e){
            e.printStackTrace();
        }finally{
            try{
                if (r != null){
                    r.close();
                    r = null;
                }
                if (ps != null){
                    ps.close();
                    ps = null;
                }
                if (c != null){
                    c.close();
                    c = null;
                }
            }catch (SQLException e){
                e.printStackTrace();
            }
        }
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
}

```
### JDBC事务的使用
```java

package com.scriptwang.test;
 
 
import java.sql.*;
 
/**
 * Created by ScriptWang on 16/11/30.
 * JDBC的事务使用的基本方法
 */
public class TestJDBC {
    public static void main(String[] args){
        Connection c = null;
        Statement s = null;
        PreparedStatement ps = null;
        ResultSet r = null;
        try{
            //1:new驱动程序
            Class.forName("com.mysql.jdbc.Driver");
 
            //2:通过url,username和pwd拿到Connection
            c = DriverManager.getConnection("jdbc:mysql://*","*","*");
 
            //3:将自动提交设置为false
            c.setAutoCommit(false);
 
            //4:通过Conenction创建Statement
            s = c.createStatement();
 
            //5:通过Statement执行sql
            int row = s.executeUpdate("insert into tablename values (2,'B')");
 
            //6:手动提交
            c.commit();
 
            //7:将自动提交还原
            c.setAutoCommit(true);
 
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }catch (SQLException e){
            /**
             * 当以上操作catch到任何的SQLException的时候,需要回滚
             *
             */
            try{
                //1:回滚数据
                c.rollback();
 
                //2:设置自动提交为true
                c.setAutoCommit(true);
                 
            }catch (SQLException e1){
                e1.printStackTrace();
            }
            e.printStackTrace();
        }finally{
            try{
                if (r != null){
                    r.close();
                    r = null;
                }
                if (ps != null){
                    ps.close();
                    ps = null;
                }
                if (c != null){
                    c.close();
                    c = null;
                }
            }catch (SQLException e){
                e.printStackTrace();
            }
        }
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
}

```
### JDBC实现数据访问对象层DAO（Data Access Object）
```java

package com.scriptwang.test;
 
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
 
/**
 * Created by ScriptWang on 16/11/30.
 * Dao(Data Access Object)数据访问对象层
 * Dao的设计意图是让上层对待底层数据的时候,能够用一个对象的眼光来看待
 */
public class TestDao {
    public static void main(String[] args){
        List<User> list = new JdbcDao().getAllUsers();
        print(list);
 
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
}
 
 
class User{
    private int id;
    private String name;
    User(int id,String name){
        this.id = id;
        this.name = name;
    }
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    @Override
    public String toString(){
        return id + " : " + name;
    }
}
 
class JdbcDao{
 
    /**
     * 静态初始化Driver,无论多少个JdbcDao对象static语句块始终只执行一次
     * 也就是说Driver只初始化一次!!!!
     */
    static {
        try{
            Class.forName("com.mysql.jdbc.Driver");
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }
    }
 
    /**
     *
     * 拿到连接
     */
    public Connection getConn(){
        Connection c = null;
        String url = "jdbc:mysql://*";
        String user = "*";
        String pwd = "*";
        try{
            c = DriverManager.getConnection(url,user,pwd);
        }catch (SQLException e){
            e.printStackTrace();
        }
        return c;
    }
 
    /**
     * 释放资源
     */
    public void release(ResultSet r , Statement s , Connection c){
 
        try{
            if (r != null){
                r.close();
                r = null;
            }
            if (s!= null){
                s.close();
                s = null;
            }
            if (c != null){
                c.close();
                c = null;
            }
        }catch (SQLException e){
            e.printStackTrace();
        }
    }
 
    public void release(Statement s , Connection c){
        try{
            if (s!= null){
                s.close();
                s = null;
            }
            if (c != null){
                c.close();
                c = null;
            }
        }catch (SQLException e){
            e.printStackTrace();
        }
    }
 
    /**
     * 查询所有用户(查)
     */
    public List<User> getAllUsers(){
        List<User> list = new ArrayList<User>();
        Connection c = null;
        Statement s = null;
        ResultSet r = null;
        String sql = "select * from t";
        try{
            c = this.getConn();
            s = c.createStatement();
            r = s.executeQuery(sql);
            while (r.next()){
                User user = new User(r.getInt(1),r.getString(2));
                list.add(user);
            }
            return list;
        }catch (SQLException e){
            e.printStackTrace();
        }finally{
            release(r,s,c);
        }
        return null;
    }
 
 
    /**
     * 根据id获取User对象(查)
     */
    public User getUserById(int id){
        Connection c = null;
        PreparedStatement ps = null;
        ResultSet r = null;
        String sql = "select * from t where id = ?";
        try{
            c = this.getConn();
            ps  = c.prepareStatement(sql);
            ps.setInt(1,id);
            r = ps.executeQuery();
            if (r.next()){
                return new User(r.getInt(1),r.getString(2));
            }
        }catch (SQLException e){
            e.printStackTrace();
        }finally {
            this.release(r,ps,c);
        }
 
        return null;
    }
 
 
    /**
     * 将旧得User改为新的User,返回新的User(改)
     */
    public User updateUser(User oldU,User newU){
        Connection c = null;
        PreparedStatement ps = null;
        String sql = "update t set id = ? ,name = ? where id = ? and name = ?";
        try{
            c = this.getConn();
 
            c.setAutoCommit(false);//事务开始
            ps = c.prepareStatement(sql);
            ps.setInt(1,newU.getId());
            ps.setString(2,newU.getName());
            ps.setInt(3,oldU.getId());
            ps.setString(4,oldU.getName());
            int row = ps.executeUpdate();
            if (row>0){
                c.commit();//提交
                return newU;
            }
        }catch (SQLException e){
            try{
                c.rollback();
                c.setAutoCommit(true);
            }catch (SQLException e1){
                e1.printStackTrace();
            }
            e.printStackTrace();
        }finally {
            release(ps,c);
        }
        return null;
    }
 
 
 
    /**
     * 根据id删除用户(删)
     */
    public boolean deleteUserById(int id){
        Connection c = null;
        PreparedStatement ps = null;
        String sql = "delete from t where id = ?";
        int row = 0 ;
        try{
            c = this.getConn();
 
            c.setAutoCommit(false);// 事务开始
 
            ps = c.prepareStatement(sql);
            ps.setInt(1,id);
            row = ps.executeUpdate();//返回受影响的行数,如果没有,返回0
            if (row > 0){
                c.commit();        //提交事务
                return true;
            }
 
        }catch (SQLException e){
            try{
                /**
                 * 如果上面代码catch到Exception
                 * 第一要做的事情是rollback
                 * 第二是将自动提交设置回原来的值
                 */
                c.rollback();
                c.setAutoCommit(true);
            }catch (SQLException e1){
                e1.printStackTrace();
            }
            e.printStackTrace();
        }finally{
            System.out.println(row + " rows has delete!");
            this.release(ps,c);
        }
        return false;
    }
 
    /**
     * 插入用户数据(增)
     */
    public boolean insUser(User user){
        Connection c = null;
        PreparedStatement ps = null;
        String sql = "insert into t values (?,?)";
        try{
            c = this.getConn();
 
            c.setAutoCommit(false); //事务开始
 
            ps = c.prepareStatement(sql);
            ps.setInt(1,user.getId());
            ps.setString(2,user.getName());
            int row = ps.executeUpdate();
            if (row > 0){
                c.commit();       //提交
                System.out.println("user"+"("+user.getId()+" : "+user.getName()+")"+" has inserted!");
                return true;
            }
        }catch (SQLException e){
 
            try{
                /**
                 * catch到Exception回滚并还原
                 */
                c.rollback();
                c.setAutoCommit(true);
            }catch (SQLException e1){
                e1.printStackTrace();
            }
            e.printStackTrace();
        }finally{
            release(ps,c);
        }
        return false;
    }
 
}
```
## 基本算法
### 打印100以内的素数
```java
package com.scriptwang.test1;
 
import java.util.Date;
 
/**
 * Created by ScriptWang on 16/12/2.
 * 基本算法
 */
public class Test1 {
    public static void main(String[] args){
        test1();
    }
 
    public static void print(Object o){
        System.out.println(o);
    }
 
 
 
    /**
     * 打印100以内的素数
     * 素数:只能被1和自身整除的数,换句话说,
     * 如果除开1和自身还有另外两个整数相乘可以得到这个数
     * 那么这个数就不是素数
     * 比如2,3,5是素数
     * 而4不是素数
     */
    public static void test1(){
        long time = new Date().getTime();
 
        for(int i=1;i<=100;i++){
            if (isPrime(i)) print(i);
        }
 
        long time1 = new Date().getTime();
        print("Used time : " + (time1 - time) );
    }
 
 
    //判断该数是否是素数
    public static boolean isPrime(int i){
        if (i == 1) return false;//任何时候1都不是素数
        if (i == 2) return true;//任何时候2都是素数
 
        /**
         * 最普遍的方法,一个数传进来,比如5,循环2,3,4(不包括1和自身)
         * 如果找到能够除尽的整数,则返回false,说明该数不是素数
         */
        for (int j=2;j<i;j++) {
            if (i % j == 0){
                return false;
            }
        }
        return true;
    }
 
}
```
### 打印九九乘法表
```java


package com.scriptwang.test1;
 
import java.util.Date;
 
/**
 * Created by ScriptWang on 16/12/3.
 * 打印九九乘法表
 */
public class Test2 {
    public static void main(String[] args){
        test1();
    }
 
    //两层循环嵌套
    public static void test1(){
        long time = new Date().getTime();
        for (int i=1;i<=9;i++){
            for (int j=1;j<=i;j++){
                System.out.print(i + "*" + j + "=" + (i*j) + " ");
            }
            System.out.println(" ");
        }
        System.out.println("used time : "+(new Date().getTime() - time));
    }
 
 
 
 
}

```
### 打印10000以内的回文数字
```java

package com.scriptwang.test1;
 
import java.util.Date;
 
/**
 * Created by ScriptWang on 16/12/3.
 * 打印10000以内的回文数字
 * 回文数字:一个数字如果正反都表示同一个数字的话
 *         那么这个数字就是回文数字
 *         比如12521,141,252等
 */
public class Test3 {
    public static void main(String[] args){
        test1();
        test2();
    }
 
    //利用字符串反转,缺点:效率低
    public static void test1(){
        long time = new Date().getTime();
        //1~9不是回文数字,不用参加循环
        for (int i=10;i<=10000;i++){
            String oldNum = String.valueOf(i);
 
            //利用StringBuilder的reverse方法反转字符串
            String newNum = new StringBuilder(oldNum).reverse().toString();
            if (newNum.equals(oldNum)) System.out.print(i + " ");
        }
        System.out.println();
        System.out.println("used time:"+ (new Date().getTime() - time));
    }
 
 
    //用%将int中的每个数字提取出来,再重新组装成新的数字,优点:效率高
    //经过测试,此法是字符串取反方法效率的两倍至三倍
    public static void test2(){
        long time = new Date().getTime();
        for (int i=10;i<=10000;i++){
             
            int value = i;//临时保存i的值
            int oldNum = i;//保存i的值，用来和refNum比较
 
            int temp1 = 0;//保存每次乘以10的结果
            int temp2 = 0;//保存每次循环的个位数
            int refNum = 0;//保存结果
 
            /**
             * 一个数去 % 10回得到这个数的最后一位(个位),比如123%10=3,546%10=6;
             * 一个int去除以10会舍去小数点后面的数字,比如int num=123,num=num/10,此时的num为12
             * 下面的算法正是利用了这两个特点,将一个数"倒装"起来.
             */
            while (value > 0){
                //将每次得到的结果乘10达到个位变十位，十位变百位... 的效果
                temp1 = refNum * 10;
 
                //将每次的value值%10，得到当前value值的最一位
                temp2 = value % 10;
                 
                //得到每次处理后的结果
                refNum = temp1 + temp2;
                 
                //更新value，丢掉value的最后以为(因为value是int型的，会丢掉小数)
                //当value只有一位数的时候，valu/10的结果为0推出循环
                value /= 10;
            }
            if (refNum == oldNum) System.out.print(i+" ");
        }
        System.out.println();
        System.out.println("used time:"+ (new Date().getTime() - time));
    }
}
```
### 三种排序算法：（选择排序，冒泡排序，插入排序）
```java

/**
 * Created by Script Wang on 2016/12/12.
 */
public class Sort {
    public static void main(String[] args){
        int[] s = new int[20];
        for (int i=0;i<s.length;i++){
            s[i] = (int)(Math.random()*100);
        }
 
        Sort.insertSort(s);
        for (int i=0;i<s.length;i++) System.out.print(s[i]+" ");
    }
 
    /**
     * 选择排序：将第一个（最后一个）数字与之后（前）的所有数字一一比较，如果发现有
     * 比第一个数字还要小（大）的，调换他们的位置;循环完第一次将最小（大）的数字放在
     * 了第一个（最后一个）位置，如此一直循环到最后一个数字位置。
     *
     */
    public static void selectionSort(int[] nums){
        //从开始循环到最后
        for (int i=0,temp=0;i<nums.length;i++) for (int j=i+1;j<nums.length;j++){
            if (nums[i]>nums[j]){
                temp = nums[i];
                nums[i] = nums[j];
                nums[j] = temp;
            }
        }
 
        //从最后循环到开始
        for (int temp=0,i=nums.length-1;i>=0;i--) for (int j=i-1;j>=0;j--){
            if (nums[j]>nums[i]){
                temp = nums[j];
                nums[j] = nums[i];
                nums[i] = temp;
            }
        }
 
    }
 
 
    /**
     * 冒泡排序：从第一个（最后一个数开始），依次比较相邻的两个数，如果左数大于后数，
     * 则调换位置，这样外循环循环一次，就将最大的数选出来放在最右边。
     */
    public static void bubbleSort(int[] nums){
        //从开始循环到最后
        for (int i=0,temp=0;i<nums.length;i++) for (int j=0;j<nums.length-1-i;j++){
            if(nums[j] > nums[j+1]){
                temp = nums[j];
                nums[j] = nums[j+1];
                nums[j+1] = temp;
            }
        }
         
        //从最后循环到开始
        for (int temp=0,i=nums.length-1;i>=0;i--) for (int j=nums.length-1;j>nums.length-1-i;j--){
            if (nums[j-1] > nums[j]){
                temp = nums[j-1];
                nums[j-1] = nums[j];
                nums[j] = temp;
            }
        }
    }
 
 
    /**
     * 插入排序：从第二个数开始，依次插入右边的数，左边的所有数总是排好序的了
     * 如果发现右边将要排序的数大于与之相邻的左边的第一个数，那么内循环break！
     * 比如：1，2，3，4，6外循环循环到6，内循环从6开始循环到2，发现6比4大，则
     * 内循环break！
     *
     */
    public static void insertSort(int[] nums){
        //从开始循环到最后
        for (int i=1,temp=0;i<nums.length;i++) for (int j=i;j>0;j--){
            if (nums[j-1] > nums[j]){
                temp = nums[j-1];
                nums[j-1] = nums[j];
                nums[j] = temp;
            }else break;//特别注意break，插入排序是将一个数插入到已经排好序的序列中
            //如果发现待排序数大于左边的第一个数，那么break
 
        }
         
        //从最后循环到开始
        for (int temp=0,i=nums.length-1-1;i>=0;i--) for(int j=i;j<=nums.length-1-1;j++){
            if (nums[j] > nums[j+1]){
                temp = nums[j];
                nums[j] = nums[j+1];
                nums[j+1] = temp;
            }else break;
        }
 
    }
 
 
}
```
## 设计模式
### 单例模式（Singleton）、多例模式（Multiton）
```java


package com.scriptwang.dp;
 
/**
 * Created by ScriptWang on 16/12/13.
 *
 * 单例模式的特点
 * 1:只能有一个实例
 * 2:它必须自行创建自己(自行进行初始化)
 * 3:必须向整个系统提供这个实例
 *
 * 它有两种实现模式:懒汉式和恶汉式
 */
public class UserClass {
 
    public static void main(String[] args){
        Singleton1 s = Singleton1.getInstance();
        Singleton1 s1 = Singleton1.getInstance();
        System.out.println(s == s1);
        Singleton2 s3 = Singleton2.getInstance();
    }
}
 
 
 
//懒汉式 需要的时候才创建,但是可能会线程不安全,需要同步方法
class Singleton1{
    private static int type;
    private static String name;
    private static Singleton1 s;
    private Singleton1(){}//私有构造方法,不允许别人new
    static {//在类被加载的时候自行初始化变量
        type = 1;
        name = "S";
    }
    //懒汉式:当需要的时候才创建该类的实例
    public synchronized static Singleton1 getInstance(){
        if (s == null) s = new Singleton1();
        return s;
    }
}
 
//恶汉式 在类被加载的时候初始化,可能会引起资源浪费
class Singleton2{
    private static int type;
    private static String name;
    private static Singleton2 s;
    private Singleton2(){}//私有构造方法,不允许别人new
    static {
        type = 1;
        name = "S";
        //恶汉式,在加载该类的时候就创建了该类的实例,而不管需要不需要
        s = new Singleton2();
    }
 
    public static Singleton2 getInstance(){
        return s;
    }
}
        
```

实现这样一个逻辑：不允许客户端类new产品类，不管客户端调用多少次getInstance方法，始终返回同一个产品。
```java

//产品类
public class Car {
    private static Car c;//使用静态变量，保证每次返回的都是同一辆Car
 
    private Car(){}//构造器私有，不允许别人new
 
    static {//static语句块会在该类被Load的时候执行一次，对变量进行初始化
        c = new Car();
    }
 
    public static Car getInstance(){//自己控制自己产生的过程，相当于静态工厂方法
        //cheak something here
        return c;
    }
 
    public void drive(){
        System.out.println("Car is running...");
    }
}
//客户端类
public class Test {
    public static void main(String[] args){
        Car c = Car.getInstance();//通过Car提供的静态方法拿到Car的实例，此实例的产生过程由Car自己控制（比如检查权限等）
        c.drive();
        Car c1 = Car.getInstance();
        c1.drive();
        System.out.println(c == c1);//打印true，说明c和c1指向同一个对象，这就是所谓的单例模式！
    }
}
```

单例模式的另一个例子：坦克大战项目中的PropertyManager类，从配置文件中读取属性的时候，不需要每读取一次就new一个java.util.Properties的对象，这样太浪费资源，故使用单例模式！
```java
//测试类
public class Test {
    public static void main(String[] args){
        String name = PropertyManager.getProperty("name");
        System.out.println(name);//打印testMessage，表明读取属性成功
 
        Properties p1 = PropertyManager.getInstance();
        Properties p2 = PropertyManager.getInstance();
        System.out.println(p1 == p2);//打印true，表明无论PropertyManager被怎么调用，其内部只有一个java.util.Properties类的实例（这就是单列模式的应用）
    }
}
 
 
//属性文件，名称：properties
name=testMesage
 
 
//PropertyManager类
import java.io.IOException;
import java.util.Properties;
 
public class PropertyManager {
 
    //在此例中p是单例，无论PropertyManager被怎么调用，内存中只有一个p
    private static Properties p = null;
 
    static {//当类被加载的时候初始化p，且static语句块只执行一次
 
        p = new Properties();
        try {
           p.load(PropertyManager.class.
            getClassLoader().getResourceAsStream(“properties”));//加载名为properties的属性文件
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
 
    private PropertyManager(){}//不允许别人直接new本类的对象
 
    public static String getProperty(String key){//通过p拿到其属性值
        return p.getProperty(key);
    }
 
    public static Properties getInstance(){
        return p;
    }
}


```
### 对象池的使用（多例模式）
```java


package com.scriptwang.Test;
 
import java.util.HashSet;
import java.util.Set;
 
/**
 * Created by ScriptWang on 16/11/26.
 *
 * 对象池的使用示例
 */
public class Dog {
    //加载类的时候初始化一个Set对象池,static的
    private static Set<Dog> dogPool = new HashSet<>();
 
    private String name;
    private int age;
 
    //构造方法私有,不允许别人new对象,只允许自己new对象
    private Dog(String name,int age){
        this.name = name;
        this.age = age;
    }
 
    /**
     *
     * @param name
     * @param age
     * @return Dog
     * 工厂方法,在构造对象的时候,如果对象池里有,就返回已有的对象,不再new
     * 如果没有,则将new的对象加入到对象池中,并将其返回
     */
    public static Dog newInstance(String name,int age){
        //循环对象池
        for (Dog dog : dogPool){
            //比较name和age,如果相同,则返回对象池里已有的对象
            if (dog.getName().equals(name) && dog.getAge() == age){
                return dog;
            }
        }
 
        //如果对象池里没有该对象,则new出新对象加入对象池并返回
        Dog dog = new Dog(name, age);
        dogPool.add(dog);
        return dog;
    }
 
 
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
 
 
    @Override
    public String toString(){
        return "Dog : " + name + " : " + age;
    }
}

```
### 简单工厂模式（Simple Factory）
```java


package com.scriptwang.dp;
 
/**
 * Created by ScriptWang on 16/12/13.
 * 简单工厂模式:
 * 有一系列产品,他们都属于某一种产品(Cat,Dog都属于Animal)
 * 创建一个工厂来管理他们,这个工厂称为简单工厂
 * 
 * 扩展性:当新添加一种产品的时候，就要在工厂方法里新添加if...else语句
 */
public class UserClass1 {
    public static void main(String[] args){
        Animal cat = new AnimalFactory().createAnimal(0);
        Animal Dog = new AnimalFactory().createAnimal(1);
    }
}
 
class Animal{
    //...
}
class Cat extends Animal{
    //...
}
 
class Dog extends Animal{
    //...
}
class AnimalFactory{
    public static Animal createAnimal(int type){
        if (type == 0) return new Cat();
        else if (type == 1) return new Dog();
        return null;
    }
}

```
### 简单工厂模式与反射机制的结合
```java


//抽象产品
public abstract class Car {
    String name;
    public abstract void drive();//抽象方法，其子类必须实现
    public static void print(Object o){System.out.println(o);}
}
//具体产品Bmw和Benz
public class Bmw extends Car{
    Bmw(){this.name = "bmw";}
    public void drive(){ print(name + " is running!");}
}
 
public class Benz extends Car {
    Benz(){this.name = "Bezn";}
 
    public void drive(){print(name + " is running!");}
}
//简单工厂
public class CarFactory {
    private static Car car = null;
    private CarFactory(){}//private构造方法，禁止别人new对象
 
    public static Car createCar(String CarName){//工厂静态生产方法
        try{//利用反射机制，这样新增加一个产品类并不需要修改工厂类
          car = (Car) Class.forName(CarName).newInstance();
      }catch(Exception){
        e.printStackTrace();
      }
      return car;
    }
}
 
//客户端类
public class UserClass {
    public static void main(String[] agrs){
        String name = “Benz”;//相当于从配置文件读取的字符串
        Car c = CarFactory.createCar(name);//根据字符串用工厂生产产品
        c.drive();//使用产品
    }
}

```
### 抽象工厂模式
```java

package com.scriptwang.dp;
 
/**
 * Created by ScriptWang on 16/12/13.
 * 抽象工厂模式:
 * 有一系列产品,他们都属于某一种产品(Cat,Dog都继承自Animal)
 * 有一系列工厂,他们都从一个工厂继承,和产品的继承关系类似.
 * 每一个产品都由一个工厂负责,new的是什么工厂就生产什么产品
 * 
 * 扩展性:当新添一种产品的时候,就要新添加一种与之相对应的工厂
 */
public class UserClass1 {
    public static void main(String[] args){
        //创建的是什么工厂就生产什么
        AnimalFactory a1 = new CatFactory();
        AnimalFactory a2 = new DogFactory();
        Animal cat = a1.createAnimal();//生产了猫
        Animal dog = a2.createAnimal();//生产了狗
    }
}
 
class Animal{
    //...
}
class Cat extends Animal{
    //...
}
 
class Dog extends Animal{
    //...
}
 
 
abstract class AnimalFactory{
    public abstract Animal createAnimal();
}
 
class CatFactory extends AnimalFactory{
    public Animal createAnimal(){
        return new Cat();
    }
}
 
class DogFactory extends AnimalFactory{
    public Animal createAnimal(){
        return new Dog();
    }
}
```
### 观察者模式
```java

package com.scriptwang.dp;
 
import java.util.HashSet;
 
/**
 * Created by ScriptWang on 16/12/13.
 * 观察者模式:当被观察者对象作出更改时,被观察者会主动通知观察者,让观察者响应这种更改
 * 
 */
public class UserClass {
    public static void main(String[] args){
        Product p = new Product("《字体设计》",102.8);
        WebObserver web = new WebObserver();
        MailObserver mail = new MailObserver();
        web.Reg(p);
        mail.Reg(p);
        p.setPrice(100.0);
        System.out.println("==split line=====after unReg Observer===");
        mail.unReg(p);
        p.setPrice(10.0);
    }
}
 
 
/**
 * 观察者类
 */
class Observer{
    public void update(Product p){};
     
    //注册自己(把Product的HashSet拿过来,再把自己装进去)
    public void Reg(Product p){
        p.getObservers().add(this);
    }
     
    //撤销自己(把Product的HashSet拿过来,再把自己移除)
    public void unReg(Product p){
        p.getObservers().remove(this);
    }
 
}
 
 
class WebObserver extends Observer{
 
    @Override
    public void update(Product p) {
        String name = p.getName();
        double price = p.getPrice();
        System.out.println("网页助手:"+name + "已经降到"+price+"了!请及时更新网页信息");
    }
 
}
 
class MailObserver extends Observer{
    @Override
    public void update(Product p) {
        String name = p.getName();
        double price = p.getPrice();
        System.out.println("邮箱助手:"+name + "已经降到"+price+"了!");
    }
 
}
 
 
class Product{
    private String name;
    private double price;
    //同HashSet装载观察者对象,HashSet不允许重复值,可避免重复装载
    HashSet<Observer> observers;
     
    Product(String name,double price){
        this.name = name;
        this.price = price;
        //在Product被初始化的时候初始化HashSet
        observers = new HashSet<Observer>();
    }
 
    /**
     * 
     * @return
     * 此方法用于观察者注册与取消注册
     */
    public HashSet<Observer> getObservers(){
        return observers;
    }
 
 
    /**
     * 依次回调Observer的update方法,让Observer作出相应响应
     */
    public void notifyObserver(){
        if (observers != null){
            for (Observer o : observers){
                o.update(this);
            }
        }
    }
 
    public double getPrice() {
        return price;
    }
 
    /**
     * 
     * @param price
     * 当价格变化时通知Observer
     */
    public void setPrice(double price) {
        this.price = price;
        this.notifyObserver();//通知
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
     
}

```
### 数据结构：java链表
```java

import java.util.LinkedList;
 
/**
 * Created by Script Wang on 2016/12/14.
 * 单向链表（线性链表）的实现
 * 1：一般说的链表指的是单向链表，只能从一头到另一头，不能反过来
 *    如果是可以从两个方向遍历的，称为双向链表
 * 2：链表的关键点在于它对每个节点的next指针的操作，不论是插入，删除或修改操作
 *    都是去某个或某些节点的next引用！
 *
 */
public class UserClass {
    public static void main(String[] args){
        Mylist list = new Mylist();
        for (int i=0;i<20;i++) list.add(i,new Integer(i));
        System.out.println(list);
    }
}
 
 
class Mylist{
    /**
     * 节点内部类
     */
    class Node {
        Object data;//(数据域)
        Node next;//(指针域)
 
        Node(Object data){
            this.data = data;
            next = null;
        }
        public Object get(){
            return data;
        }
 
        public String toString(){
            return data.toString();
        }
 
    }
 
    Node head;//头节点，是判断链表是否为空，获取链表长度等方法实现的重要成员变量！
 
    Mylist(){
        head = null;
    }
 
    public void clear(){
        head = null;
    }
 
    public boolean isEmpty(){
        if (head == null) return true;
        return false;
    }
 
    public int length(){
        int sum = 0;
        //每循环一次更新n，直到n=null停止循环
        for (Node n = head;n != null;n=n.next) sum++;
        return sum;
    }
 
    /**
     * 拿到某个节点，实质上是循环多少次n=n.next语句
     */
    public Node get(int pos){
        if (pos < 0 || pos > length()) {
            throw new RuntimeException("一共有"+length()+"个对象，你要找啥？");
        }
        Node n = head;
        for (int i=0;i<pos;i++) n=n.next;
        return n;
    }
 
 
 
    public void add(int pos,Object data){
        if(pos < 0) throw new RuntimeException("你要往负方向插？只能往0和正方向插！");
        else if (pos > length()) throw new RuntimeException("下标大于最后一个元素！");
        Node n = new Node(data);
        if (pos == 0 ){
            /**
             * 此处理解需要注意，并不是n.next=head;head=n;n和head就无限循环了
             * 而是：
             * new MyList时，head值初始化为null，换句话说head = null
             * 现在n.next = head;是说n.next指向null，而不是head自身
             * head = n;再令head指向n
             * 因此，最终的结果（在链表里面只有一个元素的时候）
             * 是head -->> n -->> null
             * 而不是 head -->> n -->> head （无限循环，当获取length的时候怎么获取？）
             */
            n.next = head;
            head = n;
        }else if (pos == length()){
            get(length()-1).next = n;
        }else {
            /**
             * 这里的顺序是很有讲究滴，必须
             * First:先把当前位置的Node赋值给n.next引用
             * Second:再把n赋值给当前位置的上一个的next引用
             * 不能颠倒！！！！！
             */
            n.next = get(pos);
            get(pos-1).next = n;
 
        }
 
    }
 
    public Object remove(int pos){
        if(pos < 0) throw new RuntimeException("你要往负方向移除？");
        else if (pos > length()) throw new RuntimeException("下标大于最后一个元素！");
        if (pos == 0) {
            Node n = get(0);
            head = get(1);
            return n.get();
        } else if (pos == (length()-1)) {
            Node n = get(length()-1);
            get(pos-1).next = null;
            return n.get();
 
        } else {
            Node n = get(pos);
            get(pos-1).next = get(pos+1);
            return n.get();
        }
    }
 
    public String toString(){
        StringBuilder sb = new StringBuilder();
        for (Node n = head;n != null;n = n.next) {
            if (get(length()-1) == n) {
                sb.append(n.toString());
                continue;
            }
            sb.append(n.toString()+" , ");
        }
        return "["+sb.toString()+"]";
    }
 
}
```

