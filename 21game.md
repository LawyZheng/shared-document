目标：写一个 "21点" 的游戏程序。

规则基本就是先玩家和庄家(电脑)各抽两张，玩家可以看到庄家的一张牌(底牌看不到)。

玩家可以进行叫牌，直到手头的点数大于21或者手牌数量达到5张。

玩家叫牌结束后，庄家(电脑)可以进行叫牌。

庄家(电脑)叫牌的规则是，直到手上的点数大于21点或者点数大于等于玩家的点数才会停止。

其实这算不上是一个简单的程序了，说他简单，事实上还是有点复杂的。

所以决定采用先写主体框架，再细化的方式去写，可能会比较易懂一些。

那就开始呗。

---

最初先导入一个自带的random库吧，用来进行洗牌操作和随机抽牌操作。

```python
import random
```

先写一个main函数，其实就是主程序。

这个函数主要包含了游戏的基本流程：

初始化扑克卡池、相关变量 -> 洗牌 -> （进行每个回合游戏 -> 统计当前分数） -> 退出

"()" 内表示循环的部分，直到玩家选择停止游戏才会跳出循环。

中间出现了一些还没有定义的函数，没关系，先写上去，之后再补上具体的函数就可以了。


```python
def main():
    input("按Enter键开始游戏。")
    # 洗牌
    print("正在进行洗牌...")
    poker_pool = random_card()
    print("洗牌结束，游戏正式开始。")

    game_round = 1
    # 前面表示电脑得分，后面表示玩家得分
    total_scores = [0, 0]

    while(True):
        print("-"*20)
        print("第%d回合开始。" % game_round)

        # 如果卡池里的牌少于15张，就重新洗牌
        if len(poker_pool) < 15:
            print("由于卡池里的牌少于15张，正在进行重新洗牌。")
            poker_pool = random_card()
            print("重新洗牌结束，游戏继续。")

        # -1:电脑获胜  0:平局  1:玩家获胜
        result = every_round(poker_pool)

        #判定结果
        if result == -1:
            print("庄家获胜。")
            total_scores[0] += 1
        elif result == 1:
            print("玩家获胜")
            total_scores[1] += 1
        else:
            print("平局。")

        #打印当前比分
        print("当前比分为 >>>>>>  庄家 %d - %d 玩家。" %
              (total_scores[0], total_scores[1]))

        #是否继续进行下一局
        if_continue_game = continue_game()
        if if_continue_game:
            game_round += 1
        else:
            print("游戏结束。")
            break
```


回顾一下main函数。

其中出现了洗牌函数random&#95;card()，进行每一局游戏函数every&#95;round()，询问是否继续的函数continue&#95;game()。

为了说明方便，就按 洗牌函数random&#95;card()、询问是否继续的函数continue&#95;game()、进行每一局游戏函数every&#95;round()， 这样的顺序写吧。

事实上先写哪个函数无所谓，只是最后一个函数相对比较复杂，所以在这里放在最后写。

---

先写 洗牌函数random&#95;card()。

其实该函数的作用就是初始化一副扑克牌的卡池，然后进行洗牌。

这个函数不用传入任何参数，返回值是一个列表，用来表示扑克牌的卡池。

```python
def random_card():
    poker_pool = list()

    # 初始化一组牌
    card_types = ['黑桃', '红桃', '梅花', '方块']
    # 四种牌的类型
    for card_type in card_types:
        # 没种类型有13张，
        for i in range(13):
            # 0 的时候，表示A， 如黑桃A
            if i == 0:
                poker_pool.append(card_type + 'A')
            # 1~9 的时候，表示对应的数字， 如黑桃2、黑桃10
            elif i < 10:
                poker_pool.append(card_type + str(i+1))
            # 10 的时候，表示J， 如黑桃J
            elif i == 10:
                poker_pool.append(card_type + 'J')
            # 11 的时候，表示J， 如黑桃Q
            elif i == 11:
                poker_pool.append(card_type + 'Q')
            # 12 的时候，表示J， 如黑桃K
            elif i == 12:
                poker_pool.append(card_type + 'K')

    # 洗牌
    random.shuffle(poker_pool)

    return poker_pool
```

这样洗牌函数就写完了。

接着写 询问是否继续的函数continue&#95;game()。

