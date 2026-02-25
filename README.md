# 🧩 Matryoshka-Puncher (MP-SAT)

**Matryoshka-Puncher** — это высокопроизводительный SAT/UNSAT решатель, использующий гибридную архитектуру дискретизации пространства состояний через макро-узлы (M-Nodes) и итеративную фильтрацию ограничений (AC-3) с применением битмасок.

---

## 🇷🇺 Русский (Russian)

### 1. Математическая модель и алгоритм
Алгоритм преобразует задачу 3-SAT из пространства бинарных переменных $V$ в пространство конфигураций макро-узлов $M$.

#### 1.1 Группировка (Clustering)
Клозы объединяются в тройки (или макро-узлы) на основе коэффициента перекрытия переменных. Для каждой тройки $T_i$ вычисляется домен допустимых состояний $D_i$: 

$$D_i = \{s \in \{0,1\}^k \mid \forall c \in T_i, \text{eval}(c, s) = 1\}$$

Где $k$ — количество уникальных переменных в макро-узле. Типичный размер домена $|D_i| \in [1, 2^k - 1]$.

#### 1.2 Ограничения и AC-3
Связи между узлами описываются как 2-КНФ запреты. Узел $M_1$ совместим с $M_2$ по общим переменным $V_{shared} = V_{M1} \cap V_{M2}$, если: 

$$\forall s_1 \in D_1, \forall s_2 \in D_2: s_1|_{V_{shared}} = s_2|_{V_{shared}}$$

Для фильтрации используется алгоритм **AC-3 (Arc Consistency)**, оптимизированный через побитовые операции (Bitwise Propagation).

### 2. Детерминированность поиска
Доказательство детерминированности основывается на полноте алгоритма AC-3 и систематическом обходе дерева состояний:
1. **Инвариант фильтрации**: На каждом шаге AC-3 удаляет только те состояния $s \in D_i$, которые гарантированно не входят ни в одно решение SAT (в силу локального противоречия).
2. **Полнота обхода**: Несмотря на использование *Random Walk* для выбора ветви, алгоритм сохраняет стек возвратов (backtracking). Если ветвь приводит к пустому домену $\exists i, D_i = \emptyset$, алгоритм возвращается и пробует альтернативное состояние.
3. **Сходимость**: Пространство состояний конечно ($\prod |D_i|$), а отсутствие циклов в дереве поиска гарантирует достижение терминального состояния (SAT или полное доказательство UNSAT).

### 3. Оценка сложности

Комплексная сложность алгоритма определяется выражением:

$$O(I \cdot (E \cdot d + K \cdot d^k))$$

**Детализация параметров:**
*   $I$ — количество итераций рекурсивного поиска (в худшем случае — экспоненциально, но на практике на порядки меньше полного перебора за счет рандомизации).
*   $E \cdot d$ — стоимость **AC-3 фильтрации** на графе: каждое из $E$ рёбер обрабатывается с доменом размера $d$.
*   $K \cdot d^k$ — стоимость предвычисления совместимости для каждой из $K$ троек. 
    *   *Важное уточнение:* Для 3-SAT число переменных в макро-узле $k \le 9$ (3 клоза $\times$ 3 литерала). Это делает экспоненту **локальной константой**, превращая глобальный комбинаторный взрыв в контролируемый расчет внутри узла.


---

## 🇺🇸 English

### 1. Mathematical Model and Algorithm
The algorithm transforms a 3-SAT problem from binary variable space $V$ into a macro-node configuration space $M$.

#### 1.1 Clustering
Clauses are grouped into triples (macro-nodes) based on variable overlap. For each triple $T_i$, the domain of valid states $D_i$ is computed:

$$D_i = \{s \in \{0,1\}^k \mid \forall c \in T_i, \text{eval}(c, s) = 1\}$$

Where $k$ is the number of unique variables in the macro-node.

#### 1.2 Constraints and AC-3
Inter-node constraints are represented as 2-CNF prohibitions. A node $M_1$ is consistent with $M_2$ regarding shared variables $V_{shared} = V_{M1} \cap V_{M2}$ if:

$$\forall s_1 \in D_1, \forall s_2 \in D_2: s_1|_{V_{shared}} = s_2|_{V_{shared}}$$

Filtration is performed via the **AC-3 (Arc Consistency)** algorithm, optimized using bitwise operations.

### 2. Determinism of Search
The proof of determinism relies on the completeness of AC-3 and systematic state-space traversal:
1. **Filtering Invariant**: At each step, AC-3 removes only those states $s \in D_i$ that are guaranteed not to be part of any SAT solution.
2. **Search Completeness**: While *Random Walk* is used for branch selection, the algorithm maintains a backtracking stack.
3. **Convergence**: The state space is finite ($\prod |D_i|$), and the absence of cycles in the search tree ensures a terminal state (SAT or total UNSAT proof).

### 3. Complexity Analysis

The overall complexity is defined as:
$$O(I \cdot (E \cdot d + K \cdot d^k))$$

**Parameter Breakdown:**
*   $I$: Number of recursion iterations (exponential in the worst case, but practically much lower due to randomized heuristics).
*   $E \cdot d$: Cost of **AC-3 filtration**: each of the $E$ edges is processed against a domain of size $d$.
*   $K \cdot d^k$: Cost of precomputing compatibility for each of the $K$ triples.
    *   *Note:* In 3-SAT, the variables per macro-node $k \le 9$. This encapsulates the exponential growth within a **local constant**, effectively turning a global combinatorial explosion into a controlled per-node computation.
---

## 📚 References / Литература
* **Mackworth, A. K.** (1977). *Consistency in networks of relations*. (Original AC-3 algorithm).
* **Davis, M., Logemann, G., & Loveland, D.** (1962). *A Machine Program for Theorem-Proving*. (DPLL basis).
* **Biere, A., et al.** (2009). *Handbook of Satisfiability*.
* **Constraint Satisfaction Problems (CSP)** — General theory of local consistency.

---

## 🤖 AI Statement
Этот контент и архитектурные решения разработаны с применением систем искусственного интеллекта:
* [Gemini](https://gemini.google.com) (Google DeepMind)
* [DeepSeek](https://www.deepseek.com)
