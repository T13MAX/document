# Card



## 总入口

实现CommandLineRunner接口重写run方法 异步启动Netty服务器

各个管理器初始化 调用init方法

异步启动 @Async@EnableAsync

### ThreadManager

```java
全局线程 房间管理线程 匹配管理线程 消息发送线程
init();启动的时候调用init方法 
使用全局线程执行一些东东 死循环执行(是不是应该支持shutdown???)
隔一段时间(几十毫秒)使用匹配线程执行MatchManager.matchDoing(); 房间线程执行RoomManager.update();
```

### HeroManager

```java
public static Map<Integer, Hero> allHero = new HashMap<>();	//所有武将的Map
public static List<Hero> mainHero = new ArrayList<>();		//所有主公的集合
public static List<Hero> notMainHero = new ArrayList<>();	//其他武将的集合
public static List<Hero> tempAll;
getMainSel()		//随机几个主公以供选择
getOtherSel(int exclude)
init()				//初始化所有英雄 id 名字血量等(不应该读表吗)
```

### CardManager

```java
// 一副卡牌,为 每个房间 增加牌堆使用
static List<CardItem> cardsList = new ArrayList<>(108);
// 每个 cid 对应的卡牌
static Map<Integer, ACard> cards = new HashMap<>();
init()		//创建所有类型的牌 放入cards
createCardList()	//创建牌堆 32张杀 18张闪 无中生有 无懈可击等等
newCardList(int index, List<CardItem> list) //根据index生成一组牌 index递增当id
get(int cid)  //根据cid获取一种牌 
```

### Hero

```java
id name 国家 是否主公 血
touchCard();	//摸牌 有的人可能 能摸三张
useSkill()		//使用技能
injuredSkill()	//受伤触发
hpDownSkill()	//掉血触发
beforeOverSkill()//弃牌
overSkill()		//出牌结束
equipSkill()	//使用武器技能
useKill()		//使用杀 
byKill() 		//受到杀
useYao() 		//使用药
byYao()			//受到药
die()			//死亡时
子类重写 每个人可能不一样
```



### ACard

```java
public int cid;                         // 卡牌配置id
canInitiative(Player player)			//是否主动使用
canPassive()	    					//是否被动使用
setResp()								//无目标卡牌 响应 设置
setRespHaveTarget()						//有目标
haveTarget()							//是否需要目标
use()									//使用
canResp()								//是否能响应
abandonResp()							//玩家放弃 响应
abandonShowCard()						//玩家放弃 出牌
addWxkj()    							//将拥有 无懈可击 的玩家加入列表,从 player开始
wxkjResp()								//被无懈可击 则执行
getEquipIndex()							//返回装备下标	(这不该写在card里吧...)
distance()								//计算距离   
    
子类重写
```

### Equip

```java
可以主动使用
使用后放入装备区 同一位置则替换 发消息
```



### MatchManager

```java
static Queue<Player> playerQueue = new ConcurrentLinkedQueue<>();//匹配队列
凑够5人的临时集合
响应状态/等人状态
响应最后时间戳

加入 离开匹配队列
matchDoing();   		//匹配方法 判断人齐不齐 该执行什么操作 
signPlayerResp();		//验证玩家是否全部接受 超时直接恢复状态 回到队列 禁止匹配 接受则使用房间线程池创建房间
sendMatchResp();		//通知玩家状态
匹配写的应该有问题 应该是凑够人放进一个临时房间 每个房间有一个自己的isResp 自己的超时时间 应该是简化了 
```

### RoomManager

```java
index			//房间id 自增
所有房间的容器		//List(要不要线程安全呢 感觉应该不够就创建 有空闲则使用空闲 给房间加一个状态重置
update();		//循环所有房间 使用房间线程池执行每个房间的update()
createRoom(List<Player> list)	//获取房间 调用init方法 
getRoom();		//找一个空闲房间 或者新建一个
```

### Room

```java
房间id
当前房间的处理线程
房间阶段
玩家列表
当前出牌的玩家
出牌结束时间戳
牌堆
index -> id分配 
当前出的牌  
需要响应的玩家列表
需要救的玩家
无懈可击列表
Send
Receiver
五谷丰登的牌列表
借刀杀人 被杀的目标
寒冰剑消息    
感觉字段写的很墨迹呢...
init();		//初始化 创建集合 XX对象等 初始化玩家的手牌管理类 debuff列表 装备列表 最后修改房间状态为init
update()    //根据房间状态判断现在要做什么 看ERoomStatus 
sendWin()	//给玩家发送胜利失败 参数是身份 谁赢了 然后判断每个人要
gameOver()	//判断哪一方胜利 发送胜利消息 改变房间状态
getPlayerById()//竟然循环遍历取玩家 (虽然不多,但是Map不好么)
setNextShowCard()//设置下一个出牌的玩家 修改当前玩家的状态
excludeDie(List<Player> ref)//排除引用中,死亡的玩家

感觉有些地方可以用Map存
```

