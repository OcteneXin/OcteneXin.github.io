---
layout: post
---

------

<img src="https://www.mcmod.cn/pages/center/208/album/20150313/14262320907576.png" alt="img" style="zoom:150%;" />

> 暮色森林模组的生物群系之一：萤火虫森林。冒险者常常在这里看到镶嵌萤火虫罐子的“路灯”，这些遗迹仿佛预示着这里曾经存在文明。



我们在现实生活中见到的树木通常不仅仅只是树干和树叶。有些古树上垂挂的藤蔓在风中不时摇动；森林中树木的的树干往往遍布地衣和苔藓，幸运的你甚至会发现树枝上的一两个木耳；被人们视为神灵的巨大的樱花树，枝条上挂着祈福的红丝带和叮叮作响的风铃；城市街头巷尾的行道树往往被灯带所点缀，在夜间流光溢彩。

是的，一棵树的使命不只是枝繁叶茂，还有很多。

一个盛放着两枚带有褐色斑点的鸟蛋的，枯枝编成的鸟巢；几朵在树皮缝隙中生长的蘑菇；鸟儿衔来的偶然落在枝杈中的兰花种子……

或许树并不知道这些事物将依靠它而存在，它只是承载着这些罢了。正如承载着一切的大地这样。



### TreeDecorator的继承树

大家还记得上一篇文章的末尾处吗？我们按两次shift搜索类名（记得勾选右上角的Include non-project items），选择类名后按ctrl+H在右侧显示继承树。

```
TreeDecorator
	|__LeaveVineTreeDecorator	//悬挂在树叶侧面的藤蔓，应用在丛林树和沼泽中的橡树
	|__AlterGroundTreeDecorator	//改变地面方块，用于大云杉和大松树（知道为什么Java版放四个云杉树苗长出的树会使周围的草方块变成灰化土了吧）
	|__BeehiveTreeDecorator		//用于生成悬挂的蜂巢
	|__TrunkVineTreeDecorator	//用于生成树干上的藤蔓
	|__CocoaTreeDecorator		//用于生成丛林树树干上的可可豆
```



### 抽象类TreeDecorator

```java
public abstract class TreeDecorator {
   public static final Codec<TreeDecorator> CODEC = Registry.TREE_DECORATOR_TYPES.dispatch(TreeDecorator::type, TreeDecoratorType::codec);
    //不明

   protected abstract TreeDecoratorType<?> type();
    //与注册有关

   public abstract void place(ISeedReader p_225576_1_, Random p_225576_2_, List<BlockPos> p_225576_3_, List<BlockPos> p_225576_4_, Set<BlockPos> p_225576_5_, MutableBoundingBox p_225576_6_);
    //主要抽象方法

   protected void placeVine(IWorldWriter p_227424_1_, BlockPos p_227424_2_, BooleanProperty p_227424_3_, Set<BlockPos> p_227424_4_, MutableBoundingBox p_227424_5_) {
      this.setBlock(p_227424_1_, p_227424_2_, Blocks.VINE.defaultBlockState().setValue(p_227424_3_, Boolean.valueOf(true)), p_227424_4_, p_227424_5_);
   }
    //一个方便地放置藤蔓的方法

   protected void setBlock(IWorldWriter p_227423_1_, BlockPos p_227423_2_, BlockState p_227423_3_, Set<BlockPos> p_227423_4_, MutableBoundingBox p_227423_5_) {
      p_227423_1_.setBlock(p_227423_2_, p_227423_3_, 19);
      p_227423_4_.add(p_227423_2_);
      p_227423_5_.expand(new MutableBoundingBox(p_227423_2_, p_227423_2_));
   }
    //通用的放置方块的方法
}
```

有了以前的经验，我们自然会注意到place()这个抽象方法。事实上，TreeDecorator本身和它的实现类都非常简单，与之前的TrunkPlacer和FoliagePlacer都好理解许多。（两次汇总的Placement类不在我们的考虑范围内，这东西更难懂……）

place参数解释：

| 参数                 | 解释             |
| -------------------- | ---------------- |
| ISeedReader          | 相当于World      |
| Random               | 随机数           |
| List&lt;BlockPos&gt; | 所有原木坐标     |
| List&lt;BlockPos&gt; | 所有树叶坐标     |
| Set&lt;BlockPos&gt;  | 所有装饰方块坐标 |
| MutableBoundingBox   | 边界箱           |



###实例：LeaveVineTreeDecorator

