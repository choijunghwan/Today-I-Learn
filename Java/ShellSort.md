# Shell Sort(셸 정렬)
셸 정렬은 삽입 정렬을 기반으로 한다.

삽입 정렬이란 현재 비교하고자 하는 target과 그 이전의 원소들과 비교하며 자리를 교환(swap) 하는 정렬 방법이다.

삽입정렬은 거의 정렬 된 상태에 가까울수록 시간복잡도가 O(N)에 가까워지는 장점이 있지만 역순에 가까울수록 이전의 원소들을 모두 탐색하며 교환(swap)해야하는 단점이 있다.

그리고 이러한 삽입정렬의 장점은 살리면서 단점을 최소화하기 위해 고안된 방식이 **셸 정렬**이다.

그럼 **셸 정렬**은 어떻게 삽입 정렬의 단점을 최소화 시킬 수 있을까??

바로 이전의 원소를 모두 비교, 교환하는 것이 아니라 일정 간격을 주기로 검사하면서 **target(타켓)** 의 위치를 잡아주는것이다.

예를들어 원소 10개가 있다면 처음에 간격을 3으로 설정하여 삽입정렬을 하고 다시 간격을 2로 설정하여 삽입정렬을 하고 마지막으로 간격을 1로 설정하여 삽입정렬을 하는것이다.

이렇게 간격을 다르게 하여 **삽입 정렬**을 반복하는것을 **셸 정렬**이라고 한다.

<br><br>

# 로직
1. 간격(gap)을 설정한다.
2. 각 간격별로 분류 된 서브(부분) 리스트에 대해 삽입정렬을 한다.
3. 각 서브(부분) 리스트의 정렬이 끝나면 간격을 줄인다.
4. 간격이 1이 될 때 까지 2번 과정으로 되돌아가며 반복한다.

셸 정렬은 기본적으로 삽입 정렬을 기반으로 하여 **제자리 정렬(in-place sort)** 이다. 다만 삽입 정렬과 다르게 일정 간격을 가지고 비교하기 때문에 **불안정 정렬**이다.

그러면 `간격은 어떻게 설정할까??` 라는 궁금증이 생기실꺼다.

답은 **적당히** 잘 설정하는게 답입니다. 무책임하게 들리실 수도 있지만 간격(gap)이 너무 적으면 자주자주 검사를 해야하니 속도가 느려지고 간격이 너무 많으면 오버헤드가 발생하기 때문입니다.

이러한 고민때문에 사람들이 좋은 간격을 찾는 실험을 했는데 평균적으로 좋은 간격들을 **갭 시퀀스(gap sequence)** 라고 합니다.

아래 글을 보면 많은 갭 시퀀스들이 있습니다.

https://en.wikipedia.org/wiki/Shellsort#Gap_sequences

가장 좋은 성능을 보여주는것은 Ciura 시퀀스이지만 저희는 편의상 갭 시퀀스를 {1, 2, 4,...}로 2의 승수로 정해서 진행하겠습니다.

간단하게 정렬과정을 보자.

![image1](/Img/ShellSort.png)

얼핏 보기에는 반복을 여러번 하기 때문에 오히려 삽입 정렬보다 성능이 떨어져 보이지만 예를들어 round1의 두 번째 서브리스트인 9와 2를 보자. 만약에 gap 없이, 그냥 삽입정렬을 하면 원래의 경우 총 5번의 비교, 교환 과정이 발생한다. 
하지만 위 처럼 gap을 두고 부분적으로 삽입정렬을 시행하기 때문에 2의 경우 단 한 회만에 gap의 크기인 4만큼 jump하여 위치를 잡는다.

이렇게 간격을 통해 **거의 정렬 된 상태**를 만들고 최종적으로 마지막에 삽입정렬을 시도하는 방법인 것이다.

<br><br>

# 구현

```Java
public class Shell_Sort {

    private final static int[] gap =
            { 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024,
                    2048, 4096, 8192, 16384, 32768, 65536,
                    131072, 262144, 524288, 1048576};

    public static void shell_sort(int[] a) {
        shell_sort(a, a.length);
    }

    private static void shell_sort(int[] a, int size) {
        int index = getGap(size); // 첫 gap을 사용할 index

        for (int i = index; i >= 0; i--) {
            for (int j = 0; j < gap[i]; j++) {
                insertion_sort(a, i, size, gap[i]);
            }
        }
    }

    private static void insertion_sort(int[] a, int start, int size, int gap) {
        // 부분 배열의 두 번째 원소부터 size까지 반복한다.
        for (int i = start + gap; i < size; i += gap) {
            
            int target = a[i];
            int j = i - gap;

            while (j >= start && target < a[j]) {
                a[j + gap] = a[j];
                j -= gap;
            }
            
            a[j + gap] = target;
        }
    }

    private static int getGap(int length) {
        int index = 0;

        int len = (int) (length / 2);
        while (gap[index] <= len) {
            index++;
        }
        return index;
    }

}
```