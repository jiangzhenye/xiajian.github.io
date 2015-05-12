---
layout: post
title: "逻辑的引擎 读书笔记"
description: "计算机革命背后的理论支持，逻辑的引擎，读书笔记"
category: note
---

## 前言

《逻辑的引擎》这本书是在自然科学与数理逻辑分馆中看到的，并立马借了过来。当时的我，热衷于学习和翻看一些计算机数学的书籍，热衷于了解数学大师的事迹，从中汲取力量，
并为自己的以后的成功寻找些蛛丝马迹(纯属自娱自乐，请勿见怪)。

《逻辑的引擎》以人物为主线，讲述了400多年来，在数学上奠定计算机发展的重要理论的诸多人物。从莱布尼兹到歌德尔。自上世纪50年代至今的计算机革命，更多是工程技术上的革命，其使用的理论基础已存在百年之久。

> 备注: 后来发现，其实自己没多少数学天分，只是个智商不高，经常卡壳，多动抽动的普通人而已。我为能清楚的认知自己感到高兴，这样我又领先了XX%。到底是从NS中借的，还是从TP那借的，
> 有些记不清了。

## 莱布尼兹 - Leibnitz

1674年，26岁的Leibnitz遇到了克里斯提安.惠梗思(就是物理上发现光的折射定律的混蛋)，后者交给Leihnitz一份书目，使得Leihnitz很快就了解到了当前数学研究的状态。

17世纪数学研究的迅速发展得益于两项主要的进展：

* 处理代数表达式的技巧已经被系统化，这种强大的技巧一直沿用至今。？？数学技巧是什么啊？？
* 笛卡尔(Descartes) 和 费曼(Fermat)分别论证了，如何通过一组组的数对表示点而把几何还原为代数

> (Pi)/(4)=1-(1)/(3)+1/(5)-1/(7)+1/(9)-1/(11)+...

这是Leibnitz发现的无限级数，将求和级数同面积牵连上关系。1673年，Leibnitz展示了可以进行加减乘除的机器--解代数方程的机器--将推理归结为一种演算，最终制成能够完成这些演算的机器。

1675年，Leibnitz即将结束在巴黎的逗留，在这一年的最后几个月中，他用极限过程实现了概念和计算上的一连串突破，即发明了微积分：

1. 计算面积和变化率的问题从某种意义上说很有代表性，应为许多不同的类型的问题都可以转化为这两类问题中的某一类。

1. 两类问题的求解在数学运算上彼此相反，就如同加法和减法运算一样相反，这些运算被称之为微分(Difference)和积分(Integer)

1. Leibnitz为微积分发展出了一套恰当的符号系统，∫表示积分，d表示微分。

> 一套好的符号系统可以推动技术和科学的发展，正则表达式和BNF都是不错的符号系统。

上述的发现将对极限的应用从只有少数人理解的奇特方法，变成了一种可以在教科书中向成千上万的人讲授的直接了当的技巧(这种意义是非凡的)。这表明选择恰当的符号并且给出它们的操作规则是极其重要的。

Leibnitz青年时期的奇思妙想：找到人类思想的真正的符号系统以及操作这些符号的恰当计算工具的宏伟梦想。 这种奇思妙想放到现在，就是制造具有人类智能的AI。

## George Boolean

George 主要考自学成才，学习了拉丁文，希腊语，法语和德语，并且能够以此写出数学论文。

George 16开始作为家庭的主要的经济来源，在远离家乡40英里的地方教书。后来自己经营学校并整日操劳讲授的课使得自己从一个数学学生成长为一个富有创造力的数学家。

当Boolean开始自己的工作时，人们开始认识到代数的力量来源于这样的一个事实，代表着量和运算的符号服从不多的几个基本规则或定律。

Boolean 《Cambridge Mathematical Journal》，《Philosophical Transactions of the Royal Society》. Boolean开始与几位顶尖的英国年轻数学进行通信。

逻辑关系，逻辑推理，谓词量化，命题逻辑和谓词逻辑(一阶，二阶，高阶)。

Boolean第一部关于逻辑作为数学的一种形式的革命性的专著出版时，他32岁，《The Laws of thought》出现在7年后。

关于新的事物的观念。

