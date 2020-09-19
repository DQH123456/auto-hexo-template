---
title: Android制作双人联机游戏
date: 2020-05-20 14:25:02
---

> 项目有涉及到双人联机游戏的机制，这之前都没有做过类似的东西，现在什么都是从零开始，故开此文记录

<!--more-->

## 前言

我觉得的联机游戏流程是：
1. 点击“双人游戏”
2. 进入匹配
   - 匹配失败👉结束
   - 匹配成功👉进入3
3. 建立可靠连接
4. 系统出题，双方答题，根据对错实时更新ui

其中2，3是最重要的点，所谓可靠链接，难点在于怎么样使两个真机通过服务端链接，这时我想到了socket ，这就涉及到网络中怎么传输数据的问题了



由于游戏的性质我还要考虑以下问题

- 怎么保证题目的一致性
- 如何自动匹配
- 怎么保证A方传的数据到了B方那里？
- 怎么保证一个客户端就匹配另一个客户端，并且不会被第三个客户端匹配到（保证一一对应）
- 互相之间要传什么数据

额 ，好乱啊，虽然一瞬间考虑了那么多，但还是觉得无从下手。

昨天又想到了一些，对于如何自动匹配，我想到的是，如果客户端有传friendID的话，则于这个friendID进行匹配，如果没有的话，则进行自动匹配。好了，思考了这么多，该动手进行实现了。首先 ，是匹配机制的实现。

## 匹配机制

首先，每个用户都会有**专属的唯一的**一个id，我们可以以id进行标识，这大大方便了我们。

好了，想要匹配上，得先连接上服务器

### 进行简单连接测试

1.android客户端：在子线程中启用socket

```java
private void initSocket() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    clientSocket=new Socket(IP,PORT);//这个ip是电脑的ip，端口自己定的
                    if(clientSocket.isConnected()){//进行简单的测试
                        Log.e("成功！","已连接到服务器！！");
                    }
                    clientSocket.close();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }).start();
    }
```

2.java服务端,

```java
try {
			serverSocket=new ServerSocket(SERVER_PORT);
			System.out.println("服务端已启动，端口号是: "+SERVER_PORT);
			System.out.println("==========等待客户端连接中==========");
			while(true) {
				Socket socket=serverSocket.accept();
				System.out.println("客户端已连接！客户端的IP地址是:"+socket.getInetAddress());
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
```

ok 先打开服务端，后打开客户端，关掉windows防火墙，我们进行一下小测试

