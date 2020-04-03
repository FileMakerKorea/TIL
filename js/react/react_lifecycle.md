# 리액트 라이프사이클

[Youtube ZeroCho TV: React 기본 강좌 5-1. 리액트 라이프사이클 소개](https://www.youtube.com/watch?v=ltw4FYagLfM) 듣고 정리.

- **라이프 사이클**
    - 컴포넌트 렌더링 결과를 DOM에 추가하는 순간에 수행해야 하는 작업이 있을 경우, 라이프 사이클을 사용한다
- 우선 크게 세 가지 라이프사이클을 살펴보자
    - **componentDidMount**
        - 컴포넌트가 성공적으로 실행되었다면, 그 직후에 `componentDidMount` 가 실행된다 (첫 번째 렌더링)
        - setState 등으로 인해 렌더링이 다시 이루어졌을 때는 `componentDidMount` 가 다시 실행되지 않는다
    - **componentWillUnmount**
        - 컴포넌트가 제거되기 직전에 실행된다 (부모 컴포넌트에 의해서 제거될 때)
        - componentDidMount 와 함께 쌍으로 사용되는 경우가 많다
    - **componentDidUpdate**
        - state나 props의 변경으로 인해 컴포넌트가 다시 렌더링 될 때 동작한다
- 라이프사이클(클래스)을 순서대로 살펴보면 다음과 같다
    1. Constructor
    2. render
    3. ref
    4. componentDidMount
    5. setState, props 바뀔 때 → shouldComponentUpdate → rendering → componentDidUpdate
    6. 부모 컴포넌트에서 현재 컴포넌트를 제거 → componentWillUnmount → 컴포넌트 제거됨
