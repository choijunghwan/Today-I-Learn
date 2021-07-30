# 문제
![문제](/Img/Backjoon1967.PNG)  
<br>

# 예제
### 입력
``` Java
12
1 2 3
1 3 2
2 4 5
3 5 11
3 6 9
4 7 1
4 8 7
5 9 15
5 10 4
6 11 6
6 12 10
```
<br>

### 출력
``` java
45
```
<br>

# 문제 풀이
<br>

이 문제는 **트리의 지름**을 구하는게 핵심이다.    
지름은 간단하게 두 노드의 경로가 제일 긴 것을 의미한다.    
두 노드의 경로를 구하기 위해서는 먼저 기준이 되는 노드를 정해줘야 한다.  
<br>
우리는 트리의 루트노드를 알기 때문에 루트노드에서 가장 멀리 떨어진 노드를 기준이 되는 노드로 정해 주자.  
그 다음에 기준 노드로 부터 가장 먼 노드를 찾아주기만 하면 된다.

1. 기준 노드를 탐색
2. 기준 노드로 부터 가장 먼 노드를 탐색
<br><br>


### 자료구조
1. Tree 자료 구조를 사용한다.  
    * 트리 자료구조 장점은 데이터의 검색에 있다.  
    * 하지만 이 문제는 데이터의 검색보다는 여러 노드를 탐색하여 서로간의 거리를 구해야 해서 효율이 떨어진다.
    * 이 문제를 트리 자료 구조를 통해 문제를 푼다면 시간초과가 발생할 것이다.
2. 그래프 탐색을 사용한다.  
   * 그래프 탐색은 크게 DFS, BFS 두가지가 있는데.
   * 이 문제는 모든 경우를 다 탐색해서 비교해 봐야 하므로 DFS와 BFS는 선택의 차이일것 같다.
   * 나는 DFS를 선택했다.
3. 인접 행렬 vs 인접 리스트
   * 인접 행렬을 사용하면 이문제에서는 최대 10000X10000 의 메모리가 사용된다.
   * 즉, 많은 메모리 낭비가 발생하게 된다.
   * 그래서 나는 인접 리스트를 사용하겠다.
  
<br><br><br>

# Code
``` Java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;
import java.util.StringTokenizer;

public class Graph_1967 {
    static List<List<Tree>> list;
    static boolean[] visited;
    static int e;
    static int result = 0;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(br.readLine());
        list = new ArrayList<>();
        for (int i = 0; i <= n + 1; i++) {
            list.add(new ArrayList<>());
        }

        for (int i = 1; i < n; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine(), " ");
            int a = Integer.parseInt(st.nextToken());
            int b = Integer.parseInt(st.nextToken());
            int c = Integer.parseInt(st.nextToken());

            list.get(a).add(new Tree(b, c));
            list.get(b).add(new Tree(a, c));
        }

        visited = new boolean[n + 1];
        dfs(1, 0);

        visited = new boolean[n + 1];
        result = 0;
        dfs(e, 0);

        System.out.println(result);

    }

    private static void dfs(int start, int sum) {

        if (result < sum) {
            result = sum;
            e = start;
        }

        visited[start] = true;

        for (Tree t : list.get(start)) {
            if (!visited[t.node]) {
                dfs(t.node, sum + t.len);
            }
        }


    }
}

class Tree {
    int node;
    int len;

    public Tree(int node, int len) {
        this.node = node;
        this.len = len;
    }
}
```
   
