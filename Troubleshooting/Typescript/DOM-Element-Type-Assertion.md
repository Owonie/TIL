# DOM Element 조작 시 메서드 타입체크 에러

## 개요

attribute를 탐색하여 component 속성을 찾아내고, createElement와 createInstance를 처리해주려 하고 있는데, 타입 에러처리를 해줘야했다.
속성을 조회하는 hasAttribute는 Element 타입에만 존재하는데, 받아오는 $Child의 타입은 ChildNode였기 때문이다.

1번: 타입 단언

```ts
    const $fragment = document.createElement('div');
    $fragment.innerHTML = template;
    console.log($fragment);
    for (const $child of [...$fragment.childNodes]) {
      createElement('', this, $child as Element);
    }
-----------------------------------------------
	import Component from './Component';
	export default function createElement(
	  type: string,
	  component: Component,
	  $child: Element
	) {
	  if ($child !== null && $child.hasAttribute('component')) {
	    console.log($child);
	  }
	  [...$child.childNodes].forEach((child) => {
	    // child의 타입을 ChildNode에서 Element으로 타입단언
	    createElement(type, component, child as Element);
	  });
	}

```

2번: 타입 명시해주기

```ts
	interface HTMLDivElementExtended extends HTMLDivElement {
	  childNodes: NodeListOf<Element>;
	}
	 const $fragment = document.createElement('div');
	    const $fragmentExtended = $fragment as HTMLDivElementExtended;
	    $fragmentExtended.innerHTML = template;
	    console.log($fragment);
	    // createElement로 구현하도록 하겠습니다
	    for (const $child of [...$fragmentExtended.childNodes]) {
	      createElement('', this, $child);
	    }
	-------------------------------------------
	export default function createElement(
	  type: string,
	  component: Component,
	  $child: Element
	) {
	  const $ChildExtended = $child as HTMLDivElementExtended;
	  if ($ChildExtended !== null && $ChildExtended.hasAttribute('component')) {
	    console.log($child);
		}
		[...$ChildExtended.childNodes].forEach((child) => {
	    createElement(type, component, child);
	  });
	}
```

여기서도 as 문법을 사용하지만, 타입 단언이 아닌, 새로운 변수로 덮어 씌워주는 방식이라 조금 더 안전하지 않을까 싶다.

3. instanceof 문법 사용하기

```ts
	  const $element = document.createElement('div');
	  $element.innerHTML = template;
	  console.log($element);
	  for (const $child of $element.childNodes) {
	    if ($child instanceof Element) {
	      createElement('', this, $child);
	    }
	  }
  --------------------------------------------
	  export default function createElement(
	  type: string,
	  component: Component,
	  $element: Element
	) {
	  if ($element.hasAttribute('component')) {
	    console.log($element);
	  }
	  for (const $child of $element.childNodes) {
	    if ($child instanceof Element) {
	      createElement(type, component, $child);
	    }
	  }
	}
```

엄청 깔끔하게 instanceof Element를 해줌으로, 타입 추론이 가능케해 해당 이슈를 편하게 해결해주었다.
