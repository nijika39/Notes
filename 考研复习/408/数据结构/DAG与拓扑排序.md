---
tags:
  - 408考研
  - 数据结构
  - DAG
  - 拓扑排序
aliases:
  - DAG
  - 拓扑排序
category: 数据结构
---

# DAG与拓扑排序

本章主要介绍 DAG 描述表达式、AOV 网、拓扑排序以及逆拓扑排序。

---

## 1. DAG描述表达式

**定义**：有向无环图（Directed Acyclic Graph, DAG）是指一个不含任何有向环路的有向图。

**构建表达式的 DAG 表示步骤**：
1. **确定顺序**：确定表达式中运算符的运算顺序（可画出表达式树辅助分析）。
2. **排列操作数**：将表达式中所有的操作数（常数或变量）在图的最底部**不重复地**排成一排。
3. **自底向上构建**：按照运算符的运算顺序构建 DAG。在构建每一个运算符节点时，**必须复用已经存在的公共子表达式顶点**（通过将指针/弧指向已有的相同顶点来合并），从而消除重复的项。

> 📌 **核心注意点**：在表达式的 DAG 表示中，**绝对不会**出现重复的操作数顶点。选择题常考“该表达式对应的 DAG 中，最少需要多少个顶点/多少条弧”，计算时需注意公共子表达式的完全合并。

---

## 2. 拓扑排序

用顶点表示活动、用有向边 $U \to V$ 表示活动 $U$ 必须先于活动 $V$ 进行的 DAG 称为 **AOV 网（Activity on Vertex Network）**。

### 2.1 拓扑排序算法步骤与代码

**拓扑排序方法步骤**：
1. 从 AOV 网中选择一个**没有前驱（入度为 0）**的顶点并输出。图中可能同时存在多个入度为 0 的顶点，随机选择一个输出即可（因此拓扑序列可能不唯一）。
2. 从网中**删除该顶点**和所有以它为起点的有向边。
3. 重复步骤 1 和 2，直到当前的 AOV 网为空，或网中**无法找到新的入度为 0 的顶点**为止。后一种情况表明有向图中必然存在环路（拓扑排序失败）。

#### ① 基于邻接表存储结构的拓扑排序算法代码 (Kahn 算法/广度优先思想)

```c
// 1. 王道邻接表结构体定义（考场可简写或不写）
#define MaxVertexNum 100
typedef struct ArcNode {
    int adjvex;
    struct ArcNode *nextarc;
} ArcNode;
typedef struct VNode {
    VertexType data;
    ArcNode *firstarc;
} VNode, AdjList[MaxVertexNum];
typedef struct {
    AdjList vertices;
    int vexnum, arcnum;
} ALGraph;

// 2. 核心算法函数
bool TopologicalSort(ALGraph G) {
    int indegree[MaxVertexNum] = {0};

    // 1. 统计所有顶点的入度
    for (int i = 0; i < G.vexnum; i++) {
        ArcNode *p = G.vertices[i].firstarc;
        while (p != NULL) {
            indegree[p->adjvex]++;
            p = p->nextarc;
        }
    }

    // 2. 将所有入度为 0 的顶点入栈
    int stack[MaxVertexNum];
    int top = -1;
    for (int i = 0; i < G.vexnum; i++) {
        if (indegree[i] == 0) {
            stack[++top] = i;
        }
    }

    int count = 0;          // 记录已输出的顶点数
    int topoSeq[MaxVertexNum]; // 暂存拓扑序列

    // 3. 循环推进
    while (top != -1) {
        int u = stack[top--]; // 弹出入度为 0 的顶点
        topoSeq[count++] = u;

        // 遍历 u 的所有邻接点，将其入度减 1
        ArcNode *p = G.vertices[u].firstarc;
        while (p != NULL) {
            int v = p->adjvex;
            indegree[v]--;
            if (indegree[v] == 0) {
                stack[++top] = v; // 若入度降为 0，则入栈
            }
            p = p->nextarc;
        }
    }

    // 4. 判断是否存在回路
    if (count < G.vexnum) {
        printf("图中存在环，拓扑排序失败！\n");
        return false;
    }

    // 输出拓扑序列
    printf("拓扑序列为: ");
    for (int i = 0; i < count; i++) {
        printf("%d ", topoSeq[i]);
    }
    printf("\n");
    return true;
}
```