在Boolean的时代，逻辑学家将逻辑等同于亚里士多德的三段论，认为三段论是逻辑的全部。实际上，这是不对的，三段论只是逻辑推理的一个中特殊的应用。---- 新的思想是在对旧的思想的重新认识和发现，这就是马克思主义哲学的发展和扬弃。

Boolean发现可以使用同一种代数来表达推理，使用方程来表达命题为真，如比X=1,表示命题X为真，方程X=0表示X为假。

> 如果X，那么Y, X且Y =>	X	
>        ||  
>        ||  
> X(1-Y)=0,  XY=1    => X=1

利用这个思想，就可以将一些常见的命题表达成代数方程组，进而求出相应的方程的解。(代数方程求解部分是成熟的技术)

其实，日常生活中人们隐式使用这些推理，只有数学家才独具慧眼将这些智慧发掘出来。数学可以系统的概括极为复杂的逻辑推理，要以一种完备性为目的逻辑推理进行最终检验，看其是否包括了一切数学推理。

将复杂的形而上学推理还原成简单方程的运算。

Geroge Boolean的伟大之处在于一劳永逸的证明了逻辑演绎可以成为数学的分支。

## 费雷格

逻辑方法应用于算术基础--- 正当工作就要完成的之时，发现那时大厦的根基已经动摇。

费雷格提出将普通数学中的一切演绎推理都包含在一个完备的逻辑体系中，使用逻辑分析工具来研究语言的开拓性工作为哲学的发展提供了基础。1925年过世，死前认为自己一生工作毫无结果，死也没有受到学术界注意。

1879，出版了一本不到100页的《概念文字》[<Begriffsschrift>],副标题是：一种模仿算术语言构造的纯思维的形式语言。其试图找到一个能够包含数学实践中全部演绎推理的逻辑系统。Boolean以普通代数为出发点，用代数符号来标识逻辑关系；费雷格以逻辑为基础从而帮代数构造出来。Boolean认为用于表示其他命题之间关系的命题是二阶命题,费雷格则发现那些连接命题的关系也可被用于分析命题的结构，并将这些关系作为其逻辑的基础.---演化现代逻辑。

逻辑关系“如果..... , 那么......”----全称量词		“....且.....” ----存在量词	“.....或.....”

费雷格不仅仅对逻辑进行了一种数学处理，实际上他创造了一种新的语言。

Boolean用普通数学方法来发展一门数学分支，这其中包括逻辑推理，但是用逻辑来发展逻辑，有点循环依赖了。费雷格使用一种不同的方法来发展逻辑，将语法规则或这句法规则发展成一种人工语言,其逻辑第一次使用精密的数理逻辑体系在原则上包含了数学家们使用的全部推理。人们从Leibnitz和Newton的微积分中得到了丰硕的结果，数学家习惯使用的某些推理步骤是否正确？？----19实际的数系理论,自然数。

## 康托(Cantor) :在无限中探索

19世纪中叶，数学问题要求在精确描述中使用完成了的无限----创立一种关于实无限的深刻而一致的数学理论,将数学科学的方法带进无限领地。Cantor的无限过程引发的激烈的讨论，研究和争辩最终将成为通用计算机的发展提供重要的启发。

在柏林，年轻的Cantor在三个大数学家学习,无俸的编外讲师。爱德华.海涅建议Cantor研究与无穷级数有关的问题---三角函数，为了证明两种类型的不同级数收敛于同一值，将无穷级但做完成了的整体处理，并对其进行复杂的计算(??)。对于无限集中元素数目的困境：

1. 谈论一个无限集中元素的数目是没有意义的 
2. 某些无限集将与其一个子集具有相同的数目。

无限集和自然数之间建立一一对应的关系。用分数表示的数称之为有理数，√2这样的数连同所有的有理数被称之为代数数(可以作为代数方程的根)，不是任何代数方法的根的数称之为超越数。Cantor引入了基数(cardinal)这个概念,基数的概念在处理无限集时非常的有用。Cantor将无限集的基数称之为超限数，并创造了两个奇怪的记号来表示实数集(continuun-连续统)和自然数集的基数，在这两个基数之间不存在别的基数—连续统假设。

Cantor的对角线方法，1891年的论文。超限基数和序数都收在一个集合中，如果存在一个由所有基数组成的集合，那么他的基数应该是多少？？ 这其实就是一个悖论。

康德的批判哲学：1.纯粹数学如何可能？ 2.纯粹自然科学如何可能？