该函数的功能很简单，就是询问玩家是否继续进行下一局游戏。不用传入参数，返回布尔值。

需要注意的是当玩家输入别的东西的时候，应该递归调用该函数，直到玩家输入有效值。

别忘了最后else部分的return语句，否则在进行递归调用的时候函数不会返回任何值，会导致最终的函数返回值为None。(在这里就不做赘述了，有兴趣的同学可以自己去测试一下)

```python
def continue_game():
    if_continue = input("是否继续进行游戏？ (Y/N) >>>>>>>>>")
    if if_continue.upper() == 'Y':
        return True
    elif if_continue.upper() == 'N':
        return False
    else:
        print("无效的输入，请重新输入。")
        return continue_game()
```

好，这样询问继续函数就写完了。

还剩 进行每局游戏的函数every&#95;round()。

该函数需要从主函数接收一个参数poker&#95;pool，即表示扑克牌的卡池。

返回值则为这一局游戏的结果，-1表示庄家(电脑)获胜，0表示平局，1表示玩家获胜。

这个函数可能比较复杂一点。还是一样，先写流程：

双方发牌每人两张 -> (玩家叫牌 -> 计算玩家点数大小) -> 庄家叫牌 -> 计算庄家点数大小 -> 判断结果

"()" 里面为循环，直到玩家停止叫牌，或者玩家手牌数达到5张，或者玩家超过21点，才会跳出循环。

函数里面也出现了一些没有被定义的函数，同样不急，先写框架，再细化写具体的函数内容。

```python
def every_round(poker_pool):
    # 返回 电脑胜-1 or 平局0 or 玩家胜1
    # 叫牌规则：
    # 玩家在手牌小于5张，且点数小于21点的时候可以叫牌。
    # 电脑在手牌小于5张，且点数大于等于玩家点数的时候可以叫牌。
    
    #初始化两个列表，表示双方的手牌。
    pc_hand_card = list()
    player_hand_card = list()

    # 初始化发牌，电脑和玩家各发两张
    for i in range(2):
        pc_hand_card.append(get_one_card(poker_pool))
    for i in range(2):
        player_hand_card.append(get_one_card(poker_pool))

    # 查看双方的手牌
    print("庄家的手牌为: [**, %s]" % pc_hand_card[1])
    print("目前你的手牌为: {}".format(player_hand_card))

    # 计算双方目前的点数
    pc_score = score_count(pc_hand_card)
    player_score = score_count(player_hand_card)

    # 玩家是否继续叫牌
    if_get_next_card = if_get_next()
    while if_get_next_card:
        # 是否持有5张手牌
        if len(player_hand_card) == 5:
            print("已经有5张手牌，无法继续叫牌。")
            break

        # 玩家叫取一张牌
        player_hand_card.append(get_one_card(poker_pool))
        print("目前你的手牌为: {}".format(player_hand_card))

        # 计算玩家点数，是否大于21。
        player_score = score_count(player_hand_card)
        if player_score > 21:
            print("玩家点数大于21点，本回合游戏结束。")
            print("庄家的手牌为: {}".format(pc_hand_card))
            return -1

        # 是否继续叫牌
        if_get_next_card = if_get_next()

    # 庄家叫牌
    print("玩家停止叫牌。")
    print("庄家正在进行叫牌...")

    pc_get_card(pc_hand_card, player_score, poker_pool)
    pc_score = score_count(pc_hand_card)
    print("庄家叫牌结束。")
    print("庄家的手牌为: {}".format(pc_hand_card))

    # 结果判断
    if pc_score > 21:
        print("庄家点数大于21点，本回合游戏结束。")
        return 1
    elif pc_score == player_score:
        return 0
    else:
        return -1 if pc_score > player_score else 1
```

看一下这个函数里的具体内容，一共有四个函数没有被具体的定义，分别是:

叫牌函数get&#95;one&#95;card()、分数计算函数score&#95;count()、询问是否继续叫牌的函数if&#95;get&#95;next()、庄家叫牌函数pc&#95;get&#95;card()。

再依次进行具体的定义呗。

这里需要注意的是，因为在 pc&#95;get&#95;card() 里面，我使用到了get&#95;one&#95;card() 和 score&#95;count() 这两个函数。所以这两个函数需要在pc&#95;get&#95;card() 函数定义之前就先被定义好才行。

