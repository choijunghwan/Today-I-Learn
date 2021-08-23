# 문제
![문제](/Img/Baekjoon2186.PNG)  
<br>

# 문제풀이
이번 문제는 문제 풀이방법보다 막혔던 부분 위주로 설명해보겠습니다.

먼저 저는 BFS를 이용해 주어진 단어 BREAK 에 맞춰 탐색해나갔습니다.

BFS 코드를 바로 보겠습니다.

```Java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.LinkedList;
import java.util.Queue;
import java.util.StringTokenizer;

public class Main {

    static int N;
    static int M;
    static int K;
    static String[][] map;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine(), " ");
        N = Integer.parseInt(st.nextToken());
        M = Integer.parseInt(st.nextToken());
        K = Integer.parseInt(st.nextToken());

        map = new String[N][M];
        for (int i = 0; i < N; i++) {
            map[i] = br.readLine().split("");
        }
        String word = br.readLine();

        int result = bfs(word);

        System.out.println(result);
    }

    private static int bfs(String word) {
        Queue<Point> q = new LinkedList<>();

        String s = Character.toString(word.charAt(0));
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (map[i][j].equals(s)) {
                    q.add(new Point(i, j, 0));
                }
            }
        }

        int[] dx = {0, 1, 0, -1};
        int[] dy = {1, 0, -1, 0};
        int count = 0;

        while (!q.isEmpty()) {
            Point p = q.poll();

            if (p.depth + 1 == word.length()) {
                count++;
                continue;
            }

            String nextCh = Character.toString(word.charAt(p.depth + 1));
            for (int i = 0; i < 4; i++) {
                for (int j = 1; j <= K; j++) {
                    int nextX = p.x + dx[i] * j;
                    int nextY = p.y + dy[i] * j;

                    if (nextX < 0 || nextX >= M || nextY < 0 || nextY >= N) {
                        continue;
                    }

                    if (map[nextX][nextY].equals(nextCh)) {
                        q.add(new Point(nextX, nextY, p.depth + 1));
                    }
                }
            }
        }

        return count;
    }

    static class Point {
        int x;
        int y;
        int depth;

        public Point(int x, int y, int depth) {
            this.x = x;
            this.y = y;
            this.depth = depth;
        }
    }
}

```

BFS를 이용해 문제를 풀었을때 **메모리 초과**라는 결과가 발생하였습니다. 

![에러1](/Img/Baekjoon2186Error1.PNG)

<br>

처음에는 무엇인가 불필요한 반복이 일어나고 있는건가??  
라는 생각이 들어 방문한곳을 체크해주자는 생각에 boolean 배열을 이용해 방문한곳을 체크해주고 DFS 로 다시 풀어보았습니다.

DFS 풀이
```JAVA
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

    static int N;
    static int M;
    static int K;
    static String[][] map;
    static String word;
    static int result;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine(), " ");
        N = Integer.parseInt(st.nextToken());
        M = Integer.parseInt(st.nextToken());
        K = Integer.parseInt(st.nextToken());

        map = new String[N][M];

        for (int i = 0; i < N; i++) {
            map[i] = br.readLine().split("");
        }
        word = br.readLine();

        String s = Character.toString(word.charAt(0));
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (map[i][j].equals(s)) {
                    dfs(i, j, 0);
                }
            }
        }


        System.out.println(result);
    }

    private static void dfs(int x, int y, int depth) {

        int[] dx = {0, 1, 0, -1};
        int[] dy = {1, 0, -1, 0};
        int count = 0;

        if (depth + 1 == word.length()) {
            result++;
            return ;
        }

        String nextCh = Character.toString(word.charAt(depth + 1));
        for (int i = 0; i < 4; i++) {
            for (int j = 1; j <= K; j++) {
                int nextX = x + dx[i] * j;
                int nextY = y + dy[i] * j;

                if (nextX < 0 || nextX >= M || nextY < 0 || nextY >= N) {
                    continue;
                }


                if (map[nextX][nextY].equals(nextCh)) {
                    dfs(nextX, nextY, depth + 1);
                }
            }
        }

    }

}
```