## 大卫.希尔博特(David Hilbert)

1737,乔治二世在哥延根建议了一个大学,主要受到David Hilbert的感召，世界各地的学生来到了这个无可争议的世界数学中心。Hilbert掀起了一场令人瞩目的运动，用数学来证明数学，从Hilbert到Turing对计算本性的洞察，其间发生了一连串怪事。

微积分的创立到Hilbert从未数学家，极限过程可以漂亮的应用在许多方面。很多结果只是对负号的纯形式的操作。需要对符号进行概念理解的问题层出不穷。

克隆内克：他的力量和权威迫使数学服从专断的哲学原理。主张数学证明应该是构造性的。

世界是不断变化的，但是有些东西却是不变的。数学家通常关心的是找到那些在其他食物百嗯话时保持不变的东西。代数不变量结构的证明,一流的数学家，对数学的基础抱有深刻的兴趣。Russell悖论，《Principle in Mathemetrica》证明了符号逻辑系统对数学进行完全形式化是绝对可能的。基于实数系统的处理极限的过程。

Hilbert纲领：数学与逻辑将通过一种纯形式的符号语言被发展，这样语言有两个视角，内-就是数学，每步演绎都可以完全的弄清楚；外-仅仅是公式和符号操作。形式系统，一阶逻辑的完备性，自然数公里系统，符号系统，维也纳学派，符号串—人工语言--通过自然数来编码符号逻辑系统，本质上讲任何形式逻辑系统都可以用自然数来编码。

Hilbert纲领的主要目的就是：用被认为仅仅构成PM的一个非常有限的子集的有限方法来证明像PM这样的系统的一致性。哥德尔通过自然数编码PM系统，发现了存在命令U，其本身在PM中是不可获证的--->PM的全部能力而言，也是不足以证明自身的一致性。

1930年,哥德尔(24岁)写的那些关于不可判定性论文，编码的45个公式，看起来像计算机程序，哥证明过PM中的证明代码可以使用PM的编码表示，在证明过成中出现的问题和设计编程语言和用这些语言编写程序时所面临的问题是相同的。

任何特定的形式系统，都会有数学问题超越它。哥德尔---独特的思考问题的方法，以及妄想症。

> 伟大的哥根廷学派领袖，伟大的大卫.希尔博特。

## 阿兰.图灵(Allen Turing)

查尔斯.巴贝奇—分析机。Leibnitz梦想将人的理性还原为计算。费雷格第一次给出似乎可以解释一切演绎推理的规则系统。Hilbert在找只要一阶逻辑的符号系统写出的某些前提和给定的，通过计算机程序总是可以判定的。传统的数学课程很大程度上是由被称之为算法的计算机程序够成的(值得思考，也就说传统中，我们学到的知识都是可以被计算机代替的，教育的优势消弭)。

数学家喜欢从两个方向解决一个困难的问题。一方面，试图考虑一般问题的特殊情形，另一方面则是把一般问题还原为某些特殊情形。Hilbert的判定问题：已经找到的算法的特殊情形与一般问题被还原成特殊情形之间的鸿沟。

Turing的认识：一种算法往往是通过一系列规则指定的，一个人可以以一种精确的机械方式遵循这些规则，就想按照菜谱做菜一样。Turing把关注焦点从这些规则转移到了人在执行他们时实际所做的事情，除去一些非本质的细节，可以用少数几种极为简单的基本操作，而不会改变计算结果。通用计算机器的数学模型。

任何计算所具有的本质特征：

1. 在计算的每一个阶段，只有少数符号受到了注意
1. 每个阶段所采取的行动仅仅取决于受到注意的那些符号以及计算者当前的心灵状态。
1. 任何计算都可以看作是下面的方式进行的：
1. 计算他哦嗯过在一条划分成方格的纸带上写下符号来进行
1. 执行计算的人在每一步都只注意其中一个方格中的符号
1. 执行的下一步仅仅取决这个符号和其心灵状态
1. 下一步是：在当前注意的方格里些下一个符号，然后把注意力转向它左边或右边的相邻的方格

通过Turing的计算概念的分析，结论：通过某种算法程序可计算的任何东西都可以通过一台Turing Machine来完成。运转的Turing Machine可以通过一系列五元组(集合)来表示。与物理装置不同，Turing Machine是数学抽象，所以不必受纸带量的限制。

