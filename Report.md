# Report

| 实验内容   | 学号     | 姓名   |
| ---------- | -------- | ------ |
| 无信息搜索 | 16337339 | 朱祎康 |

---

## 算法原理

### Breadth First Search

**BFS** 算法的基本思想很简单：一层一层的拓展搜索树。

先拓展$dep = 1$的节点，然后拓展$dep=2$的，依次递推。

由于是按照层次顺序进行拓展，所以最先找到的可行解一定是深度最小的 $\text{First ans} = \text{min } dep$ 。



算法实现上，为了保证按照层次进行拓展，维护一个**FIFO**队列$Q$。

每次从$Q$中取出头部节点，将所有拓展节点加入$Q$的尾部。

由于$Q$的**FIFO**性质，保证了节点按照层次访问。

```pseudocode
function BFS(st, ed)
	Q <- empty queue
	push(Q, st)
	while not empty(Q)
		x <- front(Q)
		pop(Q)
		for each y which x can arrive
			push(Q, y)
		end
	end
end
```



```flow
st=>start
e=>end
op1=>operation: Initialize Q
op2=>operation: Push start into Q
sub1=>subroutine: x <- the front of Q
sub2=>subroutine: Push all y which x can arrive into Q
cond=>condition: Q is empty ?

st->op1->op2->cond
cond(yes)->e
cond(no)->sub1->sub2(right)->cond  
```

> **环检测**：
>
> 记录下所有加入过$Q$的节点，在拓展的时候，如果该节点曾经加入过$Q$，不再加入。



### Depth First Search

同**BFS** 相反，**DFS** 按照深度优先的顺序进行拓展。

在搜索树中沿着一条路走到底，当不能继续走下去的时候回溯，再去访问其他节点。

实现上，使用递归来书写即可，本质上是利用**系统栈**，根据栈的**LIFO**性质，从而实现回溯功能。

> 路径检测：
>
> 记录下搜索树跟节点到该节点的路径上的所有节点，再拓展的时候，这些节点不再拓展。

如果不加入**路径检测**的话，**DFS** 不具有*完备性*、*最优性* ，因为可能会在一些节点之间无限循环下去。

加入**路径检测**之后，**DFS**具备*完备性*，但是仍然不具备*最优性*。

```pseudocode
function DFS(x, S)
	push(S, x)
	for all y which x can arrive
		if y not in S
			DFS(y, S)
		end
	end
end
```



### Iteration Depth First Search

由于**DFS**不具有*最优性*，所以我们需要对其进行改进。

**IDF**的基本思想很简单：在**DFS**的基础上进行深度限制$dep\leq \text{limit}$。如果没有搜索到可行解，那么就将$\text{limit} \leftarrow \text{limit} + 1$ 。直到找到可行解为止，显然这个解是深度最小的解。所以具备*最优性*。

```pseudocode
function IDF()
	for dep <- 0..
		result <- DFS(st, dep)
		if result
			return dep
		end
	end
end
```

```flow
st=>start
ed=>end
op1=>operation: dep <- 0
op2=>operation: DFS with limit dep
cond=>condition: Find the answer ?
op3=>operation: dep <- dep + 1

st->op1->op2->cond(yes)->ed
cond(no)->op3(right)->op2
```



### Bidirectional Search

双向搜索是从另外一个层面对已有的搜索算法进行的优化。

双向搜索的基本思想很简单：同时从起点$S$和终点$T$进行搜索，如果两边相遇，那么就找到一组可行解。

#### 双向广度优先搜索

改进**BFS**为双向搜索是比较自然的一件事情，因为**BFS**本身就是一层一层拓展的，那么如果两边同时开始**BFS**，他们的边界相遇时，一定是“最短路径”，所以具有最优性。

在具体实现上，如何能保证两边同时开始拓展？

其实不需要使用两个**FIFO** 队列，只需要维护一个**FIFO**队列即可，队列中的节点打上$label$用来标记是从起点出发还是从终点出发。

算法终止的条件：维护两个集合表示两个搜索已经搜索的空间，当这两个集合有交集时，算法终止。

```pseudocode
function bidirectional_BFS()
	Q <- empty queue
	S[0] <- empty set
	S[1] <- empty set
		
	push(Q, (st, 0))
	push(Q, (ed, 1))
	push(S[0], st)
	push(S[1], st)
		
	while not empty Q
    	(x, tag) <- front(Q)
       	pop(Q)
       	for all y which x can arrive
        	if y in S[tag ^ 1]
        		break
        	end
        	push(Q, (y, tag))
        	push(S[tag], y)
       	end
   	end
end
```



#### 双向迭代加深搜素

对**DFS**直接进行双向搜索改进是不对的，因为**DFS**本身不具有*最优性*，所以我们选择对**IDF**进行改进。

**IDF**的思想和**BFS**其实没有本质区别，都是一层一层的进行拓展。

当$\text{limit} = D$，我们记录$S_1 = \lbrace x: dis(x, S) = D\rbrace, S_2 = \lbrace x: dis(x, T) = D\rbrace$ ，

当$S_1\cap S_2 \neq \emptyset$ ，找到“最短路径”。

```pseudocode
function bidirectional_IDF()
	S1 <- empty set
	S2 <- empty set
	for dep <- 0..
		S1 <- IDF(st, dep)
		if S1 & S2
			return 2 * dep - 1
		end
		S2 <- IDF(ed, dep)
		if S1 & S2
			return 2 * dep
		end
	end
end
```

```flow
st=>start
ed=>end
op1=>operation: Initialize S1 S2 with empty set
op2=>operation: Initialize dep with 0
op3=>operation: S1 <- IDF(st, dep)
op4=>operation: S2 <- IDF(ed, dep)
cond1=>condition: S1 join with S2 is empty ?
cond2=>condition: S1 join with S2 is empty ?
op5=>operation: dep <- dep + 1

st->op1->op2->op3->cond1(yes)->op4->cond2(yes)->op5->op3
cond1(no)->ed
cond2(no)->ed
```



## 关键代码

1. Breadth First Search

   ```python
   def BFS(maze, st, circleCheck = True):
       global n, m
       n, m = len(maze), len(maze[0])
       tot = 0
       Q = deque()
       S = set()
       Q.append((st, 0))
       S.add(st)
       while len(Q) > 0:
           x, dis = Q.popleft()
           tot += 1
           print(tot, x)
           if maze[x[0]][x[1]] == 'E':
               print("Find the END at {} with {} steps".format(x, dis))
               break
           for d in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
               y = (x[0] + d[0], x[1] + d[1])
               if canVis(maze, y):
                   if circleCheck:
                       if y not in S:
                           Q.append((y, dis + 1))
                           S.add(y)
                   else:
                       Q.append((y, dis + 1))
   
   ```

