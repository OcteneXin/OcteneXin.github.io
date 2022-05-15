---
layout: post
---

------

![](https://img-blog.csdnimg.cn/img_convert/eb556cddc5be22771ac6d029236dad98.png)

> Java1.18光影包Complementary Shaders游戏内截图。原图来自于MCBBS。

<br />

> 无名无款，只此一卷；青绿千载，山河无垠。
>
> ——《只此青绿》舞蹈诗剧

<br />

人们自古以来就崇拜树，而树冠是树的灵魂。从庄子笔下“以八千岁为春，八千岁为秋”的椿，到北欧神话承载九大王国的世界树，这些枝繁叶茂的巨树寄托着人们对生命和自然的敬意。在巨大的伞盖一样的树冠之下，无数树叶在风中沙沙作响，仿佛传递着神明的某种指示。

主世界分布最广、种类最繁的生物群系就是森林，树冠笼罩着世界的大部分土地。树叶的绿，不管是冷冽的、泛黄的、明丽的、黯淡的，都是"Foliage"，荫蔽着无数生灵的"Foliage"。

万里江山，只此青绿。



### FoliagePlacer的继承树

> 单词小课堂：
>
> ​	Foliage n.植物的叶；枝叶

```
|__FoliagePlacer
	|__PineFoliagePlacer	//小松树，特征：树叶聚集在树干顶端
	|__AcaciaFoliagePlacer	//金合欢树
	|__JungleFoliagePlacer	//大型丛林树
	|__DarkOakFoliagePlacer	//深色橡树
	|__SpruceFoliagePlacer	//小云杉，特点：树叶单层独立生长
	|__MegaPineFoliagePlacer	//大松树和大云杉
	|__BlobFoliagePlacer		//普通橡树，白桦树，小型丛林树，表现为树冠下部完全平整
		|__FancyFoliagePlacer	//大型橡树和小型气球橡树，表现为完全球形的树叶
		|__BushFoliagePlacer	//丛林地上的橡树灌木
```

在此区分一下云杉（Spruce）的两个种类：松树（Pine）和云杉（Spruce）。它们又有普通型和大型之分，最后一共分成四个种类。

![image-20220514173137708](https://img-blog.csdnimg.cn/d9fbfe1a7b4b47f58c12f6e8d2003e2a.png)

> 上图来源：Minecraft wiki，“树木”条目



### 从抽象到具体——FoliagePlacer

FoliagerPlacer是个抽象类，仍然只贴出构造器、成员方法的原型和成员变量。

```java
public abstract class FoliagePlacer {
   public static final Codec<FoliagePlacer> CODEC = Registry.FOLIAGE_PLACER_TYPES.dispatch(FoliagePlacer::type, FoliagePlacerType::codec);//不明，与注册有关
   protected final FeatureSpread radius;//树冠半径
   protected final FeatureSpread offset;//不明，与垂直偏移有关，只用在云杉和大型橡树中，一般是0

   protected static <P extends FoliagePlacer> P2<Mu<P>, FeatureSpread, FeatureSpread> foliagePlacerParts(Instance<P> p_242830_0_) {
      return p_242830_0_.group(FeatureSpread.codec(0, 8, 8).fieldOf("radius").forGetter((p_242829_0_) -> {
         return p_242829_0_.radius;
      }), FeatureSpread.codec(0, 8, 8).fieldOf("offset").forGetter((p_242828_0_) -> {
         return p_242828_0_.offset;
      }));
   }//不明

   public FoliagePlacer(FeatureSpread p_i241999_1_, FeatureSpread p_i241999_2_) {
      this.radius = p_i241999_1_;
      this.offset = p_i241999_2_;
   }//构造器

   protected abstract FoliagePlacerType<?> type();
    //获取树叶生成器的类型，注册会讲

   public void createFoliage(IWorldGenerationReader p_236752_1_, Random p_236752_2_, BaseTreeFeatureConfig p_236752_3_, int p_236752_4_, FoliagePlacer.Foliage p_236752_5_, int p_236752_6_, int p_236752_7_, Set<BlockPos> p_236752_8_, MutableBoundingBox p_236752_9_) {
      this.createFoliage(p_236752_1_, p_236752_2_, p_236752_3_, p_236752_4_, p_236752_5_, p_236752_6_, p_236752_7_, p_236752_8_, this.offset(p_236752_2_), p_236752_9_);
   }//加偏移量后，调用抽象方法createFoliage

   protected abstract void createFoliage(IWorldGenerationReader p_230372_1_, Random p_230372_2_, BaseTreeFeatureConfig p_230372_3_, int p_230372_4_, FoliagePlacer.Foliage p_230372_5_, int p_230372_6_, int p_230372_7_, Set<BlockPos> p_230372_8_, int p_230372_9_, MutableBoundingBox p_230372_10_);
    //最重要的抽象方法!负责放置叶子

   public abstract int foliageHeight(Random p_230374_1_, int p_230374_2_, BaseTreeFeatureConfig p_230374_3_);
    //抽象方法，获取树冠高度

   public int foliageRadius(Random p_230376_1_, int p_230376_2_) {
      return this.radius.sample(p_230376_1_);
   }
    //获取树冠半径

   private int offset(Random p_236755_1_) {
      return this.offset.sample(p_236755_1_);
   }
    //获取树冠垂直偏移量，一般是0

   protected abstract boolean shouldSkipLocation(Random p_230373_1_, int p_230373_2_, int p_230373_3_, int p_230373_4_, int p_230373_5_, boolean p_230373_6_);
    //抽象方法，指示哪些地方需要被跳过

   protected boolean shouldSkipLocationSigned(Random p_230375_1_, int p_230375_2_, int p_230375_3_, int p_230375_4_, int p_230375_5_, boolean p_230375_6_);
    //另一种方法，指示那些地方需要被跳过

   protected void placeLeavesRow(IWorldGenerationReader p_236753_1_, Random p_236753_2_, BaseTreeFeatureConfig p_236753_3_, BlockPos p_236753_4_, int p_236753_5_, Set<BlockPos> p_236753_6_, int p_236753_7_, boolean p_236753_8_, MutableBoundingBox p_236753_9_);
    //放一层正方形的树叶，跳过需要跳过的部分

    
    //内部类Foilage
   public static final class Foliage {
      private final BlockPos foliagePos;//叶子节点坐标
      private final int radiusOffset;//半径偏移量
      private final boolean doubleTrunk;//是否双层原木（只在深色橡木的FoliagePlacer那里出现过）

      public Foliage(BlockPos p_i232035_1_, int p_i232035_2_, boolean p_i232035_3_) {
         this.foliagePos = p_i232035_1_;
         this.radiusOffset = p_i232035_2_;
         this.doubleTrunk = p_i232035_3_;
      }

      public BlockPos foliagePos() {
         return this.foliagePos;
      }

      public int radiusOffset() {
         return this.radiusOffset;
      }

      public boolean doubleTrunk() {
         return this.doubleTrunk;
      }
   }
}
```

注意，FoliagePlacer的radius和Foliage的radiusOffset不是一种东西，文末会说明。

在此之前有必要说明一下抽象方法createFoliage的方法声明，因为都是srg名，要一下理清楚是不太容易的。注意，就在它旁边还有一个非抽象的方法也叫createFoliage，但我们讨论的是带abstract的抽象方法。

```java
protected abstract void createFoliage(IWorldGenerationReader p_230372_1_, Random p_230372_2_, BaseTreeFeatureConfig p_230372_3_, int p_230372_4_, FoliagePlacer.Foliage p_230372_5_, int p_230372_6_, int p_230372_7_, Set<BlockPos> p_230372_8_, int p_230372_9_, MutableBoundingBox p_230372_10_);
```

- 返回值：void
- 形参：
  - IWorldGenerationReader：上一章说过，和World差不多
  - Random：一个随机数
  - BaseTreeFeatureConfig：配置【未配置的特性TREE】的config
  - int：一个叶子节点坐标，具体来说应该叫**叶子生成基准点**
  - Foliage：本次生成的叶子节点对象
  - int：树冠高度
  - int：FoliagePlacer的树冠半径
  - Set<BlockPos>：每一个树叶方块的坐标组成的集合（记得placeTrunk方法也有这样一个参数吗？它记录了所有原木方块的坐标）
  - int：通过某种偏移量得到的一个数，暂不知用途，一般是0
  - MutableBoundingBox：整个结构的边界箱

### 从具体到抽象——JungleFoliagePlacer

我仍然从大型丛林树的树叶开始分析。只关心抽象方法。

上文的抽象方法共三个，分别是createFoliage，foliageHeight和shouldSkipLocation

```java
protected void createFoliage(IWorldGenerationReader p_230372_1_, Random p_230372_2_, BaseTreeFeatureConfig p_230372_3_, int p_230372_4_, FoliagePlacer.Foliage p_230372_5_, int p_230372_6_, int p_230372_7_, Set<BlockPos> p_230372_8_, int p_230372_9_, MutableBoundingBox p_230372_10_) {
      int i = p_230372_5_.doubleTrunk() ? p_230372_6_ : 1 + p_230372_2_.nextInt(2);

      for(int j = p_230372_9_; j >= p_230372_9_ - i; --j) {
         int k = p_230372_7_ + p_230372_5_.radiusOffset() + 1 - j;
         this.placeLeavesRow(p_230372_1_, p_230372_2_, p_230372_3_, p_230372_5_.foliagePos(), k, p_230372_8_, j, p_230372_5_.doubleTrunk(), p_230372_10_);
      }

   }

   public int foliageHeight(Random p_230374_1_, int p_230374_2_, BaseTreeFeatureConfig p_230374_3_) {
      return this.height;
   }

   protected boolean shouldSkipLocation(Random p_230373_1_, int p_230373_2_, int p_230373_3_, int p_230373_4_, int p_230373_5_, boolean p_230373_6_) {
      if (p_230373_2_ + p_230373_4_ >= 7) {
         return true;
      } else {
         return p_230373_2_ * p_230373_2_ + p_230373_4_ * p_230373_4_ > p_230373_5_ * p_230373_5_;
      }
   }
```

先看createFoliage。

```java
protected void createFoliage(IWorldGenerationReader p_230372_1_, Random p_230372_2_, BaseTreeFeatureConfig p_230372_3_, int p_230372_4_, FoliagePlacer.Foliage p_230372_5_, int p_230372_6_, int p_230372_7_, Set<BlockPos> p_230372_8_, int p_230372_9_, MutableBoundingBox p_230372_10_) {
    
    
      int i = p_230372_5_.doubleTrunk() ? p_230372_6_ : 1 + p_230372_2_.nextInt(2);//我们知道doubleTrunk只在深色橡树出现过，所以i采用三元运算符的后一种算法，即1~3之间

      for(int j = p_230372_9_; j >= p_230372_9_ - i; --j) {//由于p_230372_9_=0，所以j从0->-i变化，j是y的相对坐标，会越来越小，表现为树叶自顶向下按层生成
         int k = p_230372_7_ + p_230372_5_.radiusOffset() + 1 - j;//单层叶子实际半径，等于FoliagePlacer的半径+Foliage的半径偏移量+1-j；从循环可以看出，k会越来越大
         this.placeLeavesRow(p_230372_1_, p_230372_2_, p_230372_3_, p_230372_5_.foliagePos(), k, p_230372_8_, j, p_230372_5_.doubleTrunk(), p_230372_10_);
          //这里在调用父类的非抽象方法placeLeavesRow
      }

   }
```

#### 半径基准量和偏移量

![image-20220515135043776](https://img-blog.csdnimg.cn/8c95d780c971459084b883bf34dc88b3.png)

注意这一行：

`int k = p_230372_7_ + p_230372_5_.radiusOffset() + 1 - j;`

p_230372_7_ 是FoliagePlacer半径，我称为**半径基准量**； p_230372_5_.radiusOffset()是Foliage的**半径偏移量**，就是在半径基准量上的差值。

我假设i=1，故循环两次：

```java
//第一次循环
k=4-2+1-0=3;

//第二次循环
k=4-2+1-(-1)=4;
```

所以，对于大型丛林树树叶而言，单层实际半径由半径基准量、偏移量和当前层数共同决定。

#### placeLeavesRow详解

createFoliage方法里不断地调用这个方法，有必要看一下这个方法的逻辑。

```java
   protected void placeLeavesRow(IWorldGenerationReader p_236753_1_, Random p_236753_2_, BaseTreeFeatureConfig p_236753_3_, BlockPos p_236753_4_, int p_236753_5_, Set<BlockPos> p_236753_6_, int p_236753_7_, boolean p_236753_8_, MutableBoundingBox p_236753_9_) {
      int i = p_236753_8_ ? 1 : 0;
      BlockPos.Mutable blockpos$mutable = new BlockPos.Mutable();

      for(int j = -p_236753_5_; j <= p_236753_5_ + i; ++j) {
         for(int k = -p_236753_5_; k <= p_236753_5_ + i; ++k) {
            if (!this.shouldSkipLocationSigned(p_236753_2_, j, p_236753_7_, k, p_236753_5_, p_236753_8_)) {
               blockpos$mutable.setWithOffset(p_236753_4_, j, p_236753_7_, k);
               if (TreeFeature.validTreePos(p_236753_1_, blockpos$mutable)) {
                  p_236753_1_.setBlock(blockpos$mutable, p_236753_3_.leavesProvider.getState(p_236753_2_, blockpos$mutable), 19);
                  p_236753_9_.expand(new MutableBoundingBox(blockpos$mutable, blockpos$mutable));
                  p_236753_6_.add(blockpos$mutable.immutable());
               }
            }
         }
      }

   }
```

先说方法声明。

- 返回值：void
- 形参：
  - IWorldGenerationReader：略
  - Random：略
  - BaseTreeFeatureConfig：略
  - Blockpos：**叶子生成基准点**
  - int：本层正方形树叶的边长的一半（实际半径）
  - Set<Blockpos>：所有树叶坐标的集合
  - int：本层距离**叶子生成基准点**的高度
  - boolean：是否双层原木（只有深色橡树在用）
  - MutableBoundingBox：略

最重要的就是两个int型的参数。

方法内部的双层for循环扫描2r*2r正方形区域内的每一个坐标，通过shouldSkipLocationSigned剔除无用坐标，在有效的坐标处放置一个树叶。

由于这个方法又调用了另一个重要的方法shouldSkipLocationSigned，我们先去研究那个方法。

##### shouldSkipLocationSigned和shouldSkipLocation

这两个方法见名知义，就是判断哪些地方应该被跳过。树冠俯视着看总是圆圆的，总不能有正方形的树冠吧？

```java
   protected boolean shouldSkipLocationSigned(Random p_230375_1_, int p_230375_2_, int p_230375_3_, int p_230375_4_, int p_230375_5_, boolean p_230375_6_) {
      int i;
      int j;
      if (p_230375_6_) {
         i = Math.min(Math.abs(p_230375_2_), Math.abs(p_230375_2_ - 1));
         j = Math.min(Math.abs(p_230375_4_), Math.abs(p_230375_4_ - 1));
      } else {
         i = Math.abs(p_230375_2_);
         j = Math.abs(p_230375_4_);
      }

      return this.shouldSkipLocation(p_230375_1_, i, p_230375_3_, j, p_230375_5_, p_230375_6_);
   }
```

- 参数
  - Random：略
  - int：与叶子生成基准点的相对x坐标
  - int：与叶子生成基准点的相对y坐标
  - int：与叶子生成基准点的相对z坐标
  - int：实际半径
  - boolean：是否双层原木

由于丛林树树叶肯定不是”双层原木“，所以直接走else路线，把相对x、z坐标取绝对值后传入shouldSkipLocation。



shouldSkipLocation是个抽象方法，所以我们必须看一下JungleFoliagePlacer对它的实现。

```java
   protected boolean shouldSkipLocation(Random p_230373_1_, int p_230373_2_, int p_230373_3_, int p_230373_4_, int p_230373_5_, boolean p_230373_6_) {
      if (p_230373_2_ + p_230373_4_ >= 7) {
         return true;
      } else {
         return p_230373_2_ * p_230373_2_ + p_230373_4_ * p_230373_4_ > p_230373_5_ * p_230373_5_;
      }
   }
```

这里东西不多，就一句话，意义也不言自明，就是控制每层树叶呈一个圆形。之所以加上<7的限制，我认为有可能是避免树叶枯萎造成卡顿。

![image-20220515102119810](https://img-blog.csdnimg.cn/f3a33fae4f9744abb4d6fd17283b9057.png)

这是一棵大型丛林树最大树冠（即2*2主干上的树冠）的俯视图。容易看出一共有三层，越向下半径越大。整体呈一个圆形，半径分别为4，5，6。

从上往下第一层：

![image-20220515121149671](https://img-blog.csdnimg.cn/3eb5f4f390294f7bb8d9c2564349af44.png)

从上往下第二层：

![image-20220515121658047](https://img-blog.csdnimg.cn/277a0c9a1fb843e2b4dc98bcfa68b44f.png)

从上往下第三层：

![image-20220515122121286](https://img-blog.csdnimg.cn/4f2f6a42b787444e9d39801ccdaf41aa.png)

去除了顶上三层树叶，现在的丛林树是这样的：

![image-20220515122351518](https://img-blog.csdnimg.cn/df086fcb4a614081a68cd5704f9b17df.png)

接下来研究一个小树冠。

![image-20220515122850886](https://img-blog.csdnimg.cn/15ea501a7a3448f5978baf77e92a349b.png)

容易看出，这个树冠共两层，半径分别为1和2。

再看一个略大的小树冠。

![image-20220515123339885](https://img-blog.csdnimg.cn/ff4337a2141842b2b3a8f90924675de2.png)

这个树冠共3层，半径分别为1，2，3。

<br />

现在我们回到placeLeavesRow本身，这里的逻辑就相对简单了。

```java
      for(int j = -p_236753_5_; j <= p_236753_5_ + i; ++j) {
         for(int k = -p_236753_5_; k <= p_236753_5_ + i; ++k) {
            if (!this.shouldSkipLocationSigned(p_236753_2_, j, p_236753_7_, k, p_236753_5_, p_236753_8_)) {//该坐标未被跳过
               blockpos$mutable.setWithOffset(p_236753_4_, j, p_236753_7_, k);//从树叶基准点开始，通过相对坐标计算树叶将生成的位置
               if (TreeFeature.validTreePos(p_236753_1_, blockpos$mutable)) {
                  p_236753_1_.setBlock(blockpos$mutable, p_236753_3_.leavesProvider.getState(p_236753_2_, blockpos$mutable), 19);//放置一块树叶
                  p_236753_9_.expand(new MutableBoundingBox(blockpos$mutable, blockpos$mutable));
                  p_236753_6_.add(blockpos$mutable.immutable());//新放置的树叶坐标保存在树叶坐标集合中
               }
            }
         }
      }
```

总结一下，placeLeavesRow的唯一作用就是：放一层特定形状的树叶。

#### 小结

一团丛林树叶的放置有以下几步：

1. 确定树叶基准点坐标：placeTrunk的时候向一个集合内保存了所有叶子节点，遍历这些节点，取出坐标即可；
2. 计算树冠高度：随机数计算得出；
3. 计算当前层树叶实际半径：由FoliagePlacer的radius、Foliage的radiusOffset、当前层数共同决定；
4. 循环调用placeLeavesRow一层层地放树叶。

这对于其他种类树叶的生成并不是百分百适用，但不同的FoliagePlacer实现类基本上都是这个逻辑。

![image-20220515150251920](https://img-blog.csdnimg.cn/918dc5581564401a82f404aeee9afd92.png)