### ERoomStatus

```java
房间状态	
    
Init,// 初始化阶段. 
1.初始化身份 根据人数获取哪些身份 然后随机分配
2.通知所有玩家自己的身份和其他玩家信息??? (没必要吧???怎么能啥都发呢)
3.通知主公选将 其他玩家等待 设置超时时间戳 改变房间状态(下次进来就是去主公选将的分支了)
    
MainSel,// 主公选择武将阶段  
1.判断主公是否已经选了将 则判断是否超时 超时则随机选择
2.向其他玩家发送选将消息 向主公发送等待消息 改变状态 通知等等
    
OtherSel, // 其他人选择武将阶段   
1.判断是否有人没选 有人没选且没超时则return 否则随机选择 
2.通知说有人武将信息 
3.初始化血量 发牌 改变房间状态
    
Doing,// 出牌阶段
#见EPlayerGameStatus
    
CloseIng, // 结算阶段
1.通知客户端,可以离开房间了,并弹出 结算信息
2.清空房间数据 重置状态
3.通知结算 
    
Close,     // 关闭状态,清除房间所有数据 (写错了吧 这个状态明明啥也没干)

```

### EPlayerGameStatus

```java
BeforeDrawCard,         // 摸牌之前的状态判断阶段
查看玩家状态 有乐不思蜀 兵粮寸断 要通知开始无懈可击
多种情况:
1:第一次进来,通知他人出 无懈可击,
2:是否处于他人无懈可击 状态
3:已经不是 无懈可击了,buff是否生效, 
感觉没写完 应该是分情况 等待响应无懈可击 谁无懈了谁的无懈 最终生不生效
    
DrawCard,               // 开始摸牌
兵粮寸断不摸牌 断粮摸一张(玩家技能吧)
乐不思蜀则直接跳过出牌 进去弃牌阶段
    
ShowCard,               // 出牌阶段
出牌
AwaitResp,              // 等待他人响应阶段
AwaitRescue,            // 有人死亡,等待救援
NearDeath,              // 自己濒死,等待救援
Over,                   // 结束出牌,如果手牌大于生命,需要弃牌
弃牌
OverResp,               // 出牌结束后,部分武将 可能会触发技能
Die,                    // 死亡,可以离开游戏了
```

### Player

```java
uid 账号 密码 昵称
匹配状态是否响应
房间
出牌阶段
身份
所选武将
血
最大血
临时数据 playerInfo
手牌管理器PlayerCardManager
buff信息debuffInfo
装备信息equip
下一个玩家
是否使用酒
是否出过杀
info() 		  //深拷贝一个Player
getCardNum()	//获取手牌数量
initHp()		//初始化血量 主公+1
```



## Util

### RandomUtil

```java
随机数工具类
```

### SpringBeanUtil

```java
implements ApplicationContextAware		//实现ApplicationContextAware setApplicationContext方法set context
getBean(Class<T> clazz)    				//获取bean
```

### Send

```java
Room
sendHeroInfo()		//通知所有人 武将选择信息
sendOtherSelMsg(Player player, List<Hero> heroes)	//通知其他人选择武将
dealToPlayer()		//给某个玩家发
sendMsgToAll()		//将某个消息推送给所有人
sendShowCardMsg()	//广播当前出的牌
sendHelp()			//濒死
onPlayerHpUpdate()	//玩家血量变化
sendRespShowCard()	//发送响应当前出的牌
sendAllDiscard()	//弃牌消息
sendBuffUpdate()	//发送buff变化
```



### Receiver

```java
PlayerSelHero()			//选择武将
showCardHavePlayer()	//玩家出牌
...    
收客户端的消息
```



## Net

### NetConnManager

```java
连接管理类
static Map<ChannelHandlerContext, NetConn> ctxs = new HashMap<>();    //存连接
static Map<Integer, NetConn> players = new HashMap<>();					//id->NetConn 里面是上下文和玩家
获取 增加 删除
```

### NetConn

```java
上下文和玩家
send(NetMessage msg)->ctx.writeAndFlush(msg);
```



### NetMessage

```java
所有协议顶级父类
code 
用户名密码(不是吧 这个也要写?)
昵称(没必要吧)
通用参数 (真通用)
toJson()		//序列化为json字符串    
```

### MessageDispatch

```java
总入口 switch case 处理请求 调用service
```



### BootNettyServer

```java
NettyServer
编解码器 叭叭叭 那一套
```



### BootNettyChannelInboundHandlerAdapter

```java
Handler那一套 叭叭叭
```



### MyDecoder

```java
解码器
不足4字节返回0 足则读一个int是长度 然后readBytes
```



### MyEncoder

```java
编码器
在最前面插入一个数组长度 
采用小端方式编入int 把前面8位写在后面???
```

