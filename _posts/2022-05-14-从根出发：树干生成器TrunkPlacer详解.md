---
layout: post
---


------



![image-20220513235025394](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220513235025394.png)

> 测试时偶遇的一棵优美的大型橡树。

<br />

> “我知道大多数玩家都对树干感兴趣，但是那些留在空中的树叶真的太丑了。”
>
> “我刚刚特地通知了基岩版那边的人，让他们为此加一条标语警告玩家。你给个建议？”
>
> “不要让树漂浮”
>
> ——也许是Mojang某两位程序员的对话

<br />

### TrunkPlacer的继承树

```
AbstractTrunkPlacer
	|__ForkyTrunkPlacer		//金合欢树
	|__FancyTrunkPlacer		//大型橡树
	|__GiantTrunkPlacer		//巨型云杉树
	|	|__MegeJungleTrunkPlacer	//大型丛林树
	|
	|__DarkOakTrunkPlacer	//深色橡树
	|__StraightTrunkPlacer	//未提到的所有树，包括普通橡树，白桦树，小型丛林树，普通云杉树
```



### 从抽象到具体——AbstractTrunkPlacer简析

这段代码比较长，可能不适于阅读，这里只挑出构造器、成员变量和其他成员方法的方法声明来介绍，方法体就隐去了。

```java
public abstract class AbstractTrunkPlacer {
   public static final Codec<AbstractTrunkPlacer> CODEC = Registry.TRUNK_PLACER_TYPES.dispatch(AbstractTrunkPlacer::type, TrunkPlacerType::codec);
   public final int baseHeight;		//最低高度
   public final int heightRandA;	//高度随机数A
   public final int heightRandB;	//高度随机数B

   protected static <P extends AbstractTrunkPlacer> P3<Mu<P>, Integer, Integer, Integer> trunkPlacerParts(Instance<P> p_236915_0_);	//作用不明

   public AbstractTrunkPlacer(int p_i232060_1_, int p_i232060_2_, int p_i232060_3_) {
      this.baseHeight = p_i232060_1_;
      this.heightRandA = p_i232060_2_;
      this.heightRandB = p_i232060_3_;
   }//构造器

   protected abstract TrunkPlacerType<?> type();//返回TrunkPlacerType，这与TrunkPlacer的注册有关，以后细说

   public abstract List<FoliagePlacer.Foliage> placeTrunk(IWorldGenerationReader p_230382_1_, Random p_230382_2_, int p_230382_3_, BlockPos p_230382_4_, Set<BlockPos> p_230382_5_, MutableBoundingBox p_230382_6_, BaseTreeFeatureConfig p_230382_7_);//最重要的也是唯一的抽象方法，实现类会在里面写树干生成算法

   public int getTreeHeight(Random p_236917_1_);//根据最低高度和两个随机数，算出一个最终树干高度

   protected static void setBlock(IWorldWriter p_236913_0_, BlockPos p_236913_1_, BlockState p_236913_2_, MutableBoundingBox p_236913_3_);//特殊的setBlock方法，只有放置横向原木的大型橡树算法调用过它，我们可以忽略

   private static boolean isDirt(IWorldGenerationBaseReader p_236912_0_, BlockPos p_236912_1_);//判断本方块是不是Dirt类的方块

   protected static void setDirtAt(IWorldGenerationReader p_236909_0_, BlockPos p_236909_1_);//将所选位置变为泥土方块（注意，是泥土）

   public static boolean placeLog(IWorldGenerationReader p_236911_0_, Random p_236911_1_, BlockPos p_236911_2_, Set<BlockPos> p_236911_3_, MutableBoundingBox p_236911_4_, BaseTreeFeatureConfig p_236911_5_);//放置单块原木并将其坐标加入一个“原木坐标集合”中

   protected static void placeLogIfFree(IWorldGenerationReader p_236910_0_, Random p_236910_1_, BlockPos.Mutable p_236910_2_, Set<BlockPos> p_236910_3_, MutableBoundingBox p_236910_4_, BaseTreeFeatureConfig p_236910_5_);//只是添加了一层检查的placeLog方法，只有大型丛林树和巨型云杉树在用

}
```

这里最重要的方法：**placeTrunk**

希望大家对placeTrunk留下深刻的印象，这是一切TrunkPlacer实现类工作的基础。当然，type()方法也需要留意，最后我们会专门处理这个问题。



### 从具体到抽象——以大型丛林树为例

大家可能会问，为什么不从结构简单的树，比如小型橡树和白桦树开始谈，而要从结构复杂的丛林树说起呢？

答：因为它的算法更好地体现了作为一棵树的本质。（金合欢树：我不服！但是本人就是在研究大型丛林树的代码时突然悟了）

还记得上一篇开头说的这句话吗？

> Minecraft中的树自然不是数据结构与算法中那些令人望而却步的树，但它们的本质逻辑是高度相似的。