停机问题的集合—可以看作是程序。Turing通用机。在Turing之前，机器，程序 和 数据这三种范畴是完全分离的。Turing通用机将这三种集合在一起，实际上，Turing的通用机本身就可以看成一个解释程序，通过解释一些列五元组执行，通用机的概念为括展了计算的概念，计算不仅仅是算数和代数运算，《American Journal of Mathematics》《Journal of Symbolic Logic》。

普林斯顿超越哥延根 -- 二战中，美国吸收了来自德国优秀数学家，哥延根学派的前领导人 -- xx. 外尔。

通用计算机的研制：工程师+数学家(逻辑学家)，计算机是一个混合的复杂的技术

> 貌似，讲到这里《逻辑的引擎》的笔记就结束了。书本上还有 冯 诺以曼 和歌德尔要介绍，不过，没什么好摘录的。

## 微积分历史 的部分摘录

> Here  and elsewhere we shall not obtain the best insight into things until we actually see them growing from the beginning.								--- Aristotle

问题是数学发展的源泉，在问题的激励下，启动思想，提出概念和方法，解决问题，然后将其组织成融会贯通的知识系统，于是新的学科就诞生了。

微积分诞生的四个问题：求积问题，求切问题，极值问题，研究运动的问题。

无穷步骤(infinite processes): infinity具有无穷的深度，迷人而困惑。

庄子：吾生也有涯，而知也无涯，以有涯逐无涯，殆矣。追逐微积分，the romance of soul in reason.

You cannot understand a theory(a theorem) unless you know how it was discovered. ----Mach

The true method of foreseeing the future of Mathematics is to study its history and its actual state.

> 上面其实在讲，读史使人明智，通晓历史，才能更好的认识未来。

> Mathematics is the science of infinity.						-----David Hilbert

> 备注: 微积分历史的警句已经结束了，下面是讲述的中国的数学大师的故事。

## 数学大师 笔记

### Introduction

科学发现，技术发明，都是创造性事业。需要特定的环境条件,依赖于特定素养的人,以及不与前人和同仁雷同的原始创新, 最最重要的还是创新方法。

创新方法的研究的直接和最终目标，在于正确的把握科学思维,科学方法,科学工具的意义和作用，确立科学思维创新，科学方法创新，科学工具创新的具体体制，提高科技工作着和公民的科学素养。

方法论之于科学研究，其重要新不言而喻。哲学是研究世界观和方法论的学科。科学是理性的绝对典范，方法论是科学取得成功的根本保证。思辩，唯物辩证法，特定领域的特定方法。方法论的结构性与规律性。

抛开过去，企图谈论新的方法论其本身就是矛盾的。理论与实践不可偏废,经验式方法研究。

中国数学的方法:引进, 消化, 转化, 超越--- 概括的很好，其他的学科也是一样的。落后之后如何赶超： 方法的创新, 这个问题具有一定的普遍性, 中国数学界采用的方法是：充分分析自己的不足，承认差距, 审时度势，将对方的先进之处尽快的吸收进来，采取”换新装，走出去，请进来”的战略。 

1. 换新装- 引进和推广西方数学符号

	先进的数学符号之于数学的发展意义重大。Peno : ”数学的一切进步，都是对一引入符号的反应”  Leibnitz： 符号的巧妙和符号的艺术是人们绝妙的助手，因为他们使思考工作得到了节约。Helbert: 朴素时期,形成时期, 批判时期。 数学的进展: 解答问题, 理解数学现象,引进好的符号和方便的算法。
	符号演化的方向: 更加清晰,更加抽象,更加简练的方向发展。代数，几何，解析论, 微积分, 概论论,拓扑学,
2. 走出去 -  将优秀学生送进主流的数学世界

3. 请进来 -  邀请国外著名数学家来华交流

### 数学大师成长的方法前提

1. 刻苦勤奋,锲而不舍

  华罗庚: 聪明在于积累，天才来自勤奋。  
  丘成桐: 路漫漫其修远兮，吾将上下而求索。系统的做题和有针对性且实用的思考使得其可以快速掌握课程涉及的新的概念和思想。付出不一定就有收获，但是没有付出就肯定没有收获

