# 4번 답안
## 4번 문항 : 높이 N을 입력받아 중앙 정렬된 피라미드 형태의 문자 패턴을 출력하시오. 홀수 번째 행은 *, 짝수 번째 행은 #으로 출력합니다. 입출력 예시 (N=4): 피라미드 모양
```
   *
  ###
 *****
#######
```
```
나의 프롬프트 :
C언어로 중첩 반복문을 사용해 높이 N의 중앙 정렬 피라미드를 출력해 줘.
홀수 행은 `*`, 짝수 행은 `#`을 출력하고, 주석 없이 작성해 줘.
```
```
#include <stdio.h>

int main(void) {
    int N;
    scanf_s("%d", &N);
    for (int i = 1; i <= N; i++) {
        for (int j = 1; j <= N - i; j++) printf(" ");
        if (i % 2 == 1) {
            for (int j = 1; j <= 2 * i - 1; j++) printf("*");
        }
        else {
            for (int j = 1; j <= 2 * i - 1; j++) printf("#");
        }
        printf("\n");
    }
    return 0;
}
```
<img width="1112" height="308" alt="image" src="https://github.com/user-attachments/assets/acb5bf38-2a84-46d5-a791-d224cdadedf4" />
