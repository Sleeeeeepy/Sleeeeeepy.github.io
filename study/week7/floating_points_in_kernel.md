# 커널에서의 부동소수점 연산
커널은 부동소수점(floating point) 연산을 지원하지 않습니다. 커널이 느리게 작동할 수 있다는 대략적인 이유가 있지만 그것만으로는 부동소수점 연산을 지원하지 않는 것은 이해가 되지 않습니다. 부동소수점은 C 언어에 기본적으로 있기도 하고, 유저 모드에서는 부동소수점 연산을 사용하기도 하기 때문입니다. 그러나 어떤 프로세서는 부동소수점 연산을 지원하지 않기도 합니다. 이런 환경에서 커널이 부동소수점 연산을 수행한다면 FPU를 모방해야 하기 떄문에 상당히 느리게 작동합니다. 그리고 표준 라이브러리에는 이런 상황에 대비해서 어떤 CPU에서든 프로그램이 정확하게 작동하도록 `__fixfsi`와 같은 부동소수점 연산 루틴이 들어가 있습니다.

즉, 요약하자면 아래와 같습니다.
1. 부동수소점을 사용하기 위해 트랩을 제어하기 어렵다.
2. 부동소수점 연산은 느리다.
3. 어떤 환경에서는 부동소수점을 지원하지 않는다. (그렇기 때문에 느리다)

## See also
그렇다면 아래 코드는 어떻게 `__fixfsi`에 대한 레퍼런스가 없다는 오류를 낼까요?
``` c
float f = 3.33f;
printf ("%d", (int)f);
```

우선, 컴파일 옵션을 봅시다. 우선 `__fixfsi`는 `float` 타입을 `int` 타입으로 변환하는 함수라는 사실만 유념합니다.
```
CFLAGS = -g -msoft-float -O0 -fno-omit-frame-pointer -mno-red-zone
CFLAGS += -mcmodel=large -fno-plt -fno-pic -mno-sse
CPPFLAGS = -nostdinc -I$(SRCDIR) -I$(SRCDIR)/include/lib -I$(SRCDIR)/include
CPPFLAGS += -I$(SRCDIR)/include/lib/kernel
```
여기에서, 표준 라이브러리를 사용하지 않는다는 사실을 알 수 있습니다.  그리고 soft-float를 활성화 했기 때문에 부동소수점 연산이 `__fixfsi`와 같은 함수의 호출로 변환되고, 이 함수가 현재 include에 존재하지 않으므로 사용할 수 없습니다.

사실 운영체제는 표준 라이브러리를 사용할 수 없습니다. 표준 라이브러리에서 운영체제에 의존하고 있는 부분이 있기 때문입니다. pintos에서는 이 때문에 lib/include에서 표준 라이브러리를 대응하고 있습니다.