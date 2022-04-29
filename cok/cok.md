# COK

```java
package com.elex.cok.core;				//core 前端请求Request DbManager GameEngine

package com.elex.cok.gameengine.army;	//兵
兵 医院 陷阱
package com.elex.cok.gameengine.chat;	//聊天
package com.elex.cok.gameengine.cross;	//跨服相关?
package com.elex.cok.gameengine.mail;	//邮件
package com.elex.cok.gameengine.pay;	//支付
package com.elex.cok.gameengine.ranking;//排行
package com.elex.cok.gameengine.skill;	//技能
package com.elex.cok.gameengine.territory;//联盟领土相关
package com.elex.cok.gameengine.timedPush;//推送
package com.elex.cok.gameengine.alliance; //联盟

package com.elex.cok.handlers;			//
package com.elex.cok.Modules;			//模块?
package com.elex.cok.puredb;			//0.0
package com.elex.cok.rank;				//排行榜
package com.elex.cok.trigger;			//trigger
package com.elex.cok.utils;				//utils

UserBuilding
UserResource
```



```java
Multimap<Long,Integer>  //谷歌的集结 Map<Long,List<Integer>>
```



model -> 模型	module->模块

# --------------------------------------------------------------------------------------------------

## Game

### GameEngine

```java
```

### GameModuleManager

```java
```

### FunctionOnController(新功能开关控制类)

```java
```

### COKServer

```java
```



## 地图

### World

```java
地图常量;
锁名常量;
各种包含WorldMarch的集合;					
	srcMarchRecordMap	dstMarchRecordMap// src key是拥者点id  dst key是目标点id
    allianceMarchRecord allianceBeMarchRecord	//key是联盟id
    globalWorldMarchMap						//key是uuid
各种定时任务;
LinkedBlockingQueue<Integer> monsterFlushAreaQueue;	//怪刷新队列
ConcurrentHashMap<String, Object> worldFlushLock;	//刷新锁	lock isLock unLock getLock 方法

World()									//构造方法 初始化线程池 创建集合 内部类创建含有实例
init()->WorldBornManager::init() 		//GameEngine::initWorld()调用 初始化联盟 王国 军队 更新WorldPoint 跨服相关 不同服务器的一些处理
addMarchRecord() 
startFlushResourceTimer()				//资源刷新的定时任务
getWorldInfo()							//获取地图信息
findNearearestPoint()					//某个点最近的迁城点	还有XX战场的出生点
XX战场相关操作		
noticeWorldUsers()						//同联盟 可视范围内 pushGroupMsg()
getUsers()								//视野范围内玩家
getRangeMarchRecord() getAllMarchRecord()	//范围内所有行军  所有行军
各种对几个行军的操作
```

### UserWorld(玩家的有关地图的信息)

```java
世界id
体力值
野怪
迁城
跨服
出征
initForWorldCup() initForAncesTrial() initForRobot() UserWorld().onRegister() 	//finally解锁地块  
onLogin()					//登录处理 军队加入内存 被攻击更新城防等 修正主点
returnHomeIfBadMarch()		//有问题行军数据处理
moveCity()					//迁城    注意锁 备份数据 用于回滚 军队处理 通知盟友
玩家行军相关    
    体力
总之就是这个玩家在地图上的一些业务逻辑
```

### WorldPoint

```java
PointType 点类型 
各种存表用的字段 
    
field相关操作 设置 取值 初始化fieldSet
create() 根据传入的一个东东赋值创建对象 gson似的 xx.get("xx").getInt()
isXxx()			是否为XXX
change2()		变成XXX
flushCache()	刷新缓存    
```

### PointManager

```java
List<WorldPoint> pointList;			全部点
按区域 按areaId 按地图id存的Map		
ScheduledExecutorService timer;
ForkJoinPool updaterPool;  

selectAllPoints()	根据世界id获取所有点   
loadData()			加载东东 起一个单线程的任务不断执行 调用 updateWithForkJoin()
updateWithForkJoin(); WorldPoint继承了Statable 是否要更新啥的
clean()				删除点 关闭定时任务
reload()			重新加载 重新设置值 不改变引用
searchByPointType()	随机点 把范围内的全弄出来 然后随一个
```