数据结构里的一棵树可能是这样的：

![image-20220514092836072](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514092836072.png)

但是mc中的一棵丛林树是这样的：（左一）

![img](https://static.wikia.nocookie.net/minecraft_zh_gamepedia/images/d/d9/Jungle_Trees.png/revision/latest/scale-to-width-down/702?cb=20170303222523)

你一定会说这两个东西简直毫无联系，但你看了源码后一定就不会这样想了，反而会相当佩服Mojang程序猿的设计和他们的脑洞。

上文说过placeTrunk方法很重要，它是个抽象方法，每个非抽象实现类都会覆写它。大型丛林树的树干生成器是MegaJungleTrunkPlacer，它自然也会覆写这个方法。

别急，在正式看源码之前，我先解释一下这个方法的原型。你看，这么多参数！还都是srg名！

```java
public abstract List<FoliagePlacer.Foliage> placeTrunk(IWorldGenerationReader p_230382_1_, Random p_230382_2_, int p_230382_3_, BlockPos p_230382_4_, Set<BlockPos> p_230382_5_, MutableBoundingBox p_230382_6_, BaseTreeFeatureConfig p_230382_7_);//最重要的也是唯一的抽象方法，实现类会在里面写树干生成算法
```

- 返回值：List，泛型是FoliagePlacer的内部类Foliage，有序地存放了每个叶子节点（原谅我用了数据结构的名词，但它实在很贴切）
- 参数
  - IworldGenerationReader，类似于平时常用的World类，它也可以调setBLock方法，负责向世界中放方块以及读取方块状态
  - Random：一个随机数
  - int：经一定算法得出的**树干**最终高度，考虑过方块遮挡的因素。比如在狭小空间中种的树很可能比露天种的树矮，就是这个数在起作用。
  - BlockPos：树的起始生成位置，一方块的树则是【泥土上的第一块原木】，4方块的树则是【泥土上的靠西北角的第一块原木，即x、z都最小，这样的规定与区块生成顺序有关】，嗯，没错，你可以把它理解成根节点
  - Set<BlockPos>：原木坐标集合，存放了树上**每一块**原木的坐标，比如一棵很高的大型丛林树含有128块丛林树原木（我在wiki找了一圈，唯独没有丛林树的资料，此数据仅为猜测），那么这个Set里就会存128个坐标。这些坐标在以后会有用处。
  - MutableBoundingBox：整个树结构的边界箱，记得你摆弄结构方块时看到的那个白色边界吗？这个也差不多。
  - BaseTreeFeatureConfig：这是专门对付**Feature中的TREE**的Config，它负责把TREE转化成各种变种。里面配置了一些信息，以后会详细讲解里面有什么。



好了，我们已经理解了placeTrunk方法的每一个参数，接下来正式看代码。关键的地方有注释。

```java
 public List<FoliagePlacer.Foliage> placeTrunk(IWorldGenerationReader p_230382_1_, Random p_230382_2_, int p_230382_3_, BlockPos p_230382_4_, Set<BlockPos> p_230382_5_, MutableBoundingBox p_230382_6_, BaseTreeFeatureConfig p_230382_7_) {
      List<FoliagePlacer.Foliage> list = Lists.newArrayList();//临时List，存放这次得到的叶子节点
      list.addAll(super.placeTrunk(p_230382_1_, p_230382_2_, p_230382_3_, p_230382_4_, p_230382_5_, p_230382_6_, p_230382_7_));
     //将父类placeTrunk得到的叶子节点存入自己的List，以下详解

     //从最高的那块原木下方2~6格出发，向下走，每次下降2~6格，下降到树高度的一半处停止循环。得到一个侧枝基部坐标。
      for(int i = p_230382_3_ - 2 - p_230382_2_.nextInt(4); i > p_230382_3_ / 2; i -= 2 + p_230382_2_.nextInt(4)) {
         float f = p_230382_2_.nextFloat() * ((float)Math.PI * 2F);//确定球坐标偏角
         int j = 0;
         int k = 0;

          //生成侧枝。这里运用了一些球坐标知识，等会儿细讲
         for(int l = 0; l < 5; ++l) {
            j = (int)(1.5F + MathHelper.cos(f) * (float)l);
            k = (int)(1.5F + MathHelper.sin(f) * (float)l);
            BlockPos blockpos = p_230382_4_.offset(j, i - 3 + l / 2, k);
            placeLog(p_230382_1_, p_230382_2_, blockpos, p_230382_5_, p_230382_6_, p_230382_7_);
         }

          //一根侧枝生成完后，使用侧枝末梢坐标，实例化一个叶子节点，放入临时List中
         list.add(new FoliagePlacer.Foliage(p_230382_4_.offset(j, i, k), -2, false));
      }

      return list;//返回临时List
   }
```

父类GiantTrunkPlacer的placeTrunk分析：

```java
public List<FoliagePlacer.Foliage> placeTrunk(IWorldGenerationReader p_230382_1_, Random p_230382_2_, int p_230382_3_, BlockPos p_230382_4_, Set<BlockPos> p_230382_5_, MutableBoundingBox p_230382_6_, BaseTreeFeatureConfig p_230382_7_) {
    //将2*2原木下方4格从草方块变为泥土
      BlockPos blockpos = p_230382_4_.below();
      setDirtAt(p_230382_1_, blockpos);
      setDirtAt(p_230382_1_, blockpos.east());
      setDirtAt(p_230382_1_, blockpos.south());
      setDirtAt(p_230382_1_, blockpos.south().east());
    //临时变量
      BlockPos.Mutable blockpos$mutable = new BlockPos.Mutable();

    //生成2*2主干，从【泥土上的靠西北角的第一块原木，即x、z都最小的】那块开始，每层生成4块原木，不断向上生成，直到树干最大高度
    //当生成最顶上一层原木时，只生成西北角的那块，其他三块不生成
      for(int i = 0; i < p_230382_3_; ++i) {
         placeLogIfFreeWithOffset(p_230382_1_, p_230382_2_, blockpos$mutable, p_230382_5_, p_230382_6_, p_230382_7_, p_230382_4_, 0, i, 0);
         if (i < p_230382_3_ - 1) {
            placeLogIfFreeWithOffset(p_230382_1_, p_230382_2_, blockpos$mutable, p_230382_5_, p_230382_6_, p_230382_7_, p_230382_4_, 1, i, 0);
            placeLogIfFreeWithOffset(p_230382_1_, p_230382_2_, blockpos$mutable, p_230382_5_, p_230382_6_, p_230382_7_, p_230382_4_, 1, i, 1);
            placeLogIfFreeWithOffset(p_230382_1_, p_230382_2_, blockpos$mutable, p_230382_5_, p_230382_6_, p_230382_7_, p_230382_4_, 0, i, 1);
         }
      }

      return ImmutableList.of(new FoliagePlacer.Foliage(p_230382_4_.above(p_230382_3_), 0, true));//使用主干顶部坐标，实例化一个叶子节点，并返回一个存放着这个叶子节点的list
   }
```

不要看大型丛林树结构很复杂，其实它没有多少东西，对吧？

实际生成大型丛林树树干时，先执行GiantTrunkPlacer的placeTrunk，再执行自己的placeTrunk。我画两个流程图讲解一下：

![image-20220514113438278](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514113438278.png)

![image-20220514113840402](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514113840402.png)

现在再回头看看我们再熟悉不过的丛林树，应该可以很清晰地分析出它的树干结构了。

![image-20220514113741419](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514113741419.png)

#### 补：侧枝生成算法

<img src="https://tse1-mm.cn.bing.net/th/id/R-C.330cd5022df2adc958d10258b3b6ac36?rik=h3rQvdQsDTM3wg&riu=http%3a%2f%2fwww.robot-testing.com%2fPublic%2fUploads%2fimage%2f20180809%2f5b6ba545710c2.jpg&ehk=YZ81RL4heFsp7qE7Rf4hoqJXaNft%2fue3ZV3KZkMcb%2fA%3d&risl=&pid=ImgRaw&r=0&sres=1&sresct=1" alt="查看源图像" style="zoom:50%;" />

```java
float f = p_230382_2_.nextFloat() * ((float)Math.PI * 2F);//确定球坐标偏角Ф
int j = 0;
int k = 0;

//生成侧枝         
for(int l = 0; l < 5; ++l) {
            j = (int)(1.5F + MathHelper.cos(f) * (float)l);
            k = (int)(1.5F + MathHelper.sin(f) * (float)l);
            BlockPos blockpos = p_230382_4_.offset(j, i - 3 + l / 2, k);
            placeLog(p_230382_1_, p_230382_2_, blockpos, p_230382_5_, p_230382_6_, p_230382_7_);
         }
```

我第一次看到这段代码感到一头雾水，但想到球坐标，这些东西就好懂了。

假设有一棵运气很好的树，它的起始生成位置的xz正好是（0，0）。还记得起始生成位置是什么吗？【泥土上的靠西北角的第一块原木，即x、z都最小】

再假设现在这棵树的2*2主干已经生成完毕。

图中的偏转角Ф就是f，容易看出f∈[0,2π）。

这个侧枝一共要生成5个原木，可以近似地认为r=5。我们先来看一个俯视图。

##### f为π/2整数倍

假设f正好是π/2。

![image-20220514120241722](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514120241722.png)

```java
//已知条件：侧枝基部坐标=(0,0)，f=π/2，以下均为伪代码

//第一次循环：l=0
j=(int)(1.5F)=1;
k=1;
blockpos=(1,1);

//第二次循环：l=1
j=(int)(1.5F+cos(π/2)*1)=(int)(1.5F)=1;
k=(int)(1.5F+sin(π/2)*1)=(int)(2.5F)=2;
blockpos=(1,2);

//第三次循环：l=2
j=(int)(1.5F+cos(π/2)*2)=(int)(1.5F)=1;
k=(int)(1.5F+sin(π/2)*2)=(int)(3.5F)=3;
blockpos=(1,3);

//第四次循环：l=3
j=1;
k=4;
blockpos=(1,4);

//第五次循环：l=4
j=1;
k=5;
blockpos=(1,5);
```

![image-20220514122533930](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514122533930.png)

对于其他的f也能画出类似的图。我画了0，π/2，π，3π/2的示意图。

![image-20220514123138718](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514123138718.png)

容易看出，所有的侧枝都会从（1，1）开始生成。有兴趣的玩家可以注意一下你碰到的丛林树，它的东南方向的侧枝都会较长一些。~~这反映了植物的向光性~~

##### f为任意角度

以f=3/π为例。

```java
//已知条件：侧枝基部坐标=(0,0)，f=π/3，以下均为伪代码

//第一次循环：l=0
blockpos=(1,1);

//第二次循环：l=1
j=(int)(1.5F+cos(π/3)*1)=2;
k=(int)(1.5F+sin(π/3)*1)=2;
blockpos=(2,2);

//第三次循环：l=2
j=(int)(1.5F+cos(π/3)*2)=(int)(1.5F)=2;
k=(int)(1.5F+sin(π/3)*2)=(int)(3.5F)=3;
blockpos=(2,3);

//第四次循环：l=3
j=(int)(1.5F+cos(π/3)*3)=(int)(1.5F)=3;
k=(int)(1.5F+sin(π/3)*3)=(int)(3.5F)=4;
blockpos=(3,4);

//第五次循环：l=4
j=(int)(1.5F+cos(π/3)*4)=(int)(1.5F)=3;
k=(int)(1.5F+sin(π/3)*4)=(int)(3.5F)=4;
blockpos=(3,4);
```

![image-20220514125544341](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514125544341.png)

一个更抽象的描述：

![image-20220514125553037](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514125553037.png)

##### 侧枝的抬升

我们平时看到的丛林树侧枝都是这样的：

![image-20220514130019383](C:\Users\HP\Desktop\OcteneXin.github.io\_posts\2022-05-14-从根出发：树干生成器TrunkPlacer详解.assets\image-20220514130019383.png)

它在y轴上有一个向上抬升的角度。

<img src="https://tse1-mm.cn.bing.net/th/id/R-C.330cd5022df2adc958d10258b3b6ac36?rik=h3rQvdQsDTM3wg&riu=http%3a%2f%2fwww.robot-testing.com%2fPublic%2fUploads%2fimage%2f20180809%2f5b6ba545710c2.jpg&ehk=YZ81RL4heFsp7qE7Rf4hoqJXaNft%2fue3ZV3KZkMcb%2fA%3d&risl=&pid=ImgRaw&r=0&sres=1&sresct=1" alt="查看源图像" style="zoom:50%;" />

我再次把这张图拿过来，大家应该能够看出，”侧枝向上抬升的角度“就是图中θ的**余角**。

代码中只反映在这一行：

```java
BlockPos blockpos = p_230382_4_.offset(j, i - 3 + l / 2, k);
```

我们只看offset()的第二个参数，l/2充分说明了每两块原木上升一格，抬升的角度就是30度。以此类推，我们也可以写出抬升45度、60度以及绝对值<45度的侧枝，只要我们知道它的sin值。

```java
//抬升45度的侧枝
BlockPos blockpos = p_230382_4_.offset(j, i - 3 + l, k);
placeLog(p_230382_1_, p_230382_2_, blockpos, p_230382_5_, p_230382_6_, p_230382_7_);
```

```java
//抬升60度的侧枝
placeLog(p_230382_1_, p_230382_2_, p_230382_4_.offset(j, i - 3 + l, k), p_230382_5_, p_230382_6_, p_230382_7_);

BlockPos blockpos = p_230382_4_.offset(j, i - 3 + l + 1, k);
placeLog(p_230382_1_, p_230382_2_, blockpos, p_230382_5_, p_230382_6_, p_230382_7_);
++l;//因为这次循环额外消耗了一块原木
```



#### 小结

mc的树和数据结构中的树有这些相似之处。这或许对代码没有帮助，但是，你会对mc的树结构感到更亲切。

| mc的树             | 数据结构的树 |
| ------------------ | ------------ |
| 起始生成位置       | 根节点       |
| 侧枝基部坐标       | 非叶子节点   |
| 将要生成叶子的坐标 | 叶子节点     |
