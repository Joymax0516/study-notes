##  Socket

网络中进行通信的约定 Socket，一台计算机可以接受其他计算机的数据，也可以像其他的计算机发送数据

socket起源于Unix，而Unix/Linux基本哲学之一就是“一切皆文件”，都可以用 打开open -》 读写

write/read -〉 关闭close模式来操作

我的理解就是socket就是该模式的一个实现，即socket是一种特殊的文件，一些socket函数对其进行的操作（读写I/O， 打开，关闭）

Socket()函数返回一个整型的Socket描述符，随后的连接建立，数据传输等操作都是通过该实现的



TCP是通过Stream流进行传输的

udp数据报文

## 在线聊天系统

项目流程

![截屏2024-10-12 下午7.07.05](/Users/joy/Documents/GitHub/study-notes/C#/截屏2024-10-12 下午7.07.05.png)



## 服务端

创建一个winform应用程序

端口50000

![截屏2024-10-12 下午7.10.47](/Users/joy/Documents/GitHub/study-notes/C#/截屏2024-10-12 下午7.10.47.png)

```C#
namespace demo125socket
{
   public Form1()
   {
      InitializeComponent();
   }
   private void btnStart_Click(object sender, EventArgs e)
   {
      //创建一个负责监听的socket
      Socket socketWatch = new Socket(
          AddressFamily.InterNetwork //Ipv4
          ,SocketType.Stream, //以二进制的形式进行传输
          protocolType.Tcp  
      );
      //创建ip和端口对象
     IPAddress ip = IPAddress.Any //IPAddress.Parse(txtIP.Text);
     //创建网络节点
     IPEndPoint iPEndPoint = new IPEndPoint(ip, Convert.ToInt32(txtPort));
     //绑定我们的网络节点： 网络节点： ip+端口
     socketWatch.Bind(iPEndPoint);
     socketWatch.Listen(10);
     ShowMsg("启动成功");
     
     //创建一个负责监听的socket，来接受我们客户端的连接 创建和我们客户端进行通信的socket
     Socket socketSend = socketWatch.Accept(); //Accept
     
     ShowMsg("监听成功");
     
     Thread th = new Thread(Listen(socketWatch));
     th.Start(socketWatch);
   }
   void ShowMsg(string str)
   {
      txtMsg.AppendText(str);
   }
   void Listen(object o)
   {
     Socket socketWatch = o as Scoket;
     ShowMsg("监听成功");
   }
}
```





Tcp/udp

tcp三次握手，执行效率比较低

udp是不安全的协议，但是使用起来很快，不稳定，很容易数据丢失






