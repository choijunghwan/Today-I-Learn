#  문제
![문제](/Img/Backjoon1963.PNG)

<br>

# 문제 풀이
<br>

이 문제는 생각해야할 문제 부분이 여러가지가 있다.  
차근차근 해결해야 할 핵심 문제를 정리해보자.  

<br>

## 요구사항 정의
1. 소수인지 아닌지 판별
2. 변환에 필요한 최소 횟수 구하기

크게 두가지로 나눠 볼 수 있는데 여기서 1번 문제는 소수 구하는 방법중에 하나인 **에라토스테네스의 체** 방법론을 이용해
배열에 소수인지 참/거짓 여부를 저장해 두었다.

문제에서 수 범위가 4자리만 해당하기 때문에 배열의 크기가 10000을 넘지 않아 크기를 10000으로 선언해 주었다.

배열 크기 10000은 int 타입으로 선언했다 가정했을때 4 바이트 * 10000 = 40000 바이트이고 1KB = 1024B 이므로 40000B = 39.xxKB 이다. 즉 메모리가 생각보다 많이 들지 않으니 신경쓰지 않아도 된다.

<br>

두번째 문제는 해결하기 위해선 우리는 어떤 규칙으로 수를 변환시켜서 값을 찾을 것인지를 먼저 정해야 하는데. 여기서 규칙이란 4자리의 숫자중에 천의자리 숫자를 바꿀건지, 백의자리 숫자를 바꿀건지 에 대한 규칙을 말한다.

허나 이 문제는 효율적인 규칙을 찾기가 매우 힘들고, 오히려 모든 경우를 다 비교해보는 완전 탐색을 이용하는 것이 제일 효율적이다.

그리고 최소의 횟수를 구해야 하니 BFS 알고리즘에 착안해 각 경우에서 변환할 수 있는 모든 수를 큐 자료구조에 담아 모든 경우를 체크하며 넘어가기로 하자.

# Code
```Java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.LinkedList;
import java.util.Queue;
import java.util.StringTokenizer;

public class Main {
    static boolean[] primeNum = new boolean[10000];
    static int[] check;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int T = Integer.parseInt(br.readLine());

        for (int i = 2; i < primeNum.length / 2; i++) {
            if (!primeNum[i]) {
                int temp = i * i;
                while (temp < primeNum.length) {
                    primeNum[temp] = true;
                    temp += i;
                }
            }
        }

        for (int i = 0; i < T; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine(), " ");
            int A = Integer.parseInt(st.nextToken());
            int B = Integer.parseInt(st.nextToken());

            if (A == B) {
                System.out.println("0");
            } else {
                int result = bfs(A, B);
                System.out.println(result == 0 ? "Impossible" : result - 1);
            }

        }

    }

    private static int bfs(int A, int B) {
        check = new int[10000];
        Queue<Integer> queue = new LinkedList<>();
        queue.add(A);
        check[A] = 1;

        while (!queue.isEmpty()) {
            int poll = queue.poll();

            if (poll == B) {
                return check[poll];
            }

            int temp = poll;
            for (int i = 0; i < 4; i++) {
                int digit = temp % 10;
                temp /= 10;

                int index = (int) Math.pow(10, i);
                int num = poll - (digit * index);
                for (int j = 0; j < 10; j++) {
                    if (num >= 1000 && num < 10000 && check[num] == 0 && !primeNum[num]) {
                        check[num] = check[poll] + 1;
                        queue.add(num);
                    }
                    num += index;
                }

            }
        }

        return check[B];
    }
}

```

