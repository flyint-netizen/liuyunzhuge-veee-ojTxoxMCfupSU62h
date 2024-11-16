
相关：


[python编写的扫雷游戏](https://github.com)


[如何使用计算机程序求解扫雷游戏](https://github.com)


![image-20241115143032377](https://img2023.cnblogs.com/blog/1088037/202411/1088037-20241115143101728-1670039513.png)


本文中实现的《扫雷》游戏的AI解法的项目地址：


[https://openi.pcl.ac.cn/devilmaycry812839668/AI\_mine\_game](https://github.com):[楚门加速器p](https://tianchuang88.com)


该项目的解法效果：


![image-20241115125404601](https://img2023.cnblogs.com/blog/1088037/202411/1088037-20241115143102519-706899873.png)


之前介绍了网上的一些解决《扫雷》游戏的一些解法，包括DQN和启发式等AI算法，看着这些的实现个人有些手痒，于是就花了些时间自己用python代码实现了一个启发式方法求解《扫雷》游戏的算法。


求解《扫雷》游戏，很多人给出了很多启发式规则，但是实际上我个人认为就两条或者说三条，其他的那些规则属于在这两条或者说是三条规则基础上衍生的，实际上用这两条或三条规则就足够。


第一条：


如果一个格子揭开后其显示的周围有雷的数量和周边8个格子中未揭开的格子数量相同，那么说明这几个未被揭开的格子都是有雷的；


第二条：


如果一个格子揭开后其显示的周围有雷的数量和周边8个格子中标记的有雷的格子数量相同，那么说明其他几个未被揭开的格子都是无雷的。


第三条：


之所以第三条是独立出来的，因为这一条并不等同于前两条那么基础和必要，或者说没有前两条规则那么肯定不能行，但是没有第三条规则其实很多情况下也是可行的。但是有第三条的话会一定程度上提高我们的胜利比率。（**注意：**扫雷游戏在很多情况下是没有确定的胜利的情况的，也就是说在某种情况下能否胜利是要看概率的，而我们写的算法代码可以看作只是为了去尽可能接近这个概率而已）


第三条规则就是一个格子显示雷的数值在某些情况下是可以通过附近24个格子的数值进行优化的，比如一个格子的坐标为（x, y）那么在其\-2，\+2的范围下的其他标有数值的格子是可能和其进行化简的。


比如：一个格子坐标为（x, y）数值为3，其周边8个格子标有雷的数量为0，未被揭开的格子有四个，我们假设这4个未揭开的格子的真实有雷和无雷的情况分别用0或1表示，那么我们可以得到下面这个等式：


a\+b\+c\+d\=3


而坐标为（x\+2, y\+2\)的一个格子可以得到下面的等式：


a\+b\+c\=2


那么我们可以得到结论，那就是有无雷情况用d表示的格子肯定有雷，因为:


set(a,b,c,d)\-set(a,b,c)\=d


3\-2\=1


同理，如果假设（x\-2， y\-2）的一个格子的数值表示为a\+b\+c\+d\+e\=3，那么我们可以判断出e这个表示的格子的情况一定为无雷的。


其实，第三条虽然是一种集合运算，但是其模拟的却是一种人类所用的数学推理的方法，在考虑使用这个方法之前曾经考虑过用python的线性代数运算library来实现这种数学推理的计算，但是感觉这么搞有些离谱，总觉得这个问题应该不至于到这个程度，然后盯着游戏画面好久，突然灵感一现，发现这个推理过程虽然看似是一个矩阵运算那种线性代数运算，但是如果只是小范围来看这个其实就是很简单的线性代数，因为每个变量的取值只能为0或1，并且可以完全使用for循环的方式加上set集合运算的方式实现消元化简，从而使用较为简单的编码方式就可以实现这种推理过程，而不需要搞进来一个线性数学的library，不过后来发现GitHub上的其他人实现的也都是大致用这种set结合加for循环的方式实现消元操作，看来这估计是正解。


在这三条的规则基础上我还加入了概率判断，这一点体现在随机进行格子选择时，我们可以根据一个未知格子周边（附近8个）已知格子的显示数值和这8个格子与其周边的另个格子中未知雷的数量计算出这个已知格子对这个目标的未知格子的又雷概率，我们可以取这个未知格子周边已知格子推断出的有雷概率取max，然后再计算出当前一共有多少未知格子和多少雷，然后计算出完全随机选格子的有雷概率，然后再在其中选择概率最小的，这样就能找到最小概率有雷的格子，以此实现最小概率触发雷。


再往下就是使用前面的前两个规则判断刚选的格子（假设此时为触发雷）是否可以判断出周边格子的情况。但是，这里我又加入一个规则，那就是一个格子被揭开显示数值后会影响其周边8个格子中未知格子的推理，导致这8个格子中的未知格子有可能被推理出来，因此我们将每个先揭开的格子或标记的格子其周边的格子保存起来，然后再对这些保存的各种进行判定，看其周边的格子是否可以被推理出来。


需要注意的是，一个格子被揭开显示数值或者被标记有雷后，其周边受影响可能被推断出的格子为附近8个，这时可以根据之前给出的前两条规则进行判断和推理，但是一个格子被揭开显示数值或者被标记有雷后其周边24个格子包括其自身，也就是共25个格子的set集合都有可能被推理化简，也就是之前所谓的for\+set实现的消元操作。通过将这些规则结合在一起也就有了本文给出的代码实现。


最终的代码实现：



```


|  | import numpy as np |
| --- | --- |
|  | import random |
|  | from typing import List |
|  |  |
|  |  |
|  | def belong_to(h, w, H, W): |
|  | near = [] |
|  | for i in range(h-2, h+3): |
|  | for j in range(w-2, w+3): |
|  | if i>=0 and j>=0 and iand jand (i,j)!=(h,w): |
|  | near.append((i, j)) |
|  | return near |
|  |  |
|  | def near_by(h, w, H, W): |
|  | near = [] |
|  | for i in range(h-1, h+2): |
|  | for j in range(w-1, w+2): |
|  | if i>=0 and j>=0 and iand jand (i,j)!=(h,w): |
|  | near.append((i,j)) |
|  | return near |
|  |  |
|  | def mine_count(h, w, real_state:np.array, H, W): |
|  | count = 0 |
|  | for i, j in near_by(h, w, H=H, W=W): |
|  | if real_state[i][j]==1: |
|  | count += 1 |
|  | return count |
|  |  |
|  |  |
|  | class Env(): |
|  | def __init__(self, H, W, N): |
|  | self.H = H |
|  | self.W = W |
|  | self.N = N |
|  |  |
|  | # real state中0表示无雷，1表示有雷 |
|  | self.real_state = np.zeros((H, W), dtype=np.int32) |
|  | self.mine = set() |
|  | while len(self.mine)!=N: |
|  | self.mine.add(random.randint(0, H*W-1)) |
|  | for x in self.mine: |
|  | # print(x, self.H, self.W) |
|  | # print(self.real_state.shape) |
|  | self.real_state[x//self.W][x%self.W] = 1 |
|  |  |
|  | # state_type中0表示无雷，1-8表示有雷, 用此来表示对附近雷的计数 |
|  | self.state_type = np.zeros((H, W), dtype=np.int32) |
|  | for i in range(H): |
|  | for j in range(W): |
|  | self.state_type[i][j] = mine_count(h=i, w=j, H=H, W=W, real_state=self.real_state) |
|  |  |
|  | # obs为-100表示未翻开(未知)，0-8表示翻开但无雷，数值大小表示翻开位置周边雷的数量 |
|  | # agent的状态记录所用，也可以用来作为打印之用 |
|  | self.obs = np.zeros((H, W), dtype=np.int32) -100 |
|  |  |
|  | def act(self, i, j): |
|  | done = False |
|  | if self.obs[i][j]!=-100: |
|  | print("该位置已经被揭开过，重复翻开，error！！！") |
|  | return ValueError |
|  | if self.real_state[i][j] == 1: |
|  | # game over 触雷 |
|  | done = True |
|  | return None, done |
|  |  |
|  | self.obs[i][j] = self.state_type[i][j] |
|  | return self.obs[i][j], done |
|  |  |
|  | def pp(self): |
|  | for i in range(self.H): |
|  | for j in range(self.W): |
|  | if self.obs[i][j]>=0: |
|  | print(self.obs[i][j], end=' ') |
|  | else: |
|  | print('*', end=' ') |
|  | print() |
|  |  |
|  | def input(self): |
|  | while True: |
|  | i, j = input('请输入坐标:').split() |
|  | _, done = self.act(int(i), int(j)) |
|  | if done: |
|  | print('game over!!!') |
|  | print(self.real_state) |
|  | print(self.state_type) |
|  | break |
|  | self.pp() |
|  |  |
|  | # 测试用 |
|  | # env=Env(5, 5, 5) |
|  | # env.input() |
|  |  |
|  | def play(): |
|  | N = 99  # 雷的数量 |
|  | H = 16 |
|  | W = 30 |
|  | env = Env(H=H, W=W, N=N)  # H=36, W=64, N=100 |
|  |  |
|  | known_count_dict = {} # (2,2):3, (3,3):2 |
|  |  |
|  | known_set = set()   #  (2, 2) |
|  | unknown_set = set() |
|  | boom_set = set() |
|  | for i in range(H): |
|  | for j in range(W): |
|  | unknown_set.add((i,j)) |
|  |  |
|  | new_nodes = [] |
|  | new_relation_nodes_set = set() |
|  |  |
|  | while(len(unknown_set)>0): |
|  | probs_list = [] # ((1,1), 0.5, 3), ((2,2), 0.5, 2) # (node, prob, count) # count为node附近的unknown个数 |
|  | for node in unknown_set: |
|  | p_list = [] |
|  | n_c = 0  # node附近的unknown_node的个数 |
|  | for _node in near_by(*node, H, W): |
|  | if _node in unknown_set: |
|  | n_c += 1 |
|  | if _node in known_set: |
|  | count = known_count_dict[_node] |
|  | n = 0 |
|  | for _node_node in near_by(*_node, H, W): |
|  | if _node_node in unknown_set: |
|  | n += 1 |
|  | if _node_node in boom_set: |
|  | count -= 1 |
|  | p_list.append(count/n) # 有雷的概率 |
|  | p_list.append(N/len(unknown_set)) |
|  | probs_list.append((node, max(p_list), n_c)) |
|  | m_p = min(probs_list, key=lambda x:x[1])[1] |
|  | probs_list = [x for x in probs_list if x[1]==m_p] |
|  | node = min(probs_list, key=lambda x:x[2])[0] |
|  |  |
|  | count, done = env.act(*node) |
|  | if done == True: |
|  | print('游戏失败，触雷，game over!!!') |
|  | print(node) |
|  | raise Exception |
|  | print("成功完成一步！！！ \n\n") |
|  | print("remove node:", node) |
|  | unknown_set.remove(node) |
|  | known_set.add(node) |
|  | known_count_dict[node] = count |
|  |  |
|  | env.pp()  # 打印当前游戏环境的显示 |
|  |  |
|  | new_nodes.append(node) |
|  | new_relation_nodes_set.add(node) |
|  |  |
|  |  |
|  | while new_nodes or new_relation_nodes_set: |
|  | # debug |
|  | # print(new_nodes) |
|  | # print(new_relation_nodes_set) |
|  | while new_nodes: |
|  | node = new_nodes.pop() |
|  | k = 0 |
|  | b = 0 |
|  | count = known_count_dict[node] |
|  | tmp_unk = set() |
|  | for _node in near_by(*node, H, W): |
|  | if _node in known_set: |
|  | new_relation_nodes_set.add(_node) |
|  | # k += 1 |
|  | continue |
|  | if _node in boom_set: |
|  | new_relation_nodes_set.add(_node) |
|  | b += 1 |
|  | continue |
|  | tmp_unk.add(_node) # 对unknown节点进行判断 |
|  | count -= b |
|  | if count==len(tmp_unk): |
|  | # 全是雷 |
|  | for _node in tmp_unk: |
|  | print("remove node:", _node) |
|  | unknown_set.remove(_node) |
|  | boom_set.add(_node) |
|  | new_relation_nodes_set.add(_node) |
|  | N -= 1 |
|  | if count==0 and len(tmp_unk) > 0: |
|  | # 全都不是雷 |
|  | for _node in tmp_unk: |
|  | c, done = env.act(*_node) |
|  | if done: |
|  | print("程序判断出错，把雷误触发了！！！") |
|  | raise Exception |
|  |  |
|  | print("remove node:", _node) |
|  | unknown_set.remove(_node) |
|  | known_set.add(_node) |
|  | known_count_dict[_node] = c |
|  | new_nodes.append(_node) |
|  | new_relation_nodes_set.add(_node) |
|  |  |
|  | while new_relation_nodes_set: |
|  | node = new_relation_nodes_set.pop() |
|  | tmp_set = set() |
|  | for i in range(-2, 3): |
|  | for j in range(-2, 3): |
|  | if node[0]+i>=0 and node[0]+iand node[1]+j>=0 and node[1]+j |
|  | if (node[0]+i, node[1]+j) in known_set: |
|  | if known_count_dict[(node[0]+i, node[1]+j)]==0: |
|  | continue |
|  | tmp_set.add((node[0]+i, node[1]+j)) |
|  | if len(tmp_set)==0: |
|  | continue |
|  |  |
|  | relations = [] |
|  | for node in tmp_set:  # node 为 known set |
|  | tmp_tmp_set = set() |
|  | c = known_count_dict[node] |
|  | for _node in near_by(*node, H, W): |
|  | if _node in boom_set: |
|  | c -= 1 |
|  | continue |
|  | if _node in unknown_set: |
|  | tmp_tmp_set.add(_node) |
|  | continue |
|  | if len(tmp_tmp_set)==0: |
|  | continue |
|  | relations.append([tmp_tmp_set, c, node]) |
|  |  |
|  | if len(relations)<2: |
|  | continue |
|  | for i in range(0, len(relations)): |
|  | for j in range(1, len(relations)): |
|  | if relations[i][0].issuperset(relations[j][0]): |
|  | relations[i][0] -= relations[j][0] |
|  | relations[i][1] -= relations[j][1] |
|  |  |
|  | if relations[i][1]==len(relations[i][0]) and relations[i][1]>0: |
|  | # 全是雷 |
|  | for _node in relations[i][0]: |
|  | if _node in boom_set: |
|  | continue |
|  | print("remove node:", _node) |
|  | unknown_set.remove(_node) |
|  | boom_set.add(_node) |
|  | new_relation_nodes_set.add(relations[i][2]) |
|  | N -= 1 |
|  | if relations[i][1]==0 and len(relations[i][0]): |
|  | # 全都不是雷 |
|  | for _node in relations[i][0]: |
|  | if _node in known_set: |
|  | continue |
|  | c, done = env.act(*_node) |
|  | if done: |
|  | print("程序判断出错，把雷误触发了！！！") |
|  | raise Exception |
|  | print("remove node:", _node) |
|  | unknown_set.remove(_node) |
|  | known_set.add(_node) |
|  | known_count_dict[_node] = c |
|  | new_nodes.append(_node) |
|  | new_relation_nodes_set.add(_node) |
|  |  |
|  | if relations[j][0].issuperset(relations[i][0]): |
|  | relations[j][0] -= relations[i][0] |
|  | relations[j][1] -= relations[i][1] |
|  |  |
|  | if relations[j][1]==len(relations[j][0]) and relations[j][1]>0: |
|  | # 全是雷 |
|  | for _node in relations[j][0]: |
|  | if _node in boom_set: |
|  | continue |
|  | print("remove node:", _node) |
|  | unknown_set.remove(_node) |
|  | boom_set.add(_node) |
|  | new_relation_nodes_set.add(relations[j][2]) |
|  | N -= 1 |
|  | if relations[j][1]==0 and len(relations[j][0]): |
|  | # 全都不是雷 |
|  | for _node in relations[j][0]: |
|  | if _node in known_set: |
|  | continue |
|  | c, done = env.act(*_node) |
|  | if done: |
|  | print("程序判断出错，把雷误触发了！！！") |
|  | raise Exception |
|  | print("remove node:", _node) |
|  | unknown_set.remove(_node) |
|  | known_set.add(_node) |
|  | known_count_dict[_node] = c |
|  | new_nodes.append(_node) |
|  | new_relation_nodes_set.add(_node) |
|  |  |
|  |  |
|  | print('游戏胜利，game over!!!') |
|  | return True |
|  |  |
|  | sss = [] |
|  | for xyz in range(30000): |
|  | try: |
|  | sss.append(play()) |
|  | print('第 %d 次游戏成功'%xyz) |
|  | except Exception: |
|  | print('第 %d 次游戏失败！！！'%xyz) |
|  | continue |
|  | print("成功次数： ", sum(sss)) |
|  | print("成功比例： ", sum(sss)/30000) |


```

**个人github博客地址：**
[https://devilmaycry812839668\.github.io/](https://github.com "https://devilmaycry812839668.github.io/")