那就先写 叫牌函数get&#95;one&#95;card() 呗。

该函数的作用就是从卡池里抽取一张卡，即从卡池里删除一张卡，并返回被删除的这张卡。

当然因为需要卡池这个变量，所以就需要传入一个参数poker&#95;pool，返回一个字符串，表示抽取的这样卡。

注意这里虽然对poker&#95;pool变量进行了修改，但不需要返回被修改后的变量，就可以达到改变原有的变量的目的。(具体可以去看看Python的基础知识)

```python
def get_one_card(poker_pool):
    return poker_pool.pop(random.randint(0, len(poker_pool)-1))
```

然后再写 分数计算函数score&#95;count()。

该函数的作用是计算手牌目前的点数，所以需要传入一个表示手牌的参数hand&#95;card，返回该手牌的点数。

关于A的点数计算问题，可以先把它当做1进行计算，之后再查看如果当做11计算结果是否大于21就可以了。

```python
def score_count(hand_card):
    # J Q K 都为10点。
    # A先作为11点，如果总点数大于21点的话，则作为1点。

    score = 0
    have_A = False

    # 查看每张牌的末尾字符是什么。
    # 如果是0，则应该抽到的是10点，如 梅花10。如果是其他数字，则应该是对应点数。
    # 如果是J Q K，则为10点。
    for each_card in hand_card:

        if each_card[-1] in ['J', 'Q', 'K', '0']:
            score += 10
        elif each_card[-1] == 'A':
            score += 1
            have_A = True
        else:
            score += int(each_card[-1])

    # 如果有A，则进行额外的点数添加。
    # 由于两张A如果都最为11点计算的话，结果必然大于21点，因此最多只有1张A最为11点计算。
    if have_A and score < 12:
        score += 10

    return score
```

这里有一个误区，就是有同学会觉得不必反复全部重新计算点数，只需要将这次抽到的牌加上之前抽到的牌的点数就可以得到手牌的点数了。

其实这种算法是有BUG的，比如你原本的手牌是 [A, 9]。这时的点数是20。当你再抽一张牌，比如你抽到了一张5。那么你的手牌变为 [A, 9, 5]，这时的点数变为了15，而不是25。当然算法不止这一种，比如可以设置一个列表来储存点数，最后选取最近接近21但不大于21的值就行了，有兴趣的同学可以去尝试一下。

接着写 询问是否继续叫牌的函数if&#95;get&#95;next()。

该函数的作用就是询问玩家是否继续叫牌。不需要传入参数，返回一个布尔值。

与之前询问是否继续游戏的函数大体上是一样的，就不再具体介绍了。

```python
def if_get_next():
    if_get_next_card = input("是否继续叫牌？ (Y/N) >>>>>>")
    if if_get_next_card.upper() == 'Y':
        return True
    elif if_get_next_card.upper() == 'N':
        return False
    else:
        print("无效的输入，请重新输入。")
        return if_get_next()
```

最后就是 庄家叫牌函数pc&#95;get&#95;card()。

该函数的作用就是庄家进行叫牌，叫牌规则之前已经说过，就不再赘述了。

因为需要调用get&#95;one&#95;card()和score&#95;count()，函数所以需要传入这两个函数所需要的参数poker&#95;pool和pc&#95;hand&#95;card。另外因为需要与玩家分数进行比较而决定是否叫牌，所以需要传入参数player&#95;score。

由于该函数实际上是对庄家手牌变量的修改，即列表的修改，所以不需要返回值。

```python
def pc_get_card(pc_hand_card, player_score, poker_pool):
    # 电脑叫牌。
    # 直到手牌点数大于玩家手牌点数。
    # 或者手牌数量等于5张。
    # 或者庄家手牌点数大于等于21，才停止叫牌。
    while(score_count(pc_hand_card) < player_score and len(pc_hand_card) < 5):
        pc_hand_card.append(get_one_card(poker_pool))

        # 如果电脑的点数已经达到21点，或者大于21点，则不用继续叫牌，退出循环
        if score_count(pc_hand_card) >= 21:
            break
```

OK，至此该程序已经全部完成了。

