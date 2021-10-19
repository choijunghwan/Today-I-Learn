# Bubble Sort (버블 정렬)
버블 정렬은 매번 연속된 두개 인덱스를 비교하여, 정한 기준의 값을 뒤로 넘겨 정렬하는 방법이다.

<br><br>

# 로직
1. 앞에서부터 현재 인덱스 값과 바로 다음 인덱스 값을 비교한다.
2. 만약 현재 인덱스 값이 더 크면 다음 인덱스값과 바꿔준다.
3. 현재 인덱스 값이 더 작으면 교환하지 않고 다음 연속된 배열값을 비교한다.
4. 이를 `전체 배열의 크기 - 현재까지 순환한 바퀴 수` 만큼 반복한다.

버블 정렬은 정렬에 대해 공부하면 가장 먼저 접하게 되기도 하고 가장 쉬운 정렬 방법중에 하나이기도 하다.

버블 정렬은 데이터를 비교하면서 찾기 때문에 **비교 정렬**이며 추가적인 공간이 필요하지 않아 **제자리 정렬(in-place sort)**이다.

그리고 삽입 정렬과 동일하게 **안정 정렬** 이다.

<br><br>

# 구현
```Java
public class Bubble_Sort {
    
    public static void bubble_sort(int[] a) {
        bubble_sort(a, a.length);
    }

    //오름차순 정렬
    private static void bubble_sort(int[] a, int size) {

        for (int i = 1; i < size; i++) {
            for (int j = 0; j < size - i; j++) {

                if (a[j] > a[j + 1]) {
                    swap(a, j, j + 1);
                }
            }
        }
    }

    private static void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }

}
```