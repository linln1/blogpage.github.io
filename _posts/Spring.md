---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/29 18:26:39 -->


<div id="toc"></div>
#Java笔记
##Spring
###IoC容器
public class BookService {
    private HikariConfig config = new HikariConfig();
    private DataSource dataSource = new HikariDataSource(config);

    public Book getBook(long bookId) {
        try (Connection conn = dataSource.getConnection()) {
            ...
            return book;
        }
    }
}
