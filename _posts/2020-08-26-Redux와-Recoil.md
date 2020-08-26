---
title: "Redux 와 Recoil"
date: 2020-08-26 23:26:28 -0400
categories: React Recoil
---



# Recoil 과 Redux 비교 분석

Redux 의 API 는 복잡함. 근본적으로 React 에서 사용하기 위해 나온 것이 아님. 

(대신 react-redux 와 같은 React 전용 wrapper 라이브러리가 있긴 함)

API와 동작 방식을 가능한 React스럽게 가져가면서 현상을 해결하기 위해 Recoil 을 제작함

REcoil 은 사용법이 간단함. Atom 과 Selector 를 생성하고, 컴포넌트에서 사용하는 것만 하면됨

selector 덕분에 덧붙이기 식 개발이 가능함



페이스북에서 내놓은 새로운 상태관리 라이브러리이다. 페이스북 리액트 코리아 그룹의 한 포스트에는 이러한 장점이 있다고 말해주었다.

- atom/selector라는 단위를 통해 derived state를 효과적으로 처리하고 상태의 "코드 분할"이 가능하게 합니다.
- 기존 상태관리 라이브러리(ex. MobX) 보다 훨씬 단순한 API를 제공하고, Concurrent Mode를 통합하는 것 까지 목표로 합니다.
- 페이스북 내부적으로 몇몇 프로젝트에 사용하고 있었고, 이걸 라이브러리화 한 것이라네요
- 공개된 코드는 very-early stage 라서 사용을 권장하진 않습니다.
  기본적으로 코드가 특별한 흑마법 없이 cache와 context 기반으로 되어있고 잘 정돈되어 있어서, Concurrent react 의 빌딩 블럭들을 활용하는 라이브러리들에게 훌륭한 교과서가 되줄 것 같네요.





Recoil 홈페이지에는 다음과 같은 장점을 적어놓았다.

- Minimal and Reactish
  Recoil works and thinks like React. Add some to your app and get fast and flexible shared state.
  리코일은 리액트처럼 일하고 생각한다. 앱에 일부를 추가하고 빠르고 유연한 공유 상태를 유지하십시오.
- Data-Flow Graph
  Derived data and asynchronous queries are tamed with pure functions and efficient subscriptions.
  파생 데이터와 비동기 쿼리는 순수한 기능과 효율적인
  구독에 길들여진다.
- Cross-App Observation
  Implement persistence, routing, time-travel debugging, or undo by observing all state changes across your app, without impairing code-splitting.
  코드 분할을 손상시키지 않고 앱 전체에서 모든 상태 변경을 관찰하여 지속성, 라우팅, 시간 이동 디버깅 또는 실행 취소를 구현하십시오





1. Flexible shared state
2. Derived data and querues
3. App-wide state observation



1. 수직적인 트리 구조

App 에 <RecoilRoot> 만 추가하면됨. atom을 띄워 보게 될거임

같은것을 참조함 import 해

useState => useRecoilState()

export const test = atom({

key:

value:

})

2. 너무 많은 업데이트로 인한 버그를 없앰

 바운딩박스. 어떤것들이 선택되고 어디에 있는지. 

선택자 : 상태가 변경될때 함수를 다시 계산할 수 있게 하는것. 

```
selector({
ket: 'selectionBoundingBox',
get: () => {
	const selectedIsd = get(selectionAotom)
}
})
```

get은 우리가 할 함수

선택된애들 먼저 가져오고(적용할 Atoms).

계산을 리턴하면됨



Hooks 보다 유연하다 -> 루프에서 호출할 수 있기 때문에 구조적으로 변경이 가능하다?

원자에서 다이렉트로 거치기 전에 가는것.

각 원자 앞에 선택자를 두는것임

ㅏ서버에서 ASync로 가져와서 만질수도 있음

selector 에서 Promise 함수만 도입하면됨



3. 앱 전체의 관찰

id마다 따로 저장되어 있어서 우리가 본 상태 그대로 공유할 수 있음

디버깅이나 지속성도 좋음

관찰을 할 수 있음

어디가 변경되었으니 여기로 이동하여 뭐하세요~ 이렇게

Link로 하면 해당하면 화면으로 바로 갈 수 있음





초기 상태라 실험적이지만 공식적으로 승인된 라이브러리임.

원래 concurrentmode 가 안됨. 

multi context 모두 그 위에 지어진것이다

많은 제한이 있는 초기 상태라 한다.   10개를 동시에 하려면 아직 불안정하고 실험적이다.

잘 작동하지만 실제로 테스트 하기에는 무리가 있다.

reuex스타일의 슈처세트가 리코일이다