2. 收敛与发散两相宜

  库恩: 《必要的张力:科学研究的传统和创新》。收敛性思维(科学研究要牢牢地扎根于科学传统中)和发散性思维(保持开阔的视野，具有高度灵活性，时刻准备抛弃旧的思想，开辟新的方向)要保持必要的张力。很多的数学家的文学素养很高。丘成桐：兴趣爱好比较广泛，涉及的数学领域宽泛。
3. 追随大师，从照着做开始 

学习分为两种: 1.以文字等形式表现出来的确定性知识  2. 更新知识的能力  

自然科学有一些只能意会无法言传的前提，大师的日常工作便是展示重大前提及其个人直觉 。这也是为何伟大的科学家总是出自大师门下， 

4. 关注前沿占先机

  紧紧把握数学前沿，才能高屋建瓴，指导自己作出最好的创新。关注前沿有两种： 1.随波逐流  2.引领潮流  
  努力与良好的悟性想结合。

5. 求真知，远名利

淡泊名利，心性高远，为求真知，满足好奇心而将毕生的精力奉献给数学事业。科研和求知比获得名誉和奖章更加重要：数学---揭示自然界最本质的学科。丘成桐2006在华中科技大学的演讲说,年轻人要做学问的抱负，及做学问的高尚情操。具体来说包含五个方面: 

> 1) 求不朽之业。 所谓不朽，乃如《左传》曰：”太上有立德，其次有立功，其次有立言，虽久不费，此之谓不朽”  
> 2) 要有承前启后的使命感。《几何原本》和《自然哲学的数学原理》都是承前启后的著作。历史是联系发展的, 任何人都不可能独立于历史而存在  
> 3) 有所见，有所思，而欲示诸后人，传著后世  
> 4) 浓厚的好奇心驱使，希望凭借观察，推理，来了解大自然的结构，寻找宇宙的真谛。 伟大的科学家都有强烈的好奇心，数学上的很多领域的探索也是基于数学家的好奇心  
> 5) 科学家和文学家为了寻找一个美的结构，可能穷尽毕生的精力。近代的统一场论，某些晶体的结构，数论或几何上种种雅致的命题，都引起了热烈的研究，而追寻纯美则是这种研究的主要动力。

6. 徜徉于数学之美 数学本身就是美的存在

吴文俊：在武崇林将代数和实变函数论讲的生动有趣，精彩异常，吴感受到了数学的魅力。其课后刻苦自学，反复阅读几种著作，在数学上打下了坚实的基础，钻研点集拓扑学经典著作以及波兰数学期刊《数学基础》上的论文，前几卷每卷必读，将重要的内容做笔记，在名师的指导下研究。

苏步青，读书不忘救国，救国不忘读书。

周炜良35岁，在博士毕业(经济学博士)后，才开始从事数学研究，其为人极其淡薄名利，爱真知而不好求名利。巴尔扎可30岁以后，在被债务逼的走投无路时，开始写作，成为一代文豪。

世界不缺乏美，要有一双发现美的眼睛。没有一步登天的绝世秘技，踏踏实实的走好每一步就行了。既然知道自己是个蠢蛋，那前进的方法就固定下来了，就用最笨的方法，一步一步的踏踏实实走过来就好了。不要耍什么小聪明偷懒，最愚蠢的莫过于不知自己的愚蠢，反而因为小聪明而沾沾自喜，窃为不可为之。

### 华罗庚的学术生涯和卓越成就

一, 自学成才的少年

小时候，华没有表现出过人的天赋,成绩很不理想。初中时依然贪玩, 作业考试潦草不堪。幸遇良师，放弃贪玩. 掘九井不得，不如掘一井为之愈。擅长直接法。

学习有如爬梯子，一步一步地往上爬。想要一步登天，是会摔跟头的。华在小店里干活，刻苦顽强的和命运抗争。看到这一点，我就想到自己，自己在不愁吃不愁穿的，每天有24小时可以自由支配的条件下，都在干些什么啊，看了一小会儿源代码就倦怠了，想玩了—看漫画看动漫看新闻，搞的自己满脑子都是这些东西，我要清空掉这些东西，这些对我的人生一点用处都没有，除了浪费时间，这是在慢性自杀，不行，绝对不能再这样了。

