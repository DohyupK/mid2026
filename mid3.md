# 3번 답안
## 3번 문항 : 숫자야구 판정 게임을 작성하시오.
```
나의 프롬프트 :
C언어로 숫자야구 판정 프로그램을 작성해줘.
초기 정답 숫자는 1, 2, 3으로 설정한다.
사용자가 입력한 숫자 3개와 정답을 비교한다.
숫자와 위치가 모두 같으면 Strike, 숫자만 같고 위치가 다르면 Ball로 계산한다.
3 Strike이면 "HOMERUN"을 출력한다.
변수, 배열, 반복문, 조건문, 함수 등 C언어 기초 문법을 중심으로 작성한다.
코드는 최대한 간결하게 작성하고, 출력 메시지는 영어로 표시한다.
```
```
#include <stdio.h>
#define SIZE 3

typedef struct {
    int strike, ball;
} Result;

Result checkBaseball(const int answer[], const int input[]) {
    Result result = { 0, 0 };

    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            if (answer[i] == input[j]) {
                if (i == j)
                    result.strike++;
                else
                    result.ball++;
            }
        }
    }
    return result;
}

int main(void) {
    const int answer[SIZE] = { 1, 2, 3 };
    int input[SIZE];

    printf("Enter 3 numbers: ");

    for (int i = 0; i < SIZE; i++) {
        if (scanf_s("%d", &input[i]) != 1)
            return 1;
    }

    Result result = checkBaseball(answer, input);

    printf("%d Strike, %d Ball\n", result.strike, result.ball);

    if (result.strike == SIZE)
        printf("HOMERUN\n");
    return 0;
}
```
<img width="1113" height="244" alt="image" src="https://github.com/user-attachments/assets/e9f9041f-ba9c-41e1-bd34-bd0e637e5ab6" />
