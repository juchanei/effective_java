# 16 : new 및 delete를 사용할 때는 형태를 반드시 맞추자

---
## 문제인식
메모리를 힙영역에 할당하는 방법은 두 가지가 있습니다

- new
- new []

메모리를 해제하는 방법 역시 두 가지입니다.

- delete
- delete []

할당과 해제는 반드시 같은 형식으로 짝을 맞춰서 해야 하며, 그렇지 않은 경우의 결과는 미정의사항입니다.

```
std::string addresses = new std::string[4];

// ...

delete[] addresses;  // delete addresses은 안됨.
```

상황에 따라서는 delete를 해야할지 delete[]를 해야할지 혼동되는 경우가 있는데, 아래 코드가 그 예입니다.

```
typedef AddressLines std::string[4];

AddressLines addresses = new AddressLines();

// ...

delete addresses; //????
```

위 코드는 new[]로 생성된 객체를 delete로 해제하고 있으므로 예측할 수 없는 결과를 일으킵니다.

## 문제해결
**배열타입을 typedef 타입으로 만들지 않는 것이 좋습니다.**

---
## 정리
- new는 delete로, new []는 delete []로 해제합니다.
- 배열타입을 typedef 타입으로 만들지 않는 것이 좋습니다.
