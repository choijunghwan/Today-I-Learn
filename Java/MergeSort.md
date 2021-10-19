# Merge Sort(합병 정렬)
합병 정렬은 분할 정복(Divide and conquer) 방식으로 설계된 알고리즘이다. 분할 정복은 큰 문제를 반으로 쪼개 문제를 해결해 나가는 방식으로, 분할은 배열의 크기가 1보다 작거나 같을 때까지 반복한다.

입력으로 하나의 배열을 받고, 연산 중에 두개의 배열로 계속 쪼개 나간 뒤, 합치면서 정렬해 최후에는 하나의 정렬을 출력한다.

합병은 두 개의 배열을 비교하여, 기준에 맞는 값을 다른 배열에 저장해 나간다.
오름차순의 경우 배열 A, 배열 B를 비교하여 A에 있는 값이 더작다면 새 배열에 저장해주고, A 인덱스를 증가시킨 후 A,B의 반복을 진행한다. 이는 A나 B중 하나가 모든 배열값들을 새 배열에 저장할 때 까지 반복하며, 전부 다 저장하지 못한 배열의 값들은 모두 새 배열의 값에 저장해준다.

<br><br>

# 로직
**분할 과정**
1. 현재 배열을 반으로 쪼갠다. 배열의 시작 위치와, 종료 위치를 입력받아 둘을 더한 후 2를 나눠 그 위치를 기준으로 나눈다.
2. 쪼갠 배열의 크기가 0이거나 1일 때까지 반복한다.

**합병 과정**
1. 두배열 A,B의 크기를 비교한다. 각각의 배열의 현재 인덱스를 i,j로 가정하자.
2. i에는 A배열의 시작 인덱스를 저장하고, j에는 B배열의 시작 주소를 저장한다.
3. A[i]와 B[j]를 비교한다. 오름차순의 경우 이중에 작은 값을 새 배열 C에 저장한다.
4. 이를 i나 j둘중 하나가 각자배열의 끝에 도달할 때 까지 반복한다.
5. 끝까지 저장을 못한 배열의 값을, 순서대로 전부 다 C에 저장한다.
6. C 배열을 원래의 배열에 저장해준다.

합병과정은 두 배열 A,B를 정렬하기 때문에 A배열의 크기를 N1, B배열의 크기를 N2라고 할 경우 `O(N1+N2)`와 같다. N = N1 + N2 이므로 `O(N)` 이라고 할 수 있다.

분할 과정은 `logN` 만큼 일어나고 각 분할별로 합병을 진행하므로, 합병정렬의 시간 복잡도는 `O(NlogN)`이다.

좀 더 이해하기 쉽게 이미지로 보자면 다음과 같다.

![image1](/Img/MergeSort.png)

<br><br>

# 구현

```Java
public class Merge_Sort {

    private static int[] sorted;  // 합치는 과정에서 정렬하여 원소를 담을 임시배열

    public static void merge_sort(int[] a) {
        sorted = new int[a.length];
        merge_sort(a, 0, a.length - 1);
        sorted = null;
    }

    // Top-Down 방식 구현
    private static void merge_sort(int[] a, int left, int right) {

        /**
         * left==right 즉, 부분리스트가 1개의 원소만 갖고있는경우
         * 더이상 쪼갤 수 없으므로 return 한다.
         */
        if (left == right) {
            return;
        }

        int mid = (left + right) / 2;

        //분할작업
        merge_sort(a, left, mid);
        merge_sort(a, mid + 1, right);

        //합병과정
        merge(a, left, mid, right);
    }

    private static void merge(int[] a, int left, int mid, int right) {
        int i = left;  // 왼쪽 부분리스트 시작점
        int j = mid + 1;  // 오른쪽 부분리스트의 시작점
        int idx = left;  // 채워넣을 배열의 인덱스

        while (i <= mid && j <= right) {

            if (a[i] <= a[j]) {
                sorted[idx] = a[i];
                i++;
                idx++;
            }
            else {
                sorted[idx] = a[j];
                j++;
                idx++;
            }
        }

        if (i > mid) {
            while (j <= right) {
                sorted[idx] = a[j];
                j++;
                idx++;
            }
        }
        else {
            while (i <= mid) {
                sorted[idx] = a[i];
                i++;
                idx++;
            }
        }

        // 정렬된 새 배열을 기존의 배열 복사하여 옮겨준다.
        for (int k = left; k <= right ; k++) {
            a[k] = sorted[k];
        }
        
    }


}
```

합병 정렬은 데이터를 '비교'하면서 찾기 때문에 **비교정렬**이며 대상이 되는 데이터 외에 추가적인 공간을 필요로 하기 때문에 **제자리 정렬(in-place sort)**이 아니다. 
그러나 합병정렬은 앞의 부분리스트부터 차례대로 합쳐나가기 때문에 **안정정렬**알고리즘이다.
