---
layout: post
title: "Chapter 04: 변수"
date: 2025-07-29
categories: [javascript-deep-dive]
tags: [javascript, deep-dive, chapter04, variables, hoisting, scope]
---

# Chapter 04: 변수

## 📝 핵심 개념 요약
- **변수**는 값을 저장하기 위한 메모리 공간과 그것을 식별하는 이름
- **식별자**는 변수명, 함수명 등을 포함하는 상위 개념
- **var vs let**: 호이스팅 동작과 스코프 규칙이 다름
- **메모리 관리**: 재할당 시 새로운 메모리 공간 확보, 가비지 컬렉터가 자동 정리

## 📚 용어 사전
{% assign terms = site.data.glossary.javascript | where: "chapter", "04" %}
{% for term in terms %}
**{{ term.term }}**({{ term.english }}): {{ term.definition }}
{% if term.example %}
```javascript
{{ term.example }}
```
{% endif %}
{% endfor %}

**IIFE**(Immediately Invoked Function Expression): 정의와 동시에 즉시 실행되는 함수 표현식으로, 독립적인 스코프를 생성하여 변수 충돌을 방지하고 값을 캡처하는 데 사용
```javascript
(function(param) {
  // 독립적인 스코프
  console.log(param);
})(value); // 즉시 실행
```

## ⚖️ JavaScript vs C++ 비교

| 구분 | JavaScript | C++ |
|------|------------|-----|
| **변수 선언** | `var/let/const name` | `type name` |
| **타입 시스템** | 동적 타입 (런타임 결정) | 정적 타입 (컴파일 타임 결정) |
| **메모리 관리** | 자동 (가비지 컬렉터) | 수동 (`new/delete`) |
| **호이스팅** | var: undefined 초기화<br>let: TDZ | 없음 (선언 순서 중요) |
| **스코프** | var: 함수 스코프<br>let: 블록 스코프 | 블록 스코프 |
| **재할당** | 새 메모리 공간 확보 | 같은 메모리 공간 값 변경 |

## 🧪 핵심 실험 코드

### 실험 1: 식별자 vs 변수
```javascript
// 식별자는 이름, 변수는 메모리 공간 + 이름
var score = 100;  // score: 식별자, 전체: 변수
//  ↑       ↑
// 이름   메모리 공간의 값
```

### 실험 2: TDZ (Temporal Dead Zone) 심화
```javascript
// TDZ: 선언되었지만 초기화되지 않은 "일시적 사각지대"
{
  // ← 여기서부터 TDZ 시작
  console.log(letVar); // ReferenceError! (TDZ 구간)
  let letVar = "값";   // ← 여기서 TDZ 종료
}
```

**TDZ 존재 이유**: 선언 전 접근을 차단하여 **버그 예방**
**생명주기**: 선언 → (TDZ) → 초기화 → 사용 가능

### 실험 3: 메모리 재할당
```javascript
var value = 100;    // 메모리 주소 0x001
value = 200;        // 새 주소 0x002에 200 저장
value = "문자열";   // 새 주소 0x003에 문자열 저장
```

**결과**: JavaScript는 불변값 원칙으로 항상 새 메모리 공간 확보

**🔬 전체 실험 코드**: [chapter04-variables.js]({{ site.baseurl }}/assets/experiments/javascript/chapter04-variables.js)  
**🔗 C++ 비교 코드**: [comparison-variables.cpp]({{ site.baseurl }}/assets/experiments/cpp/comparison-variables.cpp)  
**📊 호이스팅 실험**: [hoisting-test.js]({{ site.baseurl }}/assets/experiments/javascript/hoisting-test.js)  
**🎯 TDZ & var 문제점**: [tdz-and-var-problems.js]({{ site.baseurl }}/assets/experiments/javascript/tdz-and-var-problems.js)  
**⚡ 동기/비동기 비교**: [sync-async-for-loop.js]({{ site.baseurl }}/assets/experiments/javascript/sync-async-for-loop.js)

## 💼 실무 적용 사례

### 1. 모던 JavaScript에서는 let/const 사용
```javascript
// 레거시 코드 (ES5 이전)
var userName = "John";

// 모던 코드 (ES6+)
let userName = "John";        // 재할당 필요시
const API_KEY = "abc123";     // 상수값
```

### 2. var의 핵심 문제점과 해결

#### ⚠️ 문제 1: 지연된 클로저 + 함수 스코프
```javascript
// ❌ 문제 상황: 비동기 + var
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // 3, 3, 3
}

// ✅ 정상 동작: 동기 실행
for (var i = 0; i < 3; i++) {
  console.log(i); // 0, 1, 2 정상!
}
```

**왜 3만 출력되나?**
1. `var i`는 함수 스코프 → 반복문 밖에서도 접근 가능
2. 반복문 종료 후 `i = 3`
3. 나중에 실행된 콜백들이 모두 같은 `i(=3)` 참조

#### ✅ 해결 방법
```javascript
// 방법 1: let 사용 (권장)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // 0, 1, 2
}

// 방법 2: IIFE로 값 캡처
for (var i = 0; i < 3; i++) {
  // IIFE(즉시 실행 함수): 각 반복마다 독립적인 스코프 생성
  (function(captured) {           // captured 매개변수로 현재 i 값을 고정
    setTimeout(() => console.log(captured)); // 고정된 값(0,1,2) 출력
  })(i);                         // 현재 i 값을 즉시 전달하여 실행
}
```

#### ⚠️ 문제 2: var 사용하는 이유?
- **99%**: 레거시 코드, 구형 브라우저 지원
- **1%**: 의도적 함수 스코프 활용 (매우 드문 케이스)
- **결론**: 모던 JavaScript에서는 `let/const` 권장

### 3. TDZ를 활용한 안전한 코드
```javascript
// let/const는 선언 전 접근을 차단하여 버그 예방
console.log(safeVar); // ReferenceError로 즉시 발견
let safeVar = "안전한 변수";
```

## 🤔 추가 탐구점
- [ ] **TDZ 심화**: const와 let의 TDZ 차이점
- [ ] **클로저 최적화**: V8 엔진의 클로저 메모리 관리
- [ ] **스코프 체인**: Lexical Environment와 Variable Environment 구조
- [ ] **함수 호이스팅**: 함수 선언문 vs 함수 표현식 vs 화살표 함수
- [ ] **실무 적용**: 레거시 코드에서 var → let/const 마이그레이션 전략
- [ ] **성능 비교**: var vs let 런타임 성능 차이
- [ ] **Babel 트랜스파일**: ES6+ → ES5 변환 시 스코프 처리 방식

---
**다음 챕터**: [Chapter 05: 표현식과 문](링크)  
**이전 챕터**: [Chapter 03: 자바스크립트 개발 환경과 실행 방법](링크)