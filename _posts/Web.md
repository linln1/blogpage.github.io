---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/30 13:21:53 -->


<div id="toc"></div>
#Java笔记
##Web开发

###编写HTTP Server
一个HTTP Server本质上是一个TCP服务器
    public class Server{
        public static void main(String[] args) throws IOException{
            ServerSocket ss = new ServerSocket(8080);
            System.out.println("Server is running....");
            for(;;){
                Socket sock = ss.accept();
                System.out.println("Connected from" + sock.getRemoteSocketAddress());
                Thread t = new Handler(sock);
                t.start();
            }
        }
    }

    class Handler extends Thread {
        Socket sock;

        public Handler(Socket sock) {
            this.sock = sock;
        }

        public void run() {
            try (InputStream input = this.sock.getInputStream()) {
                try (OutputStream output = this.sock.getOutputStream()) {
                    handle(input, output);
                }
            } catch (Exception e) {
                try {
                    this.sock.close();
                } catch (IOException ioe) {
                }
                System.out.println("client disconnected.");
            }
        }

        private void handle(InputStream input, OutputStream output) throws IOException {
            var reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
            var writer = new BufferedWriter(new OutputStreamWriter(output, StandardCharsets.UTF_8));
            // TODO: 处理HTTP请求
        }
    }

    private void handle(InputStream input, OutputStream output) throws IOExecption{
        System.out.println("Process new http requeset...");
        var reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
        var writer = new BufferedReader(new OutputStream Writer(output, StandardCharsets.UTF8));
        //读取HTTP请求
        boolean requestOK = false;
        String first = reader.readLine();
        if(first.startsWith("GET / HTTP/1.")){
            requestOK = true;
        }
        for(;;){
            String header = reader.readLine();
            if(header.isEmpty()){
                break;
            }
            System.out.println(reader);
        }
        System.out.println(requestOk ? "Response OK" : "Response Error");
        if(!requestOK){
            writer.write("404 Not Found\r\n");
            writer.write("Content-Length: 0\r\n");
            writer.write("\r\n");
            writer.flush();
        }else{
            String data = "<html><body><h1>Hello, world!</h1></body></html>";
            int length = data.getBytes(StandardCharsets.UTF_8).length;
            writer.write("HTTP/1.0 200 OK\r\n");
            writer.write("Connection: close\r\n");
            writer.write("Content-Type: text/html\r\n");
            writer.write("Content-Length: " + length + "\r\n");
            writer.write("\r\n"); // 空行标识Header和Body的分隔
            writer.write(data);
            writer.flush();
        }
    }


###Servlet入门
    使用Servlet API编写自己的Servlet来处理HTTP请求，Web服务器实现Servlet API接口，实现底层功能

                     ┌───────────┐
                     │My Servlet │
                     ├───────────┤
                     │Servlet API│
    ┌───────┐  HTTP  ├───────────┤
    │Browser│<──────>│Web Server │
    └───────┘        └───────────┘

    @WebServlet(urlPatterns = "/")
    public class HelloServlet extends HttpServlet{
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
            // 设置响应类型:
            resp.setContentType("text/html");
            // 获取输出流:
            PrintWriter pw = resp.getWriter();
            // 写入响应:
            pw.write("<h1>Hello, world!</h1>");
            // 最后不要忘记flush强制输出:
            pw.flush();
        }
    }