#### ② 基于邻接表和 DFS 的拓扑排序算法代码

```c
// 核心状态数组与辅助栈
#define WHITE 0  // 未访问
#define GRAY  1  // 正在递归栈中（访问中）
#define BLACK 2  // 访问结束

int visited[MaxVertexNum];
int topoStack[MaxVertexNum]; // 利用栈来逆序记录输出
int topoTop = -1;

// DFS 辅助递归函数
bool DFSTopo(ALGraph G, int u) {
    visited[u] = GRAY; // 标记为灰色

    ArcNode *p = G.vertices[u].firstarc;
    while (p != NULL) {
        int v = p->adjvex;
        if (visited[v] == GRAY) {
            return false; // 发现后向边（环），拓扑排序失败
        }
        if (visited[v] == WHITE) {
            if (!DFSTopo(G, v)) {
                return false;
            }
        }
        p = p->nextarc;
    }

    visited[u] = BLACK; // 标记为黑色（安全退栈）
    topoStack[++topoTop] = u; // 核心：在顶点访问“结束”时入栈
    return true;
}

// 基于 DFS 的拓扑排序主函数
bool DFSTopologicalSort(ALGraph G) {
    for (int i = 0; i < G.vexnum; i++) {
        visited[i] = WHITE;
    }
    topoTop = -1;

    for (int i = 0; i < G.vexnum; i++) {
        if (visited[i] == WHITE) {
            if (!DFSTopo(G, i)) {
                printf("图中存在环，无拓扑序列！\n");
                return false;
            }
        }
    }

    // 逆序输出栈中元素即为正向拓扑序列
    printf("DFS 拓扑序列为: ");
    for (int i = topoTop; i >= 0; i--) {
        printf("%d ", topoStack[i]);
    }
    printf("\n");
    return true;
}
```

---

### 2.2 逆拓扑排序

对一个 AOV 网，若采用以下步骤进行排序，则称为**逆拓扑排序**：
1. 从 AOV 网中选择一个**没有后继（出度为 0）**的顶点并输出。
2. 从网中**删除该顶点**和所有以它为终点的有向边。
3. 重复步骤 1 和 2，直到当前的 AOV 网变成空为止。

> 💡 **DFS 实现逆拓扑排序**：若直接正序输出 DFS 访问结束（即变为黑色）时的顶点序列，其结果恰好就是该图的一个**逆拓扑序列**。

---

### 2.3 易错点与核心考点

1. **结果不唯一性**：拓扑排序的结果可能不唯一。拓扑排序唯一的充要条件是：**每次输出顶点时，当前入度为 0 的顶点都恰好只有一个**。
2. **邻接矩阵的形式**：可根据拓扑排序的结果对顶点重新编号，使得重新编号后的**邻接矩阵成为上（或下）三角矩阵**。
   * 对于一般的有向图而言，若其邻接矩阵可通过顶点重排转换为三角矩阵，则该图一定是 DAG，因而存在拓扑序列。
   * **特例**：若邻接矩阵本身是三角矩阵，则该图一定存在拓扑序列；但存在拓扑序列的图，其原始邻接矩阵不一定是三角矩阵。
3. **图的确定性（非常重要）**：
   * DAG 的拓扑序列唯一，**不能**够唯一确定该图。
   * *反例*：图 1 为 $A \to B, B \to C$；图 2 为 $A \to B, B \to C, A \to C$。这两个图的拓扑序列都是 $A, B, C$，但图的结构不同。
4. **序列数量关系**：DAG 的拓扑序列数量和逆拓扑序列数量**必然相等**（因为将任一合法的拓扑序列完全反转，必得到一个合法的逆拓扑序列）。
   * 延伸考点：若有向图的拓扑序列唯一（数量为 1），则逆拓扑序列也唯一，此时图中**入度为 0 的顶点**和**出度为 0 的顶点**都仅有 1 个。
5. **环路判定（非常重要）**：
   * **DFS** 和 **拓扑排序** 可以判断出一个有向图是否有环。
   * **常规的 BFS 无法判断**有向图是否有环，但是**基于 BFS 思想的拓扑排序（Kahn 算法）**可以判断。
6. **暂存容器的选择**：在拓扑排序中，暂存入度为 0 的顶点时，**可以使用栈，也可以使用队列，甚至可以使用普通的线性表**，它们对最终算法的正确性没有影响，只是会改变输出的拓扑序列顺序。