```java
public class LeaveVineTreeDecorator extends TreeDecorator {
   public static final Codec<LeaveVineTreeDecorator> CODEC;
   public static final LeaveVineTreeDecorator INSTANCE = new LeaveVineTreeDecorator();

   protected TreeDecoratorType<?> type() {
      return TreeDecoratorType.LEAVE_VINE;
   }

   public void place(ISeedReader p_225576_1_, Random p_225576_2_, List<BlockPos> p_225576_3_, List<BlockPos> p_225576_4_, Set<BlockPos> p_225576_5_, MutableBoundingBox p_225576_6_) {
      p_225576_4_.forEach((p_242866_5_) -> {
         if (p_225576_2_.nextInt(4) == 0) {
            BlockPos blockpos = p_242866_5_.west();
            if (Feature.isAir(p_225576_1_, blockpos)) {
               this.addHangingVine(p_225576_1_, blockpos, VineBlock.EAST, p_225576_5_, p_225576_6_);
            }
         }

         if (p_225576_2_.nextInt(4) == 0) {
            BlockPos blockpos1 = p_242866_5_.east();
            if (Feature.isAir(p_225576_1_, blockpos1)) {
               this.addHangingVine(p_225576_1_, blockpos1, VineBlock.WEST, p_225576_5_, p_225576_6_);
            }
         }

         if (p_225576_2_.nextInt(4) == 0) {
            BlockPos blockpos2 = p_242866_5_.north();
            if (Feature.isAir(p_225576_1_, blockpos2)) {
               this.addHangingVine(p_225576_1_, blockpos2, VineBlock.SOUTH, p_225576_5_, p_225576_6_);
            }
         }

         if (p_225576_2_.nextInt(4) == 0) {
            BlockPos blockpos3 = p_242866_5_.south();
            if (Feature.isAir(p_225576_1_, blockpos3)) {
               this.addHangingVine(p_225576_1_, blockpos3, VineBlock.NORTH, p_225576_5_, p_225576_6_);
            }
         }

      });
   }

   private void addHangingVine(IWorldGenerationReader p_227420_1_, BlockPos p_227420_2_, BooleanProperty p_227420_3_, Set<BlockPos> p_227420_4_, MutableBoundingBox p_227420_5_) {
      this.placeVine(p_227420_1_, p_227420_2_, p_227420_3_, p_227420_4_, p_227420_5_);
      int i = 4;

      for(BlockPos blockpos = p_227420_2_.below(); Feature.isAir(p_227420_1_, blockpos) && i > 0; --i) {
         this.placeVine(p_227420_1_, blockpos, p_227420_3_, p_227420_4_, p_227420_5_);
         blockpos = blockpos.below();
      }

   }

   static {
      CODEC = Codec.unit(() -> {
         return INSTANCE;
      });
   }
}
```

place()方法在干什么？我用自然语言表述一下：

遍历每个树叶坐标。对于一个树叶方块，每个侧面有25%的概率会进行检查，如果这个侧面有空间，就调用addHangingVine()方法生成一列长长的藤蔓。

addHangingVine()方法就是在所选树叶侧面生成一个藤蔓方块，并最多向下延伸四格，如果提前碰到非空气方块则停止。

LeaveVineTreeDecorator这个类只实现了这些事情，我们甚至可以自己模仿着建一个类来实现我们的功能。



### 实战：藤蔓悬挂的荧石

> 需求：首先保证即将生成藤蔓的位置以下10格均为空气。在有空间的树叶侧面生成藤蔓，每侧面概率1/16 ，随机向下延伸4~10格，藤蔓末尾有1/3的概率生成一个萤石。

分析：

- 首先保证即将生成藤蔓的位置以下10格均为空气。用一个for循环检查即可。
  - 以下n格：blockpos.below(n)
  - 为空气：Feature.isAir()
- 在有空间的树叶侧面生成藤蔓，完全可以照抄上方的两个方法（原版实现了寻找有空间的侧面和确定藤蔓的方块状态）
- 每侧面概率1/16。修改place的4个if即可。
- 随机向下延伸4~10格。修改addHangingVine的i即可，将它改成一个随机数。
- 提前停止。控制for循环条件即可。
- 末尾有1/3的概率生成一个萤石。在addHangingVine方法末尾随机数控制+自带的setBlock方法即可。

以下是我的实现：

```java
	public void place(ISeedReader p_225576_1_, Random p_225576_2_, List<BlockPos> p_225576_3_, List<BlockPos> p_225576_4_, Set<BlockPos> p_225576_5_, MutableBoundingBox p_225576_6_) {
        p_225576_4_.forEach((p_242866_5_) -> {
            if (p_225576_2_.nextInt(16) == 0) {
                BlockPos blockpos = p_242866_5_.west();
                if (checkBelow(p_225576_1_,blockpos)) {
                    this.addHangingVine(p_225576_1_, p_225576_2_,blockpos, VineBlock.EAST, p_225576_5_, p_225576_6_);
                }
            }

            if (p_225576_2_.nextInt(16) == 0) {
                BlockPos blockpos1 = p_242866_5_.east();
                if (checkBelow(p_225576_1_,blockpos1)) {
                    this.addHangingVine(p_225576_1_,  p_225576_2_,blockpos1, VineBlock.WEST, p_225576_5_, p_225576_6_);
                }
            }

            if (p_225576_2_.nextInt(16) == 0) {
                BlockPos blockpos2 = p_242866_5_.north();
                if (checkBelow(p_225576_1_,blockpos2)) {
                    this.addHangingVine(p_225576_1_, p_225576_2_, blockpos2, VineBlock.SOUTH, p_225576_5_, p_225576_6_);
                }
            }

            if (p_225576_2_.nextInt(16) == 0) {
                BlockPos blockpos3 = p_242866_5_.south();
                if (checkBelow(p_225576_1_,blockpos3)) {
                    this.addHangingVine(p_225576_1_, p_225576_2_, blockpos3, VineBlock.NORTH, p_225576_5_, p_225576_6_);
                }
            }

        });
    }

    private boolean checkBelow(ISeedReader p_225576_1_,BlockPos pos){
        for (int i=0;i<10;i++){
            if(!Feature.isAir(p_225576_1_,pos.below(i)))return false;
        }
        return true;
    }

    private void addHangingVine(IWorldGenerationReader p_227420_1_, Random rand, BlockPos p_227420_2_, BooleanProperty p_227420_3_, Set<BlockPos> p_227420_4_, MutableBoundingBox p_227420_5_) {
        this.placeVine(p_227420_1_, p_227420_2_, p_227420_3_, p_227420_4_, p_227420_5_);
        int i = 4+rand.nextInt(6);

        BlockPos blockpos = p_227420_2_.below();
        for(; Feature.isAir(p_227420_1_, blockpos) && i > 0; --i) {
            this.placeVine(p_227420_1_, blockpos, p_227420_3_, p_227420_4_, p_227420_5_);
            blockpos = blockpos.below();
        }

        if(rand.nextInt(3)==0){
            this.setBlock(p_227420_1_,blockpos, Blocks.GLOWSTONE.defaultBlockState(),p_227420_4_,p_227420_5_);
        }

    }
```