### WorldPointUpdater

```java
extends RecursiveTask<UpdateResult<WorldPoint>>				//ForkJoinTask是ForkJoinTask的子类
compute()			创建两个WorldPointUpdater 然后merge
update()			先批量 失败在逐一 和数据库有关
看看数据库相关操作 ForkJoinTask的玩法
```

### WorldBornManager(玩家出生点相关)

```java
private static final String WORLD_BORN_KEY = "world_born_" + Constants.SERVER_ID;	//生成用的前缀
各种出生点 出生点锁
redisMap		//redis的map? 内存也存一份???
```

### WorldEffectManager

```java
效果Map
线程池

```

### WorldCreatorV2

```java
```



### package com.elex.cok.gameengine.world.task;

```java
各种任务;
清除死号;

```



### Other

WorldPointLock  [跳转到锁](#锁)

[task](#package com.elex.cok.gameengine.world.task;)

## 玩家

### UserProfile

```java

```

### UserBuilding

```java
```

### UserState

```java

```

### Other

```java
UserAuth
```



## 兵 package com.elex.cok.gameengine.army;	

### UserArmyManager

```java
armyMap缓存兵数据 onLogin()的时候加载
hospitalManager 医院相关 构造方法创建
wallManager		陷阱相关 构造方法创建
userProfile		玩家相关 构造方法传入
    
onRegister(SqlSession sqlSession, String uid) 递归调用其他manager的onRegister()
addComplete(String armyId, String uuid)  收获士兵  
收获士兵有锁 其他没看到0.0 
```

### UserArmy

```java
armyid;
uid;
free;//空闲的兵
march;//  在外的兵(出征/驻守资源等等，总之不在家里)
train;//正在训练的兵
兵操作的相关方法
兵数据库操作的方法
```

### UserHospitalManager

```java
Map<String, UserHospital> hospitalMap;
UserProfile userProfile;
Map<UserResource.ResourceType, Long> resourceCost;	临时字段 资源消耗
    
getLoginInfo() 登录时发来的请求 校验离线期间完成的东东 比如治疗完成
```

### UserHospital

```java
armyid;
uid;
dead;//未治疗的兵
heal;//正在治疗兵
finishtime;//治疗结束时间
```

### UserGeneralManager(将军管理)

```java
private Map<String, UserGeneral> generalMap;	//缓存
private Set<String> genIdSet;					//id列表
private UserProfile userProfile;				//玩家信息
```

### UserGeneral(将军)

```java
星级 等级 颜色 状态 天赋 技能 等
generalEffect英雄的作用号效果
```



## 行军 (package com.elex.cok.gameengine.world.march;)

模板方法模式 一个抽象行军类 定义march() check() checkOccupy() marchSuccessfully() checkActivity() checkSoldier()等相关方法 不同出征写不同实现类 重写方法

### WorldMarchGeneralArmy

```java
private List<General> generalList;	// 对应UserGeneral, 其实就是领主
private List<Soldier> soldierList;	// 士兵列表
private WorldPoint.PointType targetPointType;//目标格类型
private int targetPointId;			//目标格id
private Map<Integer, Float> baseExtraEffects;// key=effectId  value=效果值
public WorldMarchGeneralArmyExtra extra;//额外信息
private int baseSoldierLimit;//基础出征上限
private long armyPower; //玩家的军队战力

WorldMarchGeneralArmy() 构造方法根据玩家的士兵领主创建内部类的士兵领主对象
initExtra()	//初始化额外属性
    
校验 兵数 添加 减少 计算行军时间 计算获取负重
```

### General&&Soldier&&ArmyType

```java
General&&Soldier领主和士兵 
ArmyType兵种类型 BasicArmType基础兵种	//射手也分弓兵弩兵 功臣武器分冲车投石车
```

### WorldMarchGeneralArmyExtra

```java
protected List<GeneralExtra> generalList;	//领主额外信息列表
protected List<FightDragonExtra> dragonList;//龙
protected List<SoldierExtra> soldierList;	//士兵
各种效果Map
```

### AbstractWorldMarch

```java
UserProfile userProfile;	玩家信息
WorldPoint targetPoint;		目标点
WorldMarchGeneralArmy wmGenArmy;军队
marchType 					出征类型
WorldMarch worldMarch;		行军实体
  
march(){
    checkMarch();		//检查能否出征
    setWorldMarch();	//初始化worldMarch信息
    updateWorldMarch(); //更新出征纪录
    worldMarch.setMarchFuture(marchFuture);定时任务
    onMarchSuccessfully();//出征成功后的处理
}
一些其他方法 不在模板里 但是也是不同实现
```

### WorldMarch

```java
唯一Id
拥有者相关的信息
联盟id
WorldMarchGeneralArmy genArmyObj;
开始时间 到达时间 行军时间等
世界id
采集 怪物等信息
各种Future
锁数量

joinScheduledQueue()	//起服的时候把行军拉起来 该干啥还去干啥 启动定时任务 设置Future
lock() unlock()	getLockObj()//锁/解锁一个点 获取锁对象 把一个对象往线程安全的Map里放 putIfAbsent()
checkAllianceMarch()	//是否为联盟出征 就是校验目标点的类型
canSpeedUp()			//能否加速 有些出征类型不能加 被锁不能加
retreat()				//部队返回 取消定时任务 添加回城定时任务
掠夺相关处理方法()		   //
采集相关()	setOccupyFuture() updateAllMarchTime()			 //占领结束 改变采集速度 最大数量 采集时间
pushMarch() 			//推送行军消息 GameEngine.getInstance().pushMsg()
noticeWorldUsers()
future处理
updateAllMarchTime()	//更新采集时间 World::init调用   
...待补充
```

### WorldMarchRoute

```java
计算行军时间
```



### MarchType&&MarchState

```java
MarchType 行军类型 尽可能详细;
MarchState行军状态 行军/占领/返回;

```

### AbstractWorldOccupiedTarget(占领者)

```java
```



## 联盟

### AllianceManager

```java

```

### AllianceHospitalManager

```java
```

### AllianceMember

```java
```

```
TerritoryMiniFort
TerritoryTower
```

## 锁

### ConcurrentLock(全局锁类 )

```java
ConcurrentHashMap<String, Object> lock = new ConcurrentHashMap<>()存锁对象 里面放的不是Lock 只是Object对象 作为锁对象
```



### WorldPointLock

```java
地图点锁 private ConcurrentHashMap<Long, Object> lock;用线程安全集合存 能放进去就成功了 有一个不行就回滚
```



### 战斗

### IBattleTeam

```java

```



## DB

### COKDbManager

```java
```



### SFSMysql

```java
```



```java
SessionUtil
    
```



### 处理战斗和一些任务(package com.elex.cok.gameengine.world.normal)

模板方法模式 AbstractWorldFightHandler 处理各种战斗 不同战斗不同实现类



### 定时任务(package com.elex.cok.gameengine.world.task;)



### 处理前端请求(package com.elex.cok.handlers)



### Constants

常量类

Collections.unmodifiableMap(new HashMap<String, String>() {{ put("1", "1");}})                                                                                         

### GameStat

每个启动花费的时间



### QueueManager Queue

各种队列 可以连续 一个队列 完成一个自动开始下一个UserArmyManager::addComplete 1828行

### onRegister onLogin

一层一层调 初始化 登录



## 记事本

```
UserSkill
玩家信息里存了队列(玩家的各种队列 如建造队列 最多两个)
治疗士兵 也是和收获资源差不多 等领取再加上 没有定时器









```





































































































































































































































