熊庆来—清华，哈代--剑桥，把虚荣的名誉放到一边，注重的是真才实学。华在剑桥，听了七八门课，记了很厚的笔记，回国后又重新整理了一遍，仔细地加以消化。两年的剑桥之旅，华从数学青年成长为世界一流的数学家。

西南联大，从数论-->矩阵方法(将自守函数论，矩阵几何学，典型群论与多个复变函数论集中在一起研究)，华的左腿还是残疾的。

> 经历了前22年的岁月，我终于在大三时开始觉醒了，开始明白什么如何在从大学教育中获利，开始知道什么是重要的，什么是次要的，逐渐明白马克思主义的哲学要素(联系，发展，唯物辩证法，共性与特性，矛盾的统一论等)，意识到普遍教育只能培养出普通人才来，想要成为精英，想要向正太分布的右边靠拢，不开点小灶是不行的，开小灶的方法有这样的一些：1. 找个牛逼的老师  2. 看领域中最经典的教材。开始知道适当的囫囵吞枣是有好处的，先Bottom-Up在Top-Down的思路很值得学习。就目前而言，牛逼的老师这一点应该是满足了，但是为什么自己有点自暴自弃，没有能够全身心的集中在软件测试上，很明显，这是一件顶重要的机会，这是命运给与我的一个绝好的chance，没什么自己不好好的把握，难道还想在失去后，在某个凄苍的日子里，流下悔恨的泪水吗？ You must take seriously about this chance。 古人云：时不我待，机不可失，时不再来。 够了，我已经失去了前23年，现在的我是处于我人生中身体最棒，大脑最灵活的，精力最旺盛的时候，简单来说，就是可以干出一番事业的时候。在这样珍贵的时间里，怎么能允许浪费自己的生命。  
> 我们现在有比过去更好的条件，换而言之，就是我们的舞台比过去更为广大，在这样的舞台上，你想演奏出什么样的旋律。请你好好想想，认认真真脚踏实地谱写，不要辜负了老师的教导，家长的期待以及我(作为你的共同体的存在)的希冀。汝尚不愚钝，快马加鞭，他日必成大器。

### 华罗庚的思维创新

从懵懂无知的贫困少年到蜚声世界的数学大师。1. 自身刻苦学习，努力钻研 2.个人魅力的创新思维

1. 下棋找高手，弄斧到班门

开始搞研究时，最难把握的就是质的问题，也就是深度的问题。华具有善于抓住别人最好工作的不可思议的能力，并能确切的指出他们的结果中那些是可以改进的，广泛阅读并掌握制高点。

逆向思维追求思想的更高境界和认可的完美性，用批判的眼光看问题和论文。

2. 独立创新，立足长远

数学普及：面向大众，面向中小学生。数学竞赛：分析问题，解决问题的能力。

3. 教学相长，团结协作

讨论班的方式学习，不停的问题，打破砂锅问到底的做法，授课写书的方式。

数学中典型问题研究：研究典型问题以及由其衍生的相关的方法，有利于从整体上把握问题

4. 数学乃一整体，应该标本兼治 

标则是偏向于应用，本则是纯理论研究。 数学是一门基础科学，是对现实世界的空间形式和数量关系的研究。随着人们对自然界认识的提高，形成了一门包含很多分支的庞大学科，有偏基础，有偏抽象，有和生产实践联系紧密

5. 数学很美，令人陶醉 

6. 不拘一格选人才

7. 善交往, 精诗词

### 华罗庚的方法和工具创新

初等与直接解题法(将不相干的东西排除掉，抓住最本质的东西的演算方法)，

循序渐进学习法: 从厚到薄(学习接受的过程)，从薄到后(消化，提炼的过程)

一条龙教学法:将所有的基础课合并在一起教，数学是一门内在联系紧密的学科。整体数学观，内在紧密联系。

五步法论: **调查研究与系统分析 → 建立数学模型 → 求解所建的数学模型(算法与软件) →  解释结果 → 提取有价值的问题进行理论研究 → 现实中的实际问题 →  再调用研究 。**

在建立数学模型中，要把自己的思想，技巧与实际问题泡在一起，与多学科合作者，实际工作着泡在一起，需要一个艰苦的过程才能抓住问题的实质。模型算法统一考虑，将五步缩减为四步。

### 统筹法和优先法 

陈省身
	
研究与数学所工作并重，整体微分几何的开创者。结交好友无数，获得荣誉无数。研究数学要淡泊名利，培养的学生很多，喜欢和学生一起合作文章。