그 결과는
![에러2](/Img/Baekjoon2186Error2.PNG)

똑같은 메모리 부족 에러였습니다.

계속 같은 에러가 나오는게 저의 문제 접근 방법이 잘못되었다고 생각이 들어졌습니다. 그래서 조금 찾아보았더니 메모리 부족, 시간 초과 문제를 해결 할 수 있는 방법중 하나인 **메모이제이션**을 발견하였습니다. 

흔히 동적 프로그래밍이라고도 알려져 있는 방법입니다.

dp[][][] 배열을 생성해 계산해놓은 값을 저장해두고, 나중에 반복적인 계산은 생략하고 값만 가져다 쓰는 방법으로, 로컬 캐시라고 생각해도 무방하다.

코드를 보고 설명을 이어서 하겠습니다.

```JAVA
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.StringTokenizer;

public class Main {

    static int N;
    static int M;
    static int K;
    static char[][] map;
    static char[] word;
    static int[][][] dp;
    static int[] dx = {0, 1, 0, -1};
    static int[] dy = {1, 0, -1, 0};

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine(), " ");
        N = Integer.parseInt(st.nextToken());
        M = Integer.parseInt(st.nextToken());
        K = Integer.parseInt(st.nextToken());

        map = new char[N][M];

        for (int i = 0; i < N; i++) {
            map[i] = br.readLine().toCharArray();
        }
        word = br.readLine().toCharArray();
        dp = new int[N][M][word.length];

        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                Arrays.fill(dp[i][j], -1);
            }
        }

        int ans = 0;
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (map[i][j] == word[0]) {
                    ans += dfs(i, j, 0);
                }
            }
        }


        System.out.println(ans);
    }

    private static int dfs(int x, int y, int depth) {

        if (depth + 1 == word.length) {
            return dp[x][y][depth] = 1;
        }
        if (dp[x][y][depth] != -1) {
            return dp[x][y][depth];
        }

        dp[x][y][depth] = 0;

        for (int i = 0; i < 4; i++) {
            for (int j = 1; j <= K; j++) {
                int nextX = x + dx[i] * j;
                int nextY = y + dy[i] * j;

                if (nextX < 0 || nextX >= N || nextY < 0 || nextY >= M) {
                    continue;
                }

                if (map[nextX][nextY] == word[depth+1]) {
                    dp[x][y][depth] += dfs(nextX, nextY, depth + 1);
                }
            }
        }

        return dp[x][y][depth];
    }

}
```

dp 3차원 배열을 만들어 조건에 만족했을때 값을 저장해둘 수 있고.


```JAVA
if (dp[x][y][depth] != -1) {
    return dp[x][y][depth];
}
```
이 부분을 통해 내가 이미 탐색한 경우인경우에는 결과값만 가져다 쓸 수 있어 불필요한 반복을 줄일 수 있다.

마지막으로 
```JAVA
static int[] dx = {0, 1, 0, -1};
static int[] dy = {1, 0, -1, 0};
```
원래는 dx,dy 코드를 dfs 함수 안에 넣어두었다.

어차피 dfs 함수에서만 사용하는 지역 변수인데 불필요하게 전역 변수로 선언을 해 놓을 필요가 있을까 생각이 들었다.

근데 만약 지역 변수로 선언을 해 놓는다면 dfs 특성상 재귀함수처럼 함수를 여러번 호출하게 되는데 함수를 호출할때마다 dx,dy 변수를 생성해 메모리를 할당해 주게되서, 오히려 전역변수로 선언해 놓는것보다 훨씬 비효율적인 방법이 되었다.

dfs가 아닌 bfs를 사용할때는 지역변수로 해놓는것이 좋을것 같지만, dfs처럼 재귀적인 호출이 많을때는 오히려 전역변수가 더 나은 방법인것 같다.

<br>

# 마무리
문제를 풀때 효율성 or 시간 초과 등의 문제가 발생했을때는
1. 나의 문제 접근 방법에 문제가 있나??
2. 메모이제이션(동적 프로그래밍)을 통해 효율성을 늘릴 수 있는가
   
두가지를 고려해서 생각해보면 좋을거 같다.
