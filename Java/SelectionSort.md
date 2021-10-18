# Selection Sort(선택 정렬)
선택 정렬은 이름에 맞게 현재 위치에 들어갈 값을 찾아 정렬하는 배열이다. 현재 위치에 저장될 값의 크기가 크냐, 작냐에 따라 **최대 선택 정렬(Max-Selection Sort)**와 **최소 선택 정렬(Min-Selection Sort)**로 구분할 수 있다.

최소 선택 정렬은 오름차순으로 정렬된 것이고, 최대 선택 정렬은 내림차순으로 정렬된 것이다.

<br><br>

# 로직
1. 정렬 되지 않은 인덱스의 맨 앞에서 부터, 이를 포함한 그 이후의 배열값 중 가장 작은 값을 찾는다.
2. 가장 작은 값을 찾으면, 그 값을 현재 인덱스의 값과 바꿔준다.
3. 다음 인덱스에서 위 과정을 반복한다.

선택 정렬 알고리즘은 n-1개, n-2개, ... , 1개씩 비교를 반복한다.

즉, 전체 비교를 진행하므로 `시간복잡도는 O(n^2)`이다.

데이터를 비교하면서 찾기 때문에 **비교 정렬** 이며 정렬의 대상이 되는 데이터 외에 추가적인 공간을 필요로 하지 않기 때문에 **제자리 정렬(in-place sort)** 이기도 하다.

단점으로는 **불안정 정렬** 이다.
불안정 정렬이란 정렬을 할때 기존의 같은 기준의 값들의 순서가 보장되지 않는다는 것이다.

예를들어 학생들을 이름순으로 기록해놓은 데이터가 있다. 데이터를 성적기준으로 정렬을 하고 성적이 같다면 이름순으로 정렬해야 한다고 치자. 이럴때 **안정 정렬** 이면 성적기준으로 정렬하면서 이름순서가 그대로 유지되지만 **불안정 정렬**은 이름 순서가 보장되지 않는다.

<br><br>

# 구현
```Java
public class Selection_Sort {

    public static void selection_sort(int[] a) {
        selection_sort(a, a.length);
    }

    private static void selection_sort(int[] a, int size) {

        for (int i = 0; i < size - 1; i++) {
            int min_index = i;

            for (int j = i + 1; j < size; j++) {
                if (a[j] < a[min_index]) {
                    min_index = j;
                }
            }

            swap(a, min_index, i);
        }
    }

    private static void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }

}
```