### 陈省身的思维和理念创新
	
陈的杀手锏： 不拘泥于某种成见，善于在各种思维，方法中游刃有余。规则需要运用，但规则的运用却无规则可遁。其方法主要： 

1. 扬长避短，独立发展 

2. 研究贵独创，直接对每个问题从根本上下功夫。做学问的路径：照者讲—下功夫将经典弄清楚。接着讲--与时具进，在已有成果的基础上在进一步；对这讲--对已有的方法进行批判，从现实，需要出发，从批判的角度清理已有的观点； 从头讲—没有现成的东西，属于前人不曾做过的，或这做得不成熟的工具，不怕第一个吃螃蟹。

3. 数学价值观： 做好的数学, 洞察力—一眼看到低，敏锐的捕获关键点的能力。

4. 数学好玩，玩好数学:  刻苦勤奋、执著最求，随时随地地想数学、淡薄名利、深厚的中华文化涵养

5. 研究数学需要直觉与逻辑的统一: 直觉和逻辑是一对互补的概念，依靠理论的抽象方法，进行逻辑的,严谨的分析,综合,推理和判断，也要借助直觉，灵感等思维方式取得突破。理性思维,直觉思维。数学研究需要两种能力, 一是丰富的想像力, 能够提出一个理论框架，构作概念，提出问题，找到关键；另一种能力是强大的攻坚能力，能够把一个一个具体对象构造出来，把不变量找出来，把要找的量准确的计算出来。数学设计师,数学工匠。

6. 数学没有诺贝尔奖是好事, 嘉当62当选院士, Riemann终生没有获奖 

> 数学的价值不能用奖来衡量
> 数学家的价值不能用奖来衡量
> 得奖对人有激励作用,但不应该把名利看的太重

7. 几何的实在性, 物理几何是一家

8. 纯粹数学与应用数学不可截然区分

### 陈省身的方法和工具创新

陈的方法创新主要在求学,治学,教育,管理，为人处理等方面。方法的创新体现在对传统方法的独出心裁的运用。一方面,将这些方法运用的张弛有度。另一方面, 巧妙的将这些方法与自己的整个科研生涯结合的天衣无缝。

一, 求学方法: 从高师,选大题  姜立夫, 清华数学系, 布拉施克, E.嘉当

二, 治学方法：坚持执著,勤奋刻苦

陈在一直工作在数学的前沿, 数学大师的学术生命力着实令人惊叹。直到过世前都一直活跃在数学的前沿。陈在追随嘉当时, 要全力应对嘉当(写的论文太过深邃而让人不忍卒读)提出的问题,工作非常的紧张。陈的用功程度不及华先生(华先生该有多用功啊！！)。不行啊，我实在太懒散了，我要想华先生学习。陈早上5点多就醒了，难怪我做不出成就，原来是不够用功。

三, 研究方法: 数学既是个人学问,有需要交流合作

同一个对象，可以从不同的视角，用不同的数学工具研究。数学家不一定要做主流数学，但是一定要置身主流数学界。

四, 教育方法: 做人格典范, 反对学徒制

主动和年轻的数学家交往, 人才不是培养出来的，是靠努力,灵感，机遇冒出来的。因材施教，重视学生的自学能力。读别人的书是欠别人的债，出来混，欠债是要还的。

五, 行事方法: 有所为,有所不为

六, 处事方法: 功夫在”学“外 爱国情操, 独特个性, 宽广胸怀

七, 数学工具的创造性运用： 陈的工具: 解析技术, 代数,拓扑,李群,复分析等 

## 后记

粗粗的看了一下这篇挂着《逻辑的引擎》笔记的牛头，买着各种各样的马肉。这篇笔记是学习数学愿望最强烈的时候，记录下的各种各样的所谓的方法论。结果呢，展开了一个泰勒级数，去掉了
十几分钟。做了一道求极限的考研题，花了十几分钟，一看答案，居然做错了，看了半天参考答案，思考了良久才知道好像大概这么回事。

所以呢，看别人的成功和心灵鸡汤呢，很容易。轮到自己动手去干的时候，发现自己愚钝脑袋瓜根本不是干这回事的人。这就是理想与现实的差距。人生，如果，没了阿Q精神和自嘲，很难活下去的。