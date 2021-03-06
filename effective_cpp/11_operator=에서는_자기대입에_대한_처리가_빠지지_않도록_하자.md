# 11 : operator=에서는 자기대입에 대한 처리가 빠지지 않도록 하자

---
## 문제인식
복사 대입에는 항상 중복참조에 의한 **자기대입** 의 위험이 숨어있습니다.

```
class EFG;

class ABC {
public:
  // ...
	ABC& operator=(const ABC& rhs) {
    		delete rhs.pefg;
		pefg = new EFG(*rhs.pefg);
		return *this;
	}
private:
	EFG* pefg;
};

ABC abc
abc = abc;
```

abc에 자기 자신을 대입시키는 경우 어떤 결과가 일어나는지 살펴봅시다.
abc 객체의 대입 연산자에는 자기 자신이 파리미터로 넘어옵니다.
여기서 pefg에는 복사생성자를 통해 pefg 자신이 복사되어 할당 되지만, 그 다음 라인에서 곧바로 해제되어 쓰레기 값을 갖게 됩니다.<br>

위 코드에서는 자기대입이 분명히 보이지만, 아래와 같이 중복참조에 의한 자기대입은 구분하기 어렵습니다.

```
a[i] = a[j];
*px = *py;
```

## 문제해결
중복참조된 객체를 일일히 확인할 수 없으므로, 대입 연산자 내부에서 이러한 문제가 발생하지 않도록 해야 합니다.

### 일치성검사 (좋지 못한 방법)
대입 연산자 내부에서 파라미터로 넘어온 객체가 자기자신과 같은지 확인합니다.

```
ABC& ABC::operator=(const ABC& rhs){
	if (this == &rhs) return *this;
	delete pefg;
	pefg = new EFG(*rhs.pefg);
	return *this;
}
```

이로써 자기대입의 문제는 해결되었습니다.<br>

하지만 위 코드를 좋은 방법이라 말하기 어려운데, new 연산자 부분에서 "BAD_ALLOC"과 같은 에러가 터질 경우 문제가 되기 때문입니다.
비록 이는 자기대입에 의한 문제는 아니지만, 아래 방법을 이용하면 자기대입 문제와 new 연산자 문제를 동시에 해결할 수 있습니다.

### 복사/해제 순서바꾸기
```
ABC& ABC::operator=(const ABC& rhs){
  	EFG* temp = pefg;
	pefg = new EFG(*rhs.pefg);
  	delete temp;
	return *this;
}
```

위 코드는 일치성검사가 필요 없을 뿐더러, new 연산자에서 에러가 발생해도 아직 delete 되지 않은 상태이므로 안전합니다.
언뜻 보면 자기대입의 상황에서 불필요하게 자기자신을 복사하고 해제하도록 되어있기 때문에, 일치성검사 코드를 맨 위에 다시 붙이고 싶을 수 있습니다.
하지만 일치성검사 코드가 들어가면 그만큼 목적코드의 크기가 커질뿐더러, 분기로 인한 실행속도저하, CPU 명령어 선행인출, 캐시, 파이프라이닝의 효과가 떨어지는 등 단점이 더 큽니다.

### copy and swap
copy and  swap 이란 방법도 있습니다.
예외안전성과 깊은 관련이 있어 소개합니다.

```
class ABC {
public:
  ABC& operator=(const ABC& rhs) {
    ABC temp(rhs);
    swap(temp);
    return *this;
  }
private:
  void swap(ABC& rhs) {/*...*/}
}
```

혹은

```
class ABC {
public:
  ABC& operator=(ABC rhs) { //값 복사에 의한 rhs
    swap(rhs);
    return *this;
  }
private:
  void swap(ABC& rhs) {/*...*/}
}
```

## 확장
자기대입과 유사한 문제가 또 있습니다.

```
void foo(const XYZ& lhs, const XYZ& rhs) {
  if (lhs == rhs) return;
  // ...
}
```

프로그래머는 lhs와 rhs가 서로 다른 객체를 가리킬거라 생각하고 함수를 작성했겠지만, 어떤경우에는 lhs와 rhs가 같은 객체일 수 있습니다.
위 처럼 같은 타입의 서로다른 객체가 파라미터로 넘어오는 함수를 작성할 때는, 혹시 같은 객체가 들어왔을 때 오동작하지 않는지 확인해야 합니다.

---
## 정리
- 대입 연산자를 직접 구현할 때에는, 반드시 자기대입에 대한 대책을 마련해야 합니다.
- 두 개 이상의 객체에 대해 동작하는 함수가 있다면, 넘겨지는 객체들이 서로 같은 경우 문제가 되지 않는지 확인해야 합니다.
