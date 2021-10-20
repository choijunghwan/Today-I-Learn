# Quick Sort(퀵 정렬)
퀵 정렬은 합병정렬과 마찬가지로 분할 정복(Divide and conquer)을 이용하여 정렬을 수행하는 알고리즘이다.

pivot point라고 기준이 되는 값을 하나 설정 하는데, 이 값을 기준으로 작은 값은 왼쪽, 큰 값은 오른쪽으로 옮기는 방식으로 정렬을 진행한다. 이를 반복하여 분할된 배열의 크기가 1이되면 배열이 모두 정렬된 것이다.

<br><br>

# 로직
1. pivot point로 잡을 배열의 값 하나를 정한다. 보통 맨앞, 맨뒤, 중간 값중에 하나를 사용하는데 랜덤값으로도 정할 수 있다.
2. 분할을 진행하기에 앞서, 비교를 진행하기 위해 가장 왼쪽 배열의 인덱스를 저장하는 left변수, 가장 오른쪽 배열의 인덱스를 저장한 right 변수를 생성한다.
3. right부터 비교를 진행한다. 비교는 right가 left보다 클 때만 반복하며, 비교한 배열값이 pivot point보다 크면 right를 하나 감소시키고 비교한다. pivot point보다 작은 배열 값을 찾으면 반복을 중지한다.
4. 그 다음 left부터 비교를 진행한다. 비교는 right가 left보다 클 때만 반복하며, 비교한 배열값이 pivot point보다 작으면 left를 하나 증가시키고 비교를 반복한다. pivot point보다 큰 배열 값을 찾으면, 반복을 중지한다.
5. left 인덱스의 값과 right 인덱스의 값을 바꿔준다.
6. 3,4,5 과정을 `left < right`가 만족 할 때까지 반복한다.
7. 위 과정이 끝나면 left의 값과 pivot point를 바꿔준다.
8. 맨 왼쪽부터 left -1 까지, left + 1 부터 맨 오른쪽까지로 나눠 퀵 정렬을 반복한다.
   
퀵 정렬은 분할과 동시에 정렬을 진행하는 알고리즘이다.

각 정렬은 배열의 크기 N만큼 비교하며, 이를 총 분할 깊이인 logN만큼 진행하므로 총 비교횟수는 NlogN 즉 시간 복잡도는 O(NlogN)이다.

다만 퀵 정렬에는 최악의 경우가 존재하는데, 이는 배열이 이미 정렬이 되어있는 경우이다. 이 경우 분할이 N만큼 일어나므로 시간 복잡도는 O(N^2)이다.
이를 방지하기 위해 앞서 언급했던 전체 배열 값 중 중간값이나 랜덤 값으로 pivot point를 정하는 방법을 쓰기도 한다.

# 구현
```Java
public class Quick_Sort {

    public static void quick_sort(int[] arr) {
        sort(arr, 0, arr.length - 1);
    }

    private static void sort(int[] arr, int low, int high) {
        if (low >= high) return;

        int mid = partition(arr, low, high);
        sort(arr, low, mid - 1);
        sort(arr, mid, high);
    }

    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[(low + high) / 2];
        while (low >= high) {
            while (arr[low] < pivot) low++;
            while (arr[high] > pivot) high--;
            if (low <= high) {
                swap(arr, low, high);
                low++;
                high--;
            }
        }
        return low;
    }

    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

}
```