---
tags:
  - 408考研
  - 数据结构
  - AOE网
  - 关键路径
aliases:
  - AOE网
  - 关键路径
category: 数据结构
---

# AOE网与关键路径

本章主要介绍 AOE 网的相关知识、核心算法步骤和手算算例。

---

## 一、 AOE网的基本概念与性质

### 1. 定义
* **AOE网（Activity on Edge Network）**：用**顶点**表示**事件**，**有向边**表示**活动**，并以边上的权值表示完成该活动所需开销（持续时间）的 DAG（有向无环图）。
* **关键路径**：所有从源点到汇点的路径中，**路径长度最大**（即边权值之和最大）的路径。
* **关键活动**：关键路径上的活动（即没有时间余量的活动）。

> ⚠️ **408 核心易错点（重点记忆）**：
> **若一个活动（有向边）的前驱和后继顶点（事件）都是关键事件，该活动不一定是关键活动！**
> 因为两事件之间的活动持续时间可能小于两事件发生时间的差值（存在松弛余量）。选择题经常以此设陷。

### 2. AOE网与AOV网的异同点
| 特征 | AOV网（Activity on Vertex） | AOE网（Activity on Edge） |
| :--- | :--- | :--- |
| **相同点** | 均属于有向无环图（DAG） | 均属于有向无环图（DAG） |
| **顶点含义**| 表示**活动** | 表示**事件**（活动的开始或结束） |
| **边的含义**| 表示活动之间的**先后制约关系** | 表示**活动**本身 |
| **边权值** | **无**权值 | **有**权值（表示活动的持续时间） |

### 3. 重要性质
1. **源点与汇点**：通常情况下，一个标准的 AOE 网只有一个入度为 0 的顶点（源点，代表工程开始）和一个出度为 0 的顶点（汇点，代表工程结束）。
2. **工期缩短限制**：加快关键路径上的关键活动有可能缩短总工期。但**不能任意缩短**某一个关键活动，因为一旦缩短过多，可能会导致原本非关键的路径变成新的关键路径（发生**关键路径转移**）。
3. **多条关键路径**：AOE 网的关键路径可能不止一条。若存在多条关键路径，只有加快**同时出现在所有关键路径上的关键活动**，才能有效缩短总工期。
4. **时间复杂度**：
   * 采用邻接矩阵存储时，复杂度为 $O(\lvert V \rvert^2)$。
   * 采用邻接表存储时，复杂度为 $O(\lvert V \rvert + \lvert E \rvert)$。

---

## 二、 AOE网算法步骤与核心公式

求解关键路径需要依次计算出以下 5 个核心物理量：

### 1. 核心数学公式

#### ① 事件的最早发生时间 $v_e(k)$
* **含义**：从源点出发到顶点 $v_k$ 的最长路径长度。
* **计算规则**（按**拓扑排序**顺序正向计算）：
  * $v_e(\text{源点}) = 0$
  * $v_e(k) = \max \{ v_e(j) + \text{weight}(v_j, v_k) \}$，其中 $v_j$ 是 $v_k$ 的所有直接前驱。

#### ② 事件的最迟发生时间 $v_l(k)$
* **含义**：在不影响整个工程工期（即保证汇点按时发生）的前提下，事件 $v_k$ 必须发生的最迟时间。
* **计算规则**（按**逆拓扑排序**顺序反向计算）：
  * $v_l(\text{汇点}) = v_e(\text{汇点})$
  * $v_l(k) = \min \{ v_l(j) - \text{weight}(v_k, v_j) \}$，其中 $v_j$ 是 $v_k$ 的所有直接后继。

#### ③ 活动的最早开始时间 $e(i)$
* **含义**：活动 $a_i$ 对应弧 $\langle u, v \rangle$，该活动最早可以开始的时间。
* **计算规则**：等于弧的起点（事件）的最早发生时间。
  * $e(i) = v_e(u)$

#### ④ 活动的最迟开始时间 $l(i)$
* **含义**：在不耽误工期前提下，活动 $a_i$ 对应弧 $\langle u, v \rangle$（权值为 $w$）最迟必须开始的时间。
* **计算规则**：等于弧的终点（事件）最迟发生时间减去该活动的持续时间。
  * $l(i) = v_l(v) - w$

#### ⑤ 活动的最迟与最早开始时间差额 $d(i)$
* **定义**：活动的时间余量。
  * $d(i) = l(i) - e(i)$
