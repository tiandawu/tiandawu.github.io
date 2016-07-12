---
title: Android中实现自定义XMPP消息包收发
date: 2016-06-05 09:46:14
tags: XMPP
categories: Android
comments: true
banner: http://i.imgur.com/AlXNlDS.png
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Android平台实现XMPP即时通讯主要是使用asmack这个包，asmack是XMPP协议的实现。但是asmack只能帮助我们实现一些基本消息包的收发，如果需要实现特定的自定义消息包收发需要我们自己处理。
<!--more-->
***
## 一、asmack消息的发送和接收

- **发送Message消息：**  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;发送一个message结的消息，可以使用sendMessage()发送消息，这个方法有两个重载方法，一种类型的参数是String类型，另一种则是传入Message对象。String类型的方法传入的字符串即为要发送的消息；传入message对象的方需要写一个类继承Message，重写`toXML()`方法，`toXML()`方法的返回值即为要发送的消息。例如：  
```java

	//1、通过传入String类型的sendMessage()方法发送消息：

	ChatManager chatManager = xmppConnection.getChatManager();
	 /**
       	 * String userJID 对方的JID
         * MessageListener listener 消息监听，当收到消息后会回调processMessage(Chat chat, Message message)方法
         */
	Chat mChat = chatManager.createChat(mToUser, this);
	mChat.sendMessage("your content");
	//2、通过传入Message对象的sendMessage()方法发送消息：

	/**
	*写一个类继承Message重写toXML()方法，方法的返回值即为要发送的消息
	*/
	public class MyMessage extends Message {
		
		 @Override
	    public String toXML() {
			
			return "your content";
		}
	
	}

	ChatManager chatManager = xmppConnection.getChatManager();
	 /**
       	 * String userJID 对方的JID
         * MessageListener listener 消息监听，当收到消息后会回调processMessage(Chat chat, Message message)方法
         */
	Chat mChat = chatManager.createChat(mToUser, this);
	MyMessage myMessage = new MyMessage();
	mChat.sendMessage(myMessage);

```

- **接收Message消息：** 
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接收Message类型的消息主要是在`processMessage(Chat chat, Message message)`方法中，当收到消息后都会回调这个方法，需要实现`MessageListener`这个接口，然后实现接口中的`processMessage(Chat chat, Message message)`方法。
***
## 二、发送和接收自定义类型的IQ结消息
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当发送的iq消息中包含自定义的xml结的时候，asmack提供的解析并不能识别这些自定义的xml结，所以就需要我们自己实现消息包的解析和拼装。 

1、发送含自定义xml结的iq消息：  
例如发送这样一个iq消息：`<iq id='123' type='get' from='client@xmpp/B' to='client2@xmpp/s2'><req var='read'><attr var='temprature'/></req></iq>`  

```java

	//步骤：
	//1、写一个类继承IQ并重写getChildElementXML()方法，该方法的返回值将作为消息体。
	public class MyIQ extends IQ {
		@Override
	    public String getChildElementXML() {
			StringBuilder stringBuilder = new StringBuilder();
			stringBuilder.append("&lt;req var='read'&gt;&lt; attr var='temprature'/&gt;&lt;/req&gt;");
			return stringBuilder.toString();
		}
	}
	//2、发送这个含自定义xml结的iq消息包
	MyIQ packet = new MyIQ();
	packet.setType(IQ.Type.GET);//设置IQ结type
        packet.setFrom("client@xmpp/B");//设置IQ结from
        packet.setTo("client2@xmpp/s2");//设置IQ结to
        xmppConnection.sendPacket(packet);//发送消息包
```

2、解析服务器返回的iq消息包，消息包中含自定义xml结：  
例如解析服务器返回的这样一个iq消息：`<iq id='12' type='result' from='client2@xmpp/s2' to='client@xmpp/B'><resp xmlns='data'><attr var='temprature'>17</attr></resp></iq>`  