![服务端已启动](https://cdn.jsdelivr.net/gh/fushaolei/img/20200524155439.png)

然后打开android端。

![测试成功！](https://cdn.jsdelivr.net/gh/fushaolei/img/20200524155719.png)

已成功连接！

在给服务端传一个id试试。。

```java
try{
            DataOutputStream writer=new DataOutputStream(clientSocket.getOutputStream());
            writer.writeInt(userId);
        }catch (Exception e){
            e.printStackTrace();
        }
```

服务端进行接受

```java
reader=new DataInputStream(socket.getInputStream());
while(true) {
	int id=reader.readInt();
	System.out.println("获取到客户端的id是："+id);
}
```

同样也可以接受成功。

好了 ，接下来可以考虑匹配的问题了。

### 如何匹配

首先，封装成类，方便我们调用，然后，一个id唯一对应一个socket 又有唯一的friendID，在互传数据时要进行双向验证，也就是，在只有互相是对方的friendID的情况下，才能互发数据。
如何匹配？当有一个新的id进入时（游戏结束后会删除，也就是有一个新的游戏请求就是一个新的id），在自定义的类里进行循环，如果没有friendID则进行匹配，上面讲到了，实际上这里匹配的实质就是： 把friendID设置成对方的ID，不知道在这里你想明白没有。（其实这些是我边写此文时边想到的，写字果然能促进人的思考啊(。・∀・)ノ）

按照上面的思路，我们可以一步一步进行实现
#### 一，新建类
```java
package com;

import java.net.Socket;

public class SocketClass {
	private int id;//本体的id
	private int friendid;//待匹配id
	private Socket socket;//socket
	
	public SocketClass(int id,Socket socket) {//friendid默认是0
		this.id=id;
		this.socket=socket;
	}
	
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public int getFriendid() {
		return friendid;
	}
	public void setFriendid(int friendid) {
		this.friendid = friendid;
	}
	public Socket getSocket() {
		return socket;
	}
	public void setSocket(Socket socket) {
		this.socket = socket;
	}

	@Override
	public String toString() {
		return "SocketClass [id=" + id + ", friendid=" + friendid + "]";
	}
}

```

#### 二，新建匹配方法

放在获取客户端id的后面

```java
reader=new DataInputStream(socket.getInputStream());
while(true) {
	int id=reader.readInt();
	System.out.println("获取到客户端的id是："+id);
	socketList.add(new SocketClass(id, socket));//添加进List里去备用
	match();//由于是在while（true）循环里，所以会一直等待匹配
} 

```

匹配方法

```java
	//开始匹配
	public static void match() {
//		System.out.println(socketList.get(0).toString());//int默认值是0
		//大于或2才开始，否则就不能匹配咯
		if(socketList.size()>=2) {
			System.out.println("开始匹配前的状态");
			for(SocketClass mSocket:socketList) {
				System.out.println(mSocket.toString());
			}
			for(int i=0;i<socketList.size();i++) {
				if(socketList.get(i).getFriendid()==0) {
                    //friendId(int)的基本默认值是0，所以，如果等于0的话说明没有，则进行匹配操作
					for(int j=i+1;j<socketList.size();j++) {
						if(socketList.get(j).getFriendid()==0) {
                            //如果在此循环里，也有一个friendId为0的话，就进行两两匹配
							socketList.get(i).setFriendid(socketList.get(j).getId());
							socketList.get(j).setFriendid(socketList.get(i).getId());
						}
					}
					
				}
				
			}
			System.out.println("匹配后的状态");
		for(SocketClass mSocket:socketList) {
			System.out.println(mSocket.toString());
		}
		}
		
	}
```

测试一下，，匹配成功！！！

![匹配机制成功！](https://cdn.jsdelivr.net/gh/fushaolei/img/20200524193753.png)

喜大普奔。我们终于完成了匹配机制，到这里，在看看前言我所写的，我们现在只剩下最后一步了，接下来进入关于游戏ui更新的部分。

## 同步UI

这部分也蛮简单的（才怪~），A方答题后更新A方的UI，然后把A的回答传到服务器后传给B方以更新B方的UI，B方同样如此，重要的要保证题目一致，也就是，当每次答题开始时，服务端会向A方B方发送题目以及答案。

同步UI要走以下流程，为了简单考虑，就先不考虑有一方中途推出的情况了。

1. （与服务器建立连接）匹配后 服务器向双方发送对方的id，两客户端更新对方UI信息（头像啊，名字之类的）（如何保证只发送一次？）✅
2. 由其中一方向服务器发送信息 请求题目，服务器同时向双方发送题目以及答案✅
3. 双方客户端更新题目UI✅
4. A方答题后，更新A方UI ，并把回答发送到服务器转发给B方
5. B方接收后判断并更新UI，同上
6. 后答完题的人向服务端发送信息请求题目，重复以上

这里其实涉及到了一个传输问题，开始的时候（服务端）用了一些奇奇怪怪的方法，后来了解到json数据，如果key值不存在的话，取得的是null，这就大大方便了我们，以json作为传输对象也大大简化了逻辑。比如，想更新对方的ui信息时，服务端，传一个key为otherID值为19的json数据过客户端，然后客户端接受后判断key为otherID是否存在，若存在，则更新ui，若不存在，则执行其他逻辑，也就是可以以这个key作为判断条件，判断执行什么逻辑，例如更新题目ui啊，接受到对方的答案啊之类的