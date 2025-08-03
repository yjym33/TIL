![alt text](/React/image/모던리액트DeepDive.png) <br>
출처 : https://www.yes24.com/product/goods/123161563

<br>

# 리액트 개발 도구로 디버깅하기 (React DevTools 정리)

## React 개발 도구란?

- React DevTools는 React 기반 애플리케이션을 디버깅하기 위한 도구입니다.
- React 웹뿐 아니라 React Native 등 다양한 플랫폼에서도 사용할 수 있습니다. :contentReference[oaicite:1]{index=1}

## 개발 도구 설치

- **React Developer Tools – React** 확장 프로그램 설치

## 6.3 개발 도구 활용법

### Components 패널

- **컴포넌트 트리 구조**를 한눈에 확인 가능
- 각 컴포넌트의 **props**, **Hooks** 정보 확인
- 함수형 컴포넌트는 함수 이름이 잘 표시되며, 익명 함수나 HOC는 `_default`, `Anonymous` 등으로 표시됩니다. → 명시적인 함수명 또는 `displayName` 사용 권장 :contentReference[oaicite:2]{index=2}

### 주요 기능

- **눈 아이콘**: 이 컴포넌트가 렌더된 DOM 위치 강조
- **벌레 아이콘**: 콘솔에 해당 컴포넌트 정보를 출력
- **소스코드 보기**: 컴포넌트 정의로 바로 이동
- **props**:
  - 우클릭 → "Store as global variable"로 `window.$r`에 저장
  - "Go to definition" 클릭 시 정의 위치로 이동
- **Hooks 정보**:
  - use~ 훅 사용 시, 이름이 익명으로 보일 수 있음 → 기명 함수로 넘기면 더 명확하게 표시됨 :contentReference[oaicite:3]{index=3}
- **Rendered by**: 어떤 컴포넌트가 이 컴포넌트를 렌더링했는지 표시

### ✨ Profiler 패널

- React의 렌더링 과정을 시각화하여 **어떤 컴포넌트가 얼마나 렌더링되었는지, 어떤 작업이 느렸는지** 추적 가능 :contentReference[oaicite:4]{index=4}

#### 설정 항목

- **General**: 리렌더 시 컴포넌트 강조 표시
- **Debugging**: Strict Mode에서 로그 중복 출력 방지 설정
- **Profiler**: 렌더링 원인 기록 옵션 활성화

#### 프로파일링 요소

- 녹화, 새로고침, 지우기 버튼
- **Commit 차트**, **Components 목록**, **Flamegraph**, **Ranked view**, **Timeline** 제공 :contentReference[oaicite:5]{index=5}

#### 프로파일러 뷰 설명

- **Flamegraph**: 렌더링 작업의 수행 시간을 시각적으로 표현 (바가 넓을수록 렌더링 시간이 오래 걸림)
- **Ranked**: 렌더링 비용 순으로 컴포넌트를 정렬해 보여줌
- **Timeline**: 시간 흐름에 따라 렌더링 변화를 추적

#### 성능 최적화 팁

- 프로파일러를 활용해 **왜 해당 컴포넌트가 렌더링되었는지**, **불필요한 렌더링이 발생한 원인**을 파악하고 개선할 수 있습니다.
  - React의 렌더링은 **렌더 단계(render)** → **커밋 단계(commit)**로 이루어지며, devtools로 각 커밋 단위를 확인하면서 성능 병목을 쉽게 진단할 수 있습니다. :contentReference[oaicite:6]{index=6}