```java

	//步骤：
	//1、写一个类 implements PacketListener接口并实现其中的processPacket(Packet packet)方法。
	public class MyPacketListener implements PacketListener {
	 	@Override
    	public void processPacket(Packet packet) {
			//当收到消息包就会回调该方法
		}
	}
	//2、添加包监听器
	MyPacketListener mMyPacketListener = new MyPacketListener();
	//该方法有两个参数
	//第一个参数：	PacketListener  包监听器
	//第二个参数：   PacketFilter  包过滤器
	xmppConnection.addPacketListener(mMyPacketListener, null);
	//完成以上两步后，当收到消息包都会回调MyPacketListener中的processPacket(Packet packet)方法。
	//3、写一个类继承IQ并实现getChildElementXML()方法。
	public class GetDataResp extends IQ {
		//例如我们要获取上面iq消息包中的temprature和17两个属性,所以将这两个值声明为成员变量,并生成get和set方法。
		public String var;
		public String value;
		public String getVar() {return var;}
		public void setVar(String var) {this.var = var;}
		public String getValue() {return value;}
		public void setValue(String value) {his.value = value;}
		@Override
		public String getChildElementXML(){
			//拼装消息
			StringBuilder buf = new StringBuilder();
			buf.append("&lt;resp xmlns='get:data'&gt;&lt;attr var='");
	·		buf.append(getVar());
			buf.append("'>");
			buf.append(getValue());
			buf.append("&lt;/attr&gt;&lt;/resp&gt;");
			return buf.toString();
		}
	}
	//4、写一个类implements IQProvider并实现接口中的parseIQ(XmlPullParser parser)方法。
	public class GetDataRespProvider implements IQProvider {
		@Override
		public IQ parseIQ(XmlPullParser parser) throws Exception {
			GetDataResp getDataResp = new GetDataResp();//这个对象是上面第三步中的那个类对象
			boolean done = false;
			while (!done) {
				int eventType = parser.next();
				if (eventType == XmlPullParser.START_TAG) {
					if (parser.getName().equals("attr")) {
						String var = parser.getAttributeValue("", "var");//获取var属性的value即：temprature
						String value = parser.nextText();//获取attr的文本即：17
						getDataResp.setVar(var);
						getDataResp.setValue(value);
					}
				}else if (eventType == XmlPullParser.END_TAG) {
					if (parser.getName().equals("resp")) {
						done = true;
					}
				}
			}
			 return getDataResp;
		}
	}
	//5、在配置ConnectionConfiguration时添加IQProvider
	//第一个参数是：String　元素的名称
	//第二个参数是：String  命名空间
	//第三个参数是：Object  需要传入一个prvider对象
	ProviderManager.getInstance().addIQProvider("resp", "data", new GetDataRespProvider());
	//6、在第一步MyPacketListener中的processPacket(Packet packet)方法中获取相应消息包
	public class MyPacketListener implements PacketListener {
	 	@Override
    	public void processPacket(Packet packet) {
			if (packet instanceof GetDataResp) {
				GetDataResp getDataResp = (GetDataResp) packet;
				String from = getDataResp.getFrom();
				String to = getDataResp.getTo();
				String var = getDataResp.getVar();
				String value = getDataResp.getValue();
			}
		}
	}
```
***
## 三、总结
- **发送message类型的消息中如果带有自定义xml结,需要写一个类继承Message并重写`toXML()`方法，该方法的返回值便是消息体。**
- **发送的iq类型的消息中如果带有自定义xml结，需要写一个类继承IQ并重写`getChildElementXML()`方法，该方法的返回值将作为消息体**
- **服务器返回的iq消息类型中如果带有自定义的xml结：**
	- 写一个类继承IQ并重写`getChildElementXML()`方法，将服务器返回的消息中需要的信息做成成员变量，并拼装出消息体，最后作为返回值返回。
	- 写一个类implements IQProvider并实现接口中的parseIQ(XmlPullParser parser)方法，然后在该方法中做出对应的解析过程，最后通过返回值返回上一个步骤中的IQ对象。
	- 服务器返回的iq消息中的消息体必须带有命名空间。
	- 需要通过这个方法`ProviderManager.getInstance().addIQProvider("resp", "data", new GetDataRespProvider());`添加相应的IQProvider。
- **如果需还需要实现一些自定义的解析，可以修改asmack源码中的`PacketParserUtils`这个类中对应的方法。**  

***
### 下面是源码地址：  
点击：[<font color=#0000FF>下载地址</font>](https://github.com/tiandawu/IotXmpp)