em... 其实这个程序玩起来玩家的胜率是极低的。毕竟庄家是上帝视角，而且叫牌规则也很粗暴，是一直抽到比你大，或者干脆超过21点自爆的非常激进的方式。

想要修改成庄家叫牌前先计算再抽取一张卡的胜率，如果高于50&#37;的话就继续叫牌，否则停止叫牌，这样可能会更公平一点吧？(其实我也不知道是不是更公平一点)

## 改进一下

em.. 对庄家(电脑)的叫牌规则进行了重写，改为查看牌库里的牌，计算抽一张牌的获胜和平局的概率小于30&#37;时，才会停止叫牌操作。

之前说50&#37;的，但发现这样庄家太过保守，反而造成了庄家难以获胜的情况。至于概率到底多少才合适，我也不太清楚，毕竟我不是数学专业的，数学也不太好。

回到正题，修改的地方有两处，一是修改了 庄家叫牌函数pc&#95;get&#95;card()，然后额外定义了一个用来计算概率的函数。

```python
def pc_get_card(pc_hand_card, player_score, poker_pool):
    # 电脑叫牌。
    # 手牌点数大于等于玩家手牌点数，停止叫牌。
    # 手牌数量等于5张，停止叫牌。

    # new update
    # 庄家叫牌前先计算拿到牌的获胜和平局的概率，小于30%，停止叫牌

    # 计算目前庄家手牌的点数
    pc_score = score_count(pc_hand_card)
    while(pc_score < player_score):
        # 如果手牌数等于5张，则不能再继续叫牌
        if len(pc_hand_card) == 5:
            break

        # 如果相差>=11，那么则无需进行获胜概率的计算，直接进行叫牌
        # 如果继续叫牌的获胜和平局概率小于30%，则停止叫牌
        if player_score - pc_score < 11 and win_or_equal_posibility(pc_score, player_score, poker_pool) < 0.3:
            break

        # 抽取一张牌
        pc_hand_card.append(get_one_card(poker_pool))
        # 重新计算手牌的点数
        pc_score = score_count(pc_hand_card)
``` 

代码应该不难理解，我也都写了比较详细的备注了。

可以看到这里我多定义了一个函数win&#95;or&#95;equal&#95;posibility()用来计算获胜和平局的概率。

也很简单，就是遍历卡池里现有的牌，统计抽取后能获胜和平局牌的数量，再计算总概率。

因此需要传入，庄家目前手牌点数pc&#95;score，玩家手牌点数player&#95;score，卡池poker&#95;pool这三个参数。

返回一个浮点数，表示概率。

实际上大体上的逻辑和计算手牌点数的函数差不多，所以也不多做赘述了。

```python
def win_or_equal_posibility(pc_score, player_score, poker_pool):
    delta = player_score - pc_score
    win_count = 0
    cards_number = len(poker_pool)

    # 查看牌库里的每一张牌
    for each_card in poker_pool:
        # 卡池里点数为10的牌
        if each_card[-1] in ['0', 'J', 'Q', 'K']:
            if pc_score + 10 < 22 and delta <= 10:
                win_count += 1

        # 卡池里的A
        elif each_card[-1] == 'A':
            # 如果庄家点数 < 11 的话，抽到A应该作为11点计算
            if pc_score < 11 and delta <= 11:
                win_count += 1
            # 如果庄家点数 >= 11 的话，抽到的A应该作为1点计算
            if pc_score + 1 < 22 and delta <= 1:
                win_count += 1

        # 卡池里的其他牌，即2-9
        else:
            if pc_score + int(each_card[-1]) < 22 and delta <= int(each_card[-1]):
                win_count += 1

    return win_count/cards_number
```

em.. 总的来说，这个逻辑还是以面向过程为主的，可以看到我写代码的逻辑基本都是先写出执行的流程，再补上所需要的功能函数。这似乎好像有点不太符合OOP面向对象的编程逻辑？

毕竟Python是一门面向对象的语言，所以我决定下次有空就把这个程序改成面向对象的！



[Github源代码链接 - 版本一](https://github.com/LawyZheng/greedyai_learning/blob/master/greedyai_week3.py)

[Github源代码链接 - 版本二](https://github.com/LawyZheng/greedyai_learning/blob/master/greedyai_week3_v2.py)

[知乎链接](https://zhuanlan.zhihu.com/p/78623369)
