---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/27 11:23:54 -->


<div id="toc"></div>

#Java笔记
##Safety 

###Encode
    ####URL-Encoding
    import java.net.URLEncoder;
    import java.nio.charset.StandardCharsets;
    public class Main{
        public static void main(String[] args){
            String encoded = URLEncoder.encode("zhongwen!", StandardCharsets.UTF_8);
            System.out.println(encoded);
        }
    }
    ####Base64-Encoding
    