### 实例：CocoaTreeDecorator

```java
   public void place(ISeedReader p_225576_1_, Random p_225576_2_, List<BlockPos> p_225576_3_, List<BlockPos> p_225576_4_, Set<BlockPos> p_225576_5_, MutableBoundingBox p_225576_6_) {
      if (!(p_225576_2_.nextFloat() >= this.probability)) {
         int i = p_225576_3_.get(0).getY();
         p_225576_3_.stream().filter((p_236867_1_) -> {
            return p_236867_1_.getY() - i <= 2;
         }).forEach((p_242865_5_) -> {
            for(Direction direction : Direction.Plane.HORIZONTAL) {
               if (p_225576_2_.nextFloat() <= 0.25F) {
                  Direction direction1 = direction.getOpposite();
                  BlockPos blockpos = p_242865_5_.offset(direction1.getStepX(), 0, direction1.getStepZ());
                  if (Feature.isAir(p_225576_1_, blockpos)) {
                     BlockState blockstate = Blocks.COCOA.defaultBlockState().setValue(CocoaBlock.AGE, Integer.valueOf(p_225576_2_.nextInt(3))).setValue(CocoaBlock.FACING, direction);
                     this.setBlock(p_225576_1_, blockpos, blockstate, p_225576_5_, p_225576_6_);
                  }
               }
            }

         });
      }
   }
```

这个实现类只有一个place()方法。

最外面使用随机数控制。由于树木的主干（不管是1格的还是4格的）都是从树根开始生成，边生成边加到“原木坐标列表”里，所以列表的第0个元素一定是泥土上方那块原木的坐标。i就是泥土上那块原木的y坐标。

下一步，过滤走所有离地面高度超过3的原木，只留下较矮的几块原木（在实践中往往只有3~12块原木会被留下，可能Mojang认为离地面较矮的可可豆更便于采集）。

遍历被留下的原木，每块原木遍历每个侧面，每个侧面有25%的概率会进行空间检查，发现这个侧面毗邻空气则会生成一个可可豆。这就和上面的LeaveVineTreeDecorator的藤蔓检查方式差不多。



### 实战：树枝上的荧石

> 需求：将树的原木与空气接触的**上表面**生成萤石，概率为1/12。

分析：

对于一般的树木，所有原木的上表面都不会暴露出来，不是被其他原木覆盖就是被树叶覆盖。我们暂且不考虑这个问题，只是关注于代码本身。

- 概率为1/12：随机数控制
- 原木与空气接触的上表面：
  - 遍历原木坐标列表
  - 上表面：blockpos.above()
  - 与空气接触：world.isEmptyBlock()或Feature.isAir()

我的实现如下：

```java
    public void place(ISeedReader p_225576_1_, Random p_225576_2_, List<BlockPos> p_225576_3_, List<BlockPos> p_225576_4_, Set<BlockPos> p_225576_5_, MutableBoundingBox p_225576_6_) {
        p_225576_3_.forEach((pos) -> {
            if (p_225576_2_.nextInt(12) == 0) {
                BlockPos blockpos = pos.above();
                if (p_225576_1_.isEmptyBlock(blockpos)) {
                    this.setBlock(p_225576_1_,blockpos,Blocks.GLOWSTONE.defaultBlockState(),p_225576_5_,p_225576_6_);
                }
            }

        });
    }
```



### 思考题

大家可以自己想一下标题图中“悬挂的萤火虫罐子”的实现方式。它不是利用藤蔓悬挂，而是橡木栅栏，因此方块状态的事情不需考虑了。树叶检查的也不是侧面/上面是否为空气，而是下表面。

地上放置的带立柱的萤火虫罐子也是通过这种方式实现的。提示：就是先放置萤火虫罐子再向下放置橡木栅栏。