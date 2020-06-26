---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/25 10:30:21 -->


<div id="toc"></div>

#Java笔记
##IO

###File
    import java.io.*;
    public class Main{
        public static void main(String[] args){
            File f = new File("C:\\Window\\notepad.exe");
            System.out.println(f);
        }
    }

    //遍历文件
    public class Main{
        public static void main(String[] args) throws IOException{
            File f = new File("C:\\Windows");
            File[] fs1 = f.listFiles();
            printFiles(fs1);
            File[] fs2 = f.listFiles(new FilenameFilter(){
                public boolean accept(File dir, String name){
                    return name.endsWith(".exe");
                }
            });
            printFiles(f2);
        }

        static void printFiles(File[] files){
            System.out.println("========");
            if(files != null){
                for(File f : files){
                    System.out.prinln(f);
                }
            }
            System.out.println("========")；
        }

    }

    //Java 有一个Path对象，位于java.nio.file

###InputStream
    public abstract int read() throws IOException
    public void readFile() throws IOException{
        
    }

