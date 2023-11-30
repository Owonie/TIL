# Million.js 가 React 보다 최대 70% 빠른 이유

- Million.js ?
  단순히 React Component 를 HOC로 감싸는 것으로 렌더링 속도를 빠르게 만들어 주는 virtual DOM 라이브러리. React reconciliation 과정을 생략하여 virtual DOM 을 조작하는 방식으로 성능을 개선했다고 한다.

- Speedy ?
  React 의 diff 알고리즘은 O(n) 의 시간복잡도를 갖고있다. node 의 수에 따라 diff 에 걸리는 시간은 선형적으로 늘어나게 된다. Million.js 는 reconciliation 을 사용하지 않아 tree 를 렌더마다 재생성하지 않는다. 대신 fine-grained reactivity framwork 에 가깝게 필요한 부분만 update 하는데 이를 위해 정적 분석과 더티 체킹을 사용한다.

1. 컴포넌트를 정적 분석해 JSX tree 의 변경 될 수 있는 부분의 위치와 데이터를 수집하고 저장한다. (데이터는 Edit Map 이라 부름)
2. 컴포넌트가 update 되면 Edit Map을 통해 데이터를 이전과 비교(Dirty Checking) 함으로써 변경된 부분만 DOM 업데이트를 진행한다.
   위 처리 과정은 JSX trree 의 expression 의 개수에 선형 복잡도를 갖기에 시간복잡도를 대폭 줄일 수 있다.

- Implements ?
  구현 방식은 3가지 영역으로 구분지을 수 있다.

1. Block Virtual DOM
2. Compiler
3. React HOC

Million.js 가 사용하는 시스템은 기존 virtual DOM 들과 다른 접근 방식을 가진다. 컴포넌트를 block 단위로 관리하며 컴포넌트의 node 를 static 하게 관리한다. (동적 부분은 Edit Map에 저장됨)
update 가 있다면 Edit Map 의 데이터만 비교하는 방식으로 빠른 렌더링 속도를 제공한다.

expression 을 props 로 넘기도록 구현해야 해서 compiler 를 사용한다. compiler 는 block 으로 감싸져 있는 컴포넌트를 두개로 분리한다.

1. return JSX tree 그대로 return 하며 expression 값들만 props 로 받는 stateless component + 해당 컴포넌트를 block HOC 로 감싼 Block
2. Block 을 return 하고 나머지는 그대로인 원본 컴포넌트

```ts
/*** 원본 컴포넌트 ***/
function App(props) {
  const [cnt, setCnt] = useState(0);

  return (
    <div id={props.id} onClick={() => setCnt(cnt + 1)}>
      Hello, {props.name} - ({cnt})
    </div>
  );
}
const AppBlock = block(App);

/*** 컴파일된 컴포넌트 ***/
function BlockUI({
  _$2, // {variable} 형태가 아니라면 key를 자동으로 생성해 준다.
  _$3,
  _$4,
  cnt,
}) {
  return (
    <div id={_$2} onClick={_$3}>
      Hello, {_$4} - ({cnt})
    </div>
  );
}
const BlockComponent = block(BlockUI); // (1)

function App(props) {
  // (2)
  const [cnt, setCnt] = useState(0);

  return BlockComponent({
    _$2: props.id,
    _$3: () => setCnt(cnt + 1),
    _$4: props.name,
    cnt,
  });
}
const AppBlock = App;
```

- props 가 변수가 아닐 때는 auto-increment 로key 를 생성한다.
- `BlockComponent` 에 사용된 `block` 함수는 React HOC 함수이다.

- React HOC
  HOC 형식으로 컴포넌트를 감싸주는 문법을 택한 Million.js 는 Block Virtual DOM 이 프록시 역할을 도맡아 React Component 의 생애주기에 관여한다.

```ts
const css = 'million-block, million-fragment { display: contents }'; // 1. temp tag용 style 설정
const style = document.createElement('style');
Object.assign(style, {
  type: 'text/css',
  innerHTML: css,
});
document.head.appendChild(style);

export const block = (fn, options = {}) => {
  const block = createBlock(fn, unwrap);

  return function MillionBlock(props) {
    const ref = useRef(null); // tmep tag DOM ref
    const patch = useRef(null); // patch function

    patch.current?.(props); // 3. patch 호출

    useEffect(() => {
      const currentBlock = block(props, props.key);

      if (ref.current) {
        currentBlock.mount(ref.current, null); // 2. block을 mount 한다.

        patch.current = (props) => {
          // patch ref 값을 설정한다.
          currentBlock.patch(block(props));
        };
      }

      return () => {
        currentBlock.remove(); // 4. remove
      };
    }, []);

    return <million-block ref={ref} />; // million-block이라는 tag를 container로 사용한다.
  };
};
```

useEffect 를 이용해 block HOC 를 설계하는 모습을 코드에서 볼 수 있다. 얼핏 보면 증분돔 처럼 작동하는 것 처럼 보인다. 선언적으로 렌더링을 처리해주는 React 의 core 를 이런 방식으로 성능 개선을 할 수 있다는 것이 매우 흥미롭다.
