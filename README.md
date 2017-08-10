# QQserver<br>
<p>
<ol>
QQClient端<br>
<li>改ip</li>
修改com.way.chat.common.util.Constants.SERVER_IP为你自己电脑的ip<br>
查看ip,开始---运行---cmd(命令提示符)---ipconfig<br>
<li>运行客户端</li>
真机测试时如果真机不能连接自己电脑上的服务器,先要关闭防火墙;<br>
真机不能连接自己电脑的服务器的时候,可以使用模拟器测试<br>
 
======================================================================
</ol>
<ol>
原理:socket通信+多线程<br>
<li>服务器QQServer开启了一个ServerSocket,等待客户端连接;</li>
<li>用户操作安卓客户端的时候,客户端QQClient创建一个Socket后去连接服务器,创建一个输出线程ClientOutputThread,将客户端的消息发送给服务器;</li>
<li>服务器使用InputThread接收用户发送的消息,根据消息的不同调用不同的代码进行消息处理(如登陆消息就验证用户名和密码,调用dao查询数据库,返回验证结果),处理完毕后,服务器将处理结果通过OutputThread发送给客户端;</li>
<li>客户端使用ClientInputThread接收服务器返回的结果,根据结果不同展现不同的界面(...登陆成功进入QQ主界面,验证失败给出用户名密码错误的界面)</li>
 
====================================================================<br>
  </ol>

  <pre><code>
 等待代码:<br>
@Override<br>
public void run() {
  try {<br>
   while (isStart) {
    if (msg != null) {
     System.out.println("msg:"+msg);
     oos.writeObject(msg);
     oos.flush();<br>
     if (msg.getType() == TranObjectType.LOGOUT) {
      // 如果是发送下线的消息，就直接跳出循环
      break;
     }
     synchronized (this) {
      wait();// 发送完消息后，线程进入等待状态
     }<br>
    }<br>
   }<br>
   oos.close();// 循环结束后，关闭输出流和socket
   if (socket != null && !socket.isClosed())
    socket.close();
  } catch (InterruptedException e) {
   e.printStackTrace();
  } catch (IOException e) {
   e.printStackTrace();
  }
 }
 </code></pre>

<pre><code>
唤醒代码:<br>
public void setMsg(TranObject msg) {
  this.msg = msg;
  synchronized (this) {
   notify();
  }
 }
 </code></pre>
 <ul>
  <li>客户端输入线程ClientInputThread接到消息后,会进入等待状态,直到下次再次收到服务器发送的消息后,再唤醒输入线程去读取服务器发送的消息;</li>
<li> 服务器输出线程OutputThread发消息后,会进入等待状态,直到下次服务器需要给客户端发送消息是,再唤醒输出线程;</li>
<li> 服务器输入线程InputThread接收消息后,会进入等待状态,直到下次客户端发送消息过来,再唤醒输入线程去读取客户端发送的消息;</li>
 </ul>
  </p>
