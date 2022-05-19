---
layout: post
---

------

![image-20220518153135109](https://img-blog.csdnimg.cn/956e05b4b7444449b5e48f8306edbe77.png)

> 一个海拔较低（y平均值72），地势平坦的峭壁生物群系。可以看到云杉树、蒲公英、水湖、熔岩湖、闪长岩、花岗岩的分布。



“分布”在英文中有如下几个常见表述：

| 单词       | 详解                                                  | 例句                                                         | 翻译                                     |
| ---------- | ----------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------- |
| distribute | v.使散开；使分布；分散                                | The map shows the distribution of this species across the world. | 地图上标明了这一物种在全世界的分布情况。 |
| disperse   | v.（使）分散，散开；疏散；驱散                        | They live high in the Andes, in small and dispersed groups.  | 他们零星散居在安第斯山脉的高处。         |
| scatter    | v.散开；四散；使分散；驱散 n.散落；三三两两；零零星星 | Papers were scattered here and there on the floor.           | 地板上到处散落着文件。                   |
| spread     | v.使分散；使分布                                      | Seeds and pollen are spread by the wind.                     | 种子和花粉是随风传播的。                 |
| placement  | n.（对物件的）安置，放置                              | The cuts and placement of the stones are said to be very precise. | 据说，这些石头的放置和切割十分精确。     |
| space      | v.以一定间隔排列                                      | Space the posts about a metre apart.                         | 这些杆子之间要间隔一米左右。             |

> 释义均来自于牛津高阶或柯林斯词典。例句和翻译来自于百度翻译。



作者在此不讨论单词辨析问题，只是整理一下一些近义词而已。（其实作者英语很菜的……）

1.16.5的mcp选择了placement作为位置分布器的名字。（不了解notch名，srg名和mcp名的小伙伴可以看<a href="https://fmltutor.ustc-zzzz.net/%E9%99%84%E5%BD%95B-%E6%B7%B7%E6%B7%86%E4%B8%8E%E5%8F%8D%E5%B0%84.html">fmltutor的文档</a>，他们讲得相当明白）但相对于placement，个人认为scatter这个词来形容特性的位置分布行为更加合适。

特性（和结构）的分布是mc世界极为重要的一件大事。没有人会想看到自然条件下的树整整齐齐地排列在每个区块里，也很少有人会喜欢整整齐齐排列在地下的废弃矿井（一些旧版本存在这样的bug，不过也确实是奇观了）。玩家总是要追求随机性，新的世界，一切都是不确定的，就像你不知道什么时候会遇到一个蘑菇岛一样。~~尽管作者我在生存里一个也没找到过~~

但是，我们的随机不是真正意义上的随机。为什么这么说呢？

学过高中生物必修三《种群与群落》这一章的同学们都知道，有三张图简略介绍了种群的空间分布（因为担心读者不太喜欢原文的第三张图，就只放一张概念图介绍一下）：

![查看源图像](https://img-blog.csdnimg.cn/img_convert/1678cf5464554a8889fc33a10c310653.png)

这三种分布是种群最常见的三种分布型，但与大家的直觉相反的是，随机分布反而是自然界中最少见的分布型。接下来请允许我搬运一点点生物知识过来~~我保证，立刻，马上讲代码！~~

| 分布型   | 特征                                           | 部分成因                                                     | 举例                                               |
| -------- | ---------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| 均匀分布 | 个体之间彼此保持一致的距离                     | 种群内个体的竞争                                             | 森林中的树木竞争树冠空间或根部空间可能导致均匀分布 |
| 随机分布 | 每个个体在种群分布空间内各个位置出现的机会相等 | 资源分配均匀，种群内个体没有吸引或排斥                       | 生活在森林底层的蜘蛛                               |
| 集群分布 | 个体分布不均匀，成群、成块地密集分布           | 环境资源分配不均、植物以母株为中心扩散传播种子、动物的社会行为 | 瓢虫的集群分布                                     |

集群分布是自然界最常见的分布型，而随机分布是最罕见的。

mc在一定程度上体现了这种特征，比如，同种动物倾向于集群生成（想起你经常在平原上看到一群骏马的情景了吗？）；丛林的西瓜、水边的甘蔗、针叶林的甜浆果总是一小片一小片地生成；就连同种怪物也会集群生成（僵尸，每组生成1~4个）；更甚的是，下界的火也会成群生成，或许是因为火就近传播的缘故吧。

Random生成的噪声和电视机出现的雪花点别无二致，它自然不能胜任集群生成的任务，所以Mojang采用了一个折中的办法：先把一小群东西写好，再调用正常的random来放置这些小群。

这里申明一下：采用成群生成策略的Feature大多数与植物有关，我们讨论这些Feature，尤其是与树有关的。其他的Feature有采用单独生成策略的，这不在我们的讨论范围内。



### Placement和Placements

大家如果记忆力好一点的话，应该能想起来Feature/Features，Structure/SturctureFeatures之间的爱恨情仇：前者祖宗类兼汇总类，后者二次汇总类。对于位置分布器，它也是遵循这个原则，只不过Placements变成了一个内部类，待在Features里面，等待着参与到ConfiguredFeature的建设中——没错，一个ConfiguredFeature的构建需要很多很多东西，不管是TrunkPlacer，FoliagePlacer还是今天马上要讨论的ConfiguredPlacement，都只是冰山一角。

“什么？还有ConfiguredPlacement这种东西？”

看来我们很有必要更新一下这张表格了：

![image-20220518170017813](https://img-blog.csdnimg.cn/27f9509587db4f4dbae3a32ae6ed51ec.png)

注意到，Placement的config实现了IPlacementConfig接口，与IFeatureConfig并不一样。

并且，Placement的二次汇总产物也不像前两种那样繁多，以下就是出现在Placements里面全部的产物了：

```java
   public static final class Placements {
      public static final BeehiveTreeDecorator BEEHIVE_0002 = new BeehiveTreeDecorator(0.002F);
      public static final BeehiveTreeDecorator BEEHIVE_002 = new BeehiveTreeDecorator(0.02F);
      public static final BeehiveTreeDecorator BEEHIVE_005 = new BeehiveTreeDecorator(0.05F);
      public static final ConfiguredPlacement<FeatureSpreadConfig> FIRE = Placement.FIRE.configured(new FeatureSpreadConfig(10));
      public static final ConfiguredPlacement<NoPlacementConfig> HEIGHTMAP = Placement.HEIGHTMAP.configured(IPlacementConfig.NONE);
      public static final ConfiguredPlacement<NoPlacementConfig> TOP_SOLID_HEIGHTMAP = Placement.TOP_SOLID_HEIGHTMAP.configured(IPlacementConfig.NONE);
      public static final ConfiguredPlacement<NoPlacementConfig> HEIGHTMAP_WORLD_SURFACE = Placement.HEIGHTMAP_WORLD_SURFACE.configured(IPlacementConfig.NONE);
      public static final ConfiguredPlacement<NoPlacementConfig> HEIGHTMAP_DOUBLE = Placement.HEIGHTMAP_SPREAD_DOUBLE.configured(IPlacementConfig.NONE);
      public static final ConfiguredPlacement<TopSolidRangeConfig> RANGE_10_20_ROOFED = Placement.RANGE.configured(new TopSolidRangeConfig(10, 20, 128));
      public static final ConfiguredPlacement<TopSolidRangeConfig> RANGE_4_8_ROOFED = Placement.RANGE.configured(new TopSolidRangeConfig(4, 8, 128));
      public static final ConfiguredPlacement<?> ADD_32 = Placement.SPREAD_32_ABOVE.configured(NoPlacementConfig.INSTANCE);
      public static final ConfiguredPlacement<?> HEIGHTMAP_SQUARE = HEIGHTMAP.squared();
      public static final ConfiguredPlacement<?> HEIGHTMAP_DOUBLE_SQUARE = HEIGHTMAP_DOUBLE.squared();
      public static final ConfiguredPlacement<?> TOP_SOLID_HEIGHTMAP_SQUARE = TOP_SOLID_HEIGHTMAP.squared();
   }
```

大家不必太关注这些代码里的东西。为什么呢？之所以在这里出现的ConfiguredPlacement这么少，是因为大多数ConfiguredPlacement是匿名对象，隐藏在ConfiguredFeature的构建过程之中。以普通桦树林的生成为例：

```java
public static final ConfiguredFeature<?, ?> TREES_BIRCH
    = register("trees_birch",
               BIRCH_BEES_0002
               	.decorated(Features.Placements.HEIGHTMAP_SQUARE)
               	.decorated(
                   Placement.COUNT_EXTRA
                    .configured(new AtSurfaceWithExtraConfig(10, 0.1F, 1))
               	)
              );
```

大家应该会看出来，第二个decorated括号里就是一个匿名的ConfiguredPlacement，它是通过Placement调用configured方法临时生产出来的。Placements类里是不会出现这些东西的。



### 主要Placement和ConfiguredPlacement

为什么还要介绍初次汇总类Placement呢？因为大多数ConfiguredPlacement都是由Placement的初次汇总产物匿名产生的，也有必要提一些。

以下的常量都是Features里出现频率相当高的。作者认真研究了下表每个Placement和ConfiguredPlacement的算法，对于这些分布给出了一个合理的解释。

![](https://img-blog.csdnimg.cn/fa1551cc69d4423db96efc93f5cd23f3.png)

### Placement的本质实现

我们先去找一个最基本的Placement看看。

```java
//SimplePlacement.java

public abstract class SimplePlacement<DC extends IPlacementConfig> extends Placement<DC> {
   public SimplePlacement(Codec<DC> p_i232095_1_) {
      super(p_i232095_1_);
   }

   public final Stream<BlockPos> getPositions(WorldDecoratingHelper p_241857_1_, Random p_241857_2_, DC p_241857_3_, BlockPos p_241857_4_) {
      return this.place(p_241857_2_, p_241857_3_, p_241857_4_);
   }

   protected abstract Stream<BlockPos> place(Random p_212852_1_, DC p_212852_2_, BlockPos p_212852_3_);
}
```

Placement的其他子类事实上都继承了这个类，这样才拥有了其中的place方法。place方法和getpositions方法的返回值类型是一样的，都是Stream<Blockpos>，这说明它们可能会返回一连串的方块坐标，而不仅仅是一个。

这个final的getPositions方法是从SimplePlacement的父类，Placement里来的。

```java
//Placement.java

public abstract Stream<BlockPos> getPositions(WorldDecoratingHelper p_241857_1_, Random p_241857_2_, DC p_241857_3_, BlockPos p_241857_4_);
```

对着父类的getPositions方法来一下ctrl+alt+shift+F7，全项目全库搜索这个方法的使用。果然被我找到一个：

```java
//ConfiguredPlacement.java

private final Placement<DC> decorator;
//...
public Stream<BlockPos> getPositions(WorldDecoratingHelper p_242876_1_, Random p_242876_2_, BlockPos p_242876_3_) {
   return this.decorator.getPositions(p_242876_1_, p_242876_2_, this.config, p_242876_3_);
}
```

这里的decorator就是指一个Placement类型的对象。

对着这个getPositions来一次搜索，找到一个关键的地方。

```java
//DecoratedFeature.java

public boolean place(ISeedReader p_241855_1_, ChunkGenerator p_241855_2_, Random p_241855_3_, BlockPos p_241855_4_, DecoratedFeatureConfig p_241855_5_) {
      MutableBoolean mutableboolean = new MutableBoolean();
      p_241855_5_.decorator.getPositions(new WorldDecoratingHelper(p_241855_1_, p_241855_2_), p_241855_3_, p_241855_4_).forEach((p_242772_5_) -> {
         if (p_241855_5_.feature.get().place(p_241855_1_, p_241855_2_, p_241855_3_, p_242772_5_)) {
            mutableboolean.setTrue();
         }

      });
      return mutableboolean.isTrue();
}
```

这个方法是它的父类Feature来的。

```java
//Feature.java

public abstract boolean place(ISeedReader p_241855_1_, ChunkGenerator p_241855_2_, Random p_241855_3_, BlockPos p_241855_4_, FC p_241855_5_);
```

对它搜索之，发现ConfiguredFeature（注意，没有s）在调这个place。注意，ConfiguredFeature不是Feature的子类；它们是毫无关系的两个类，ConfguredFeature甚至还更接近于世界生成的部分。

```java
//ConfiguredFeature.java

public boolean place(ISeedReader p_242765_1_, ChunkGenerator p_242765_2_, Random p_242765_3_, BlockPos p_242765_4_) {
   return this.feature.place(p_242765_1_, p_242765_2_, p_242765_3_, p_242765_4_, this.config);
}
```

搜索之，发现这个方法被调用了很多很多次。

![image-20220518213658538](https://img-blog.csdnimg.cn/326814ad36f24b0aa530f1d2abe3ca84.png)

几乎所有的特性都归生物群系管。为什么这么说呢，比如只有巨型针叶林会生成生苔的巨石，只有冰刺平原才会生成冰刺，只有黑森林才会生成深色橡树……所以我们自然而然地去biome这个包下找。

```java
//Biome.java

public void generate(StructureManager p_242427_1_, ChunkGenerator p_242427_2_, WorldGenRegion p_242427_3_, long p_242427_4_, SharedSeedRandom p_242427_6_, BlockPos p_242427_7_) {
				//...
          if (list.size() > j) {
            for(Supplier<ConfiguredFeature<?, ?>> supplier : list.get(j)) {
               ConfiguredFeature<?, ?> configuredfeature = supplier.get();
               p_242427_6_.setFeatureSeed(p_242427_4_, k, j);

               try {
                  configuredfeature.place(p_242427_3_, p_242427_2_, p_242427_6_, p_242427_7_);
               } catch (Exception exception1) {
					//...
               }

               ++k;
            }
         }

   }
```

简单地来说，就是先获取生物群系特性列表，遍历之，对每一个特性调一次place。到此，源码就追到头了，再向上走就要谈生物群系的生成了（这个我确实不会……）

<br />

好了，我画一个流程图说明一下一般特性的放置过程：

用例：普通白桦树林

```java
public static final ConfiguredFeature<?, ?> TREES_BIRCH
    = register("trees_birch", 
               BIRCH_BEES_0002
               	.decorated(Features.Placements.HEIGHTMAP_SQUARE)
               	.decorated(Placement.COUNT_EXTRA
                          .configured(new AtSurfaceWithExtraConfig(10, 0.1F, 1))
                         )
              );
```

![image-20220518222037149](https://img-blog.csdnimg.cn/5c99a9a98dcc405a90a343ccebc3bc49.png)

注：此图并不等同于实际生成的情景，均为作者对源码的个人理解，如有出入请指正。

也许大家会问，为什么ConfiguredFeature第一、二次place的时候找到DecoratedFeature，第三次就找到TreeFeature了呢？事实上，是包装的缘故，和decorated()方法有关，接下来简要介绍一下。



### decorated()方法与DecoratedFeature类

DecoratedFeature继承了Feature，是一个十分重要的类，所有经过decorated()、count()、square()修饰的ConfiguredFeature都和它有关系。

还是以上面的白桦树林举例子。

```java
public static final ConfiguredFeature<?, ?> TREES_BIRCH
    = register("trees_birch", 
               BIRCH_BEES_0002
               	.decorated(Features.Placements.HEIGHTMAP_SQUARE)
               	.decorated(Placement.COUNT_EXTRA
                          .configured(new AtSurfaceWithExtraConfig(10, 0.1F, 1))
                         )
              );
```

里面用了两个decorated()，我们只研究其中的一个就行了，比如说`decorated(Features.Placements.HEIGHTMAP_SQUARE)`。

```java
public static final ConfiguredFeature<?, ?> TREES_BIRCH
    = register("trees_birch", 
               BIRCH_BEES_0002
               	.decorated(Features.Placements.HEIGHTMAP_SQUARE)
              );
//只关注一个decor
```

我们知道BIRCH_BEES_0002是二次汇总产物，类型是ConfiguredFeature。

decorated()方法返回的也是ConfiguredFeature，需要传入一个ConfiguredPlacement类型的参数。这里有个现成的ConfiguredPlacement，就是HEIGHTMAP_SQUARE。

```java
//ConfiguredFeature.java  

public ConfiguredFeature<?, ?> decorated(ConfiguredPlacement<?> p_227228_1_) {
      return Feature.DECORATED.configured(new DecoratedFeatureConfig(
          () -> {
         		return this;
      			}, 
          p_227228_1_
      )
                                         );
   }
```

```java
//DecoratedFeatureConfig.java

public DecoratedFeatureConfig(Supplier<ConfiguredFeature<?, ?>> p_i241984_1_, ConfiguredPlacement<?> p_i241984_2_) {
   this.feature = p_i241984_1_;
   this.decorator = p_i241984_2_;
}
```

在decorated()方法内部，我们手动用一个**函数式接口的原来的ConfiguredFeature**和**刚才传入的ConfiguredPlacement**构建一个新的DecoratedFeatureConfig对象。（我其实不太了解函数式接口，但鉴于Mojang经常用它，还是尽量学习一下）

```java
//Feature.java

public static final Feature<DecoratedFeatureConfig> DECORATED = register("decorated", new DecoratedFeature(DecoratedFeatureConfig.CODEC));
//...

public ConfiguredFeature<FC, ?> configured(FC p_225566_1_) {
   return new ConfiguredFeature<>(this, p_225566_1_);
}
```

将这个对象传入configured()方法，构建并返回一个新的ConfiguredFeature。

这个ConfiguredFeature已经被包装了，它的Feature变成了Feature&lt;DecoratedFeatureConfig&gt;类型的常量**DECORATED**，它的config变成了DecoratedFeatureConfig，原来的ConfiguredFeature就被包在这个config里面。

ConfiguredFeature调place()，实际上在调它内部Feature的place()。

```java
   public boolean place(ISeedReader p_242765_1_, ChunkGenerator p_242765_2_, Random p_242765_3_, BlockPos p_242765_4_) {
      return this.feature.place(p_242765_1_, p_242765_2_, p_242765_3_, p_242765_4_, this.config);
   }
```

新的ConfiguredFeature调place方法的时候，自然会调DecoratedFeature类里面的place()，因为DecoratedFeature属于Feature&lt;DecoratedFeatureConfig&gt;类型。

```java
public class DecoratedFeature extends Feature<DecoratedFeatureConfig>{}
```

这个类的place()和别的类很不一样，它最有贡献的一点就是在一个place()里控制了特性生成的位置，毕竟谁也不想把所有东西都生成在区块的西北角吧？更重要的是，它还有一个能力，就是能利用Stream将同一个特性在周围生成好几次。

```java
   public boolean place(ISeedReader p_241855_1_, ChunkGenerator p_241855_2_, Random p_241855_3_, BlockPos p_241855_4_, DecoratedFeatureConfig p_241855_5_) {
      MutableBoolean mutableboolean = new MutableBoolean();
      p_241855_5_.decorator.getPositions(new WorldDecoratingHelper(p_241855_1_, p_241855_2_), p_241855_3_, p_241855_4_).forEach((p_242772_5_) -> {
         if (p_241855_5_.feature.get().place(p_241855_1_, p_241855_2_, p_241855_3_, p_242772_5_)) {
            mutableboolean.setTrue();
         }

      });
      return mutableboolean.isTrue();
   }
```

注意这一行：`p_241855_5_.feature.get()`

这就相当于拆包了，把原来的ConfiguredFeature从DecoratedFeatureConfig里放出来，调它的place。

原来的ConfiguredFeature会调用原来的Feature的place()。对于我们的例子，原来的Feature是TreeFeature类型的，会调TreeFeature里重写的place()。

**这个place()是我们构建一棵树的最终奥义，但是作者我写不动了……**

<br />

上面这些字我看着都头痛，还是画图来得简单：

![image-20220518233216155](https://img-blog.csdnimg.cn/4dca24d0c564493e99c0fea7d7771dfd.png)

> 使用decorated()构建新的ConfiguredFeature的流程。

![image-20220518233737855](https://img-blog.csdnimg.cn/3e3a91df6de64134852032440d84d9fe.png)

> 调用新的ConfiguredFeature的place()时发生的调用链。



### tips

- 在idea里选中一个方法或变量时，按shift+alt+ctrl+F7，将在全项目全库扫描这个方法或变量的调用。要写mc模组的话，这个办法总是百试百灵的，毕竟不是什么人都能猜出来Mojang会把某函数的调用放在什么奇奇怪怪的类里面。

- 按ctrl+F，在一些五六百行的java文件里搜索你需要的变量会有奇效。

- 摁着ctrl，点击方法名，你会跳转到它的上一层用法。

- 点击抽象方法旁边的绿色小圈，你会看到所有实现类实现的该方法。

- 连按两下shift，输入要查找的类名，就能方便地找到你要的类——前提是你已经知道类名。

- 鼠标选中较高级的父类类名，按下ctrl+H组合键，你会在右侧看到它的继承树。