* 当 **$d(i) == 0$** 时，该活动 $a_i$ 即为**关键活动**。

---

### 2. 考场手写 C 语言代码实现

```c
// 求解关键路径核心算法
bool CriticalPath(ALGraph G) {
    int ve[MaxVertexNum], vl[MaxVertexNum];
    int indegree[MaxVertexNum] = {0};

    // 1. 统计每个顶点的入度
    for (int i = 0; i < G.vexnum; i++) {
        ArcNode *p = G.vertices[i].firstarc;
        while (p != NULL) {
            indegree[p->adjvex]++;
            p = p->nextarc;
        }
    }

    // 2. 拓扑排序计算 ve 数组
    int stack[MaxVertexNum], top = -1;
    int topoStack[MaxVertexNum], topoTop = -1; // 暂存拓扑序列，用于后序逆拓扑计算

    for (int i = 0; i < G.vexnum; i++) {
        if (indegree[i] == 0) {
            stack[++top] = i;
        }
        ve[i] = 0; // 初始化所有顶点的最早发生时间为 0
    }

    int count = 0;
    while (top != -1) {
        int u = stack[top--];
        topoStack[++topoTop] = u; // 存入拓扑栈
        count++;

        ArcNode *p = G.vertices[u].firstarc;
        while (p != NULL) {
            int v = p->adjvex;
            indegree[v]--;
            if (indegree[v] == 0) {
                stack[++top] = v;
            }
            // 更新当前邻接点最早发生时间（取最大值：即最长路径）
            if (ve[u] + p->info > ve[v]) {
                ve[v] = ve[u] + p->info; // p->info 存储的是网的边权值
            }
            p = p->nextarc;
        }
    }

    if (count < G.vexnum) return false; // 图中存在环路，拓扑排序失败，无法求解

    // 3. 逆拓扑排序计算 vl 数组
    int lastVex = topoStack[topoTop]; // 汇点
    for (int i = 0; i < G.vexnum; i++) {
        vl[i] = ve[lastVex]; // 初始化所有最迟发生时间为汇点的最早发生时间
    }

    while (topoTop != -1) {
        int u = topoStack[topoTop--]; // 逆拓扑退栈
        ArcNode *p = G.vertices[u].firstarc;
        while (p != NULL) {
            int v = p->adjvex;
            // 更新当前顶点最迟发生时间（取最小值：最紧迫路径）
            if (vl[v] - p->info < vl[u]) {
                vl[u] = vl[v] - p->info;
            }
            p = p->nextarc;
        }
    }

    // 4. 计算并输出所有的关键活动
    for (int u = 0; u < G.vexnum; u++) {
        ArcNode *p = G.vertices[u].firstarc;
        while (p != NULL) {
            int v = p->adjvex;
            int e = ve[u];           // 活动最早开始时间
            int l = vl[v] - p->info; // 活动最迟开始时间
            if (e == l) {            // 差额 d(i) == 0，说明是关键活动
                printf("<v%d, v%d> ", u, v);
            }
            p = p->nextarc;
        }
    }
    return true;
}
```

---

## 三、 AOE网手算算例

### 1. 实例图示

以下是用于手算练习的 AOE 网实例：

![[attachments/aoe_example.png]]

### 2. 手算技巧与极速提分套路

在 408 选择题的时间极其紧张，如果只要求找出部分数据，可以通过以下技巧进行**降维打击**：

1. **秒杀工期长度**：若题目**只要求**计算关键路径的长度（总工期），只需按照拓扑序正向推算并填满 $v_e(k)$ 这一行即可。**汇点的 $v_e$ 值就是最短总工期**（即最长路径长度），不需要算后面的 $v_l$。
2. **快速锁定关键路径**：若题目要求写出关键路径，可以同时列出 $v_e(k)$ 和 $v_l(k)$ 两行，**挑选出 $v_e == v_l$ 的顶点**（这些是关键事件），然后回到原图中将这些关键顶点连起来，**核对边权值**即可得出正确的关键路径。
3. **连线核对法则**：若有多个顶点的 $v_e == v_l$，连接它们时一定要检查边 $a$ 能否满足：$v_e(\text{起点}) + \text{weight}(a) == v_l(\text{终点})$。若不满足，则该边不是关键活动，需要排除。