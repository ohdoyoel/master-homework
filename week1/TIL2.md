## 고급 자료형

Array나 Object와 같은 기본 자료형을 확장한 자료형

### Map

ES6에서 도입된 어떤 형태의 키를 받아 키-값 쌍을 저장하는 컬렉션 자료형

기본 특징

- 키 타입 다양성 : 객체와 달리 모든 타입(객체, 함수, 원시값 등)을 키로 사용 가능
- 순서 보장 : 삽입 순서가 유지됨
- 크기 추적 : size 프로퍼티 제공 -> 쉽게 요소의 개수 확인 가능
- 전용 메서드 : 맵 전용 유용한 메서드 제공
- 성능 : 삭제 추가에 있어 객체보다 이점 존재

이러한 기본 특징이 곧 Object보다 Map을 사용해야하는 이유가 된다. Map의 강점을 알고 있어야 실제 개발에서 적극적으로 활용할 수 있다.

#### 왜 Map이 Object에 비해 추가 삭제에 있어 성능이 좋은가?

Map은 해시 테이블 기반 구현 -> 평균적으로 O(1) 시간 복잡도를 가진다. 물론 객체도 O(1) 시간 복잡도에 준하지만, 차이가 있다.

V8 엔진에서는 객체를 생성할 때마다 해당 객체의 "모양" (프로퍼티 구조)에 따라 Hidden Class라는 것을 동적으로 생성한다. 객체에 새 프로퍼티가 추가되면 새로운 Hidden Class가 생성되고, 기존 객체는 이 새 클래스를 참조하도록 업데이트된다.

주의사항

- 동일한 Hidden Class를 공유하는 객체는 더 빠르게 접근 가능
- 프로퍼티 추가 순서가 다르다 -> 다른 Hidden Class가 생성되어 성능이 저하된다.
- 동적 프로퍼티 추가/삭제가 빈번하면 Hidden Class 변경이 잦아져 성능이 저하된다.

##### 객체의 Dictionary Mode (Slow Properties)

1. 전환 조건

- 프로퍼티가 매우 자주 추가/삭제될 때
- 프로퍼티 이름이 매우 다양할 때 (예 : 동적 생성 키)
- 대량의 프로퍼티가 있을 때

2. 특징

- 더 이상 Hidden Class를 사용하지 않고 해시 테이블 방식으로 전환
- 유연성은 높지만 접근 속도가 느려짐
- 메모리 사용량이 증가

3. Map과의 차이

- Map은 처음부터 해시 테이블로 설계되어 Dictionary Mode의 오버헤드가 없다.
- 객체는 Hidden Class 최적화를 위해 설계되었지만, Dictionary Mode로 전환되면 Map보다 느려질 수 있다.

#### Map의 기본 용법

```js
const map = new Map();

// 값 설정
map.set("name", "Alice");
map.set(42, "The Answer");
map.set({ id: 1 }, "Object Key");

// 값 가져오기
console.log(map.get("name")); // Alice
console.log(map.get(42)); // The Answer

// 크기 확인
console.log(map.size); // 3

// 존재 여부 확인
console.log(map.has("name")); // true

// 값 삭제
map.delete(42);

// 전체 초기화
map.clear();
```

#### Map의 반복 메서드

```js
const map = new Map([
  ["a", 1],
  ["b", 2],
  ["c", 3],
]);

// 키-값 쌍 순회
for (const [key, value] of map) {
  console.log(key, value);
}

// 키만 순회
for (const key of map.keys()) {
  console.log(key);
}

// 값만 순회
for (const value of map.values()) {
  console.log(value);
}

// forEatch 사용
map.forEach((value, key) => {
  console.log(key, value);
});
```

#### Map의 초기화 방법

```js
// 배열로 초기화
const map = new Map([
  ["key1", "value1"],
  ["key2", "value2"],
]);

// 객체로 초기화
const obj = { a: 1, b: 2 };
const mapFromObj = new Map(Object.entries(obj));
```

#### Map의 사용 사례

1. 키 타입 다양성 : 객체와 달리 모든 타입(객체, 함수, 원시값 등)을 키로 사용 가능
2. 순서 보장 : 삽입 순서가 유지됨
3. 크기 추적 : size 프로퍼티 제공 -> 쉽게 요소의 개수 확인 가능
4. 전용 메서드 : 맵 전용 유용한 메서드 제공

이 목적에 맞게 사용하는게 정석이다.

#### Map을 이용한 함수 결과 캐싱

```js
const fibCache = new Map();

function fibonacci(n) {
  // base case
  if (n <= 1>) return n;

  // check cache
  if (fibCache.has(n)) {
    return fibCache.get(n);
  }

  // recursive case
  const result = fibonacci(n - 1) + fibonacci(n - 2);

  // cache result
  fibCache.set(n, result);

  return result;
}
```

#### Map을 이용한 사이즈 추적 & 동적 폼 필드 관리

```js
function DynamicForm() {
  const [fields, setFields] = useState(
    new Map([
      ["username", ""],
      ["email", ""],
    ])
  );

  const handleChange = (fieldName, value) => {
    // Map의 set 메서드로 업데이트
    const newFields = new Map(fields).set(fieldName, value);
    setFields(newFields);
  };

  const addField = () => {
    const newFields = new Map(fields).set(`field-${fields.size + 1}`, "");
    setFields(newFields);
  };

  return (
    <div>
      {Array.from(fields.entries()).map(([key, value]) => (
        <input
          type="text"
          value={value}
          onChange={(e) => handleChange(key, e.target.value)}
        />
      ))}
      <button onClick={addField}>Add Field ({fields.size} fields)</button>
    </div>
  );
}
```

#### 순서 보장이 중요한 케이스 (드래그 가능한 리스트)

이 경우, @hello-pangea/dnd를 활용한다.

```js
import { useState } from "react";
import {
  DragDropContext,
  Droppable,
  Draggable,
  DropResult,
} from "@hello-pangea/dnd";

function DraggableList() {
  const [items, setItems] = useState(
    new Map([
      [1, { id: 1, text: "First" }],
      [2, { id: 2, text: "Second" }],
      [3, { id: 3, text: "Third" }],
    ])
  );

  const handleDragEnd = (result: DropResult) => {
    if (!result.destination) return;

    const newItems = new Map(items);
    // 삽입 순서 유지를 위해 배열로 변환 후 재구성
    const itemsArray = Array.from(items.entries());

    // 드래그 아이템 제거
    const [removed] = itemsArray.splice(result.source.index, 1);
    // 새로운 위치에 삽입
    itemsArray.splice(result.destination.index, 0, removed);

    // 순서 유지하면 새 Map 생성
    itemsArray.forEach(([id, item]) => {
      newItems.set(id, item);
    });

    setItems(newItems);
  };

  return (
    <DragDropContext onDragEnd={handleDragEnd}>
      <Droppable droppableId="items">
        {(provided) => (
          <div ref={provided.innerRef} {...provided.droppableProps}>
            {Array.from(items.entries()).map(([id, item], index) => (
              <Draggable key={id} draggableId={id.toString()} index={index}>
                {(provided) => (
                  <div
                    ref={provided.innerRef}
                    {...provided.draggableProps}
                    {...provided.dragHandleProps}
                  >
                    {item.text}
                  </div>
                )}
              </Draggable>
            ))}
          </div>
        )}
      </Droppable>
    </DragDropContext>
  );
}

export default DraggableList;
```

useState를 사용하지 않았다면, delete와 set을 활용해서 구현해야 했을 것이다.

```js
// 기존의 init Map 데이터 -> 신규 Map 데이터로 변환하는 순수 함수를 만들어보자.

const mapToMap = (map, removedIndex, newIndex) => {
  const newMap = new Map();

  const itemsArray = Array.from(map.entries());

  const [removed] = itemsArray.splice(removedIndex, 1);

  itemsArray.splice(newIndex, 0, removed);

  itemsArray.forEach(([id, item]) => {
    newMap.set(id, item);
  });

  return newMap;
};
```

### WeakMap

Weak이 들어간건 잘 안쓴다. 왜냐하면 Weak이 붙으면 DOM Controller와 연결된 경우가 많은데, 제일 강력한 특징은 키가 GC 대상이 되면 해당 엔트리도 자동으로 삭제한다.

1. 키로만 약한 참조 (Weak Reference)를 갖는 Map
2. 키가 GC 대상이 되면, 해당 엔트리도 자동으로 삭제한다.
3. Map과 달리 키는 반드시 객체이다.

#### 캐시 시스템에서 메모리 누수 방지

```js
const cache = new WeakMap(); // 키가 GC되면 값도 자동 제거한다.

function getHeavyData(obj) {
  if (!cache.has(obj)) {
    // obj가 캐시되어있지 않다면
    const result = heavyComputation(obj);
    cache.set(obj, result); // obj가 살아있는 동안만 캐시 유지
  }
  return cache.get(obj);
}
```

- 일반 Map을 사용하면 obj가 사라져도 cache에서 제거되지 않아 누수가 발생한다.
- WeakMap을 사용하면 obj가 GC되면 캐시도 자동 정리된다.

#### DOM 요소와 메타데이터 연결 시

```js
// 클릭 수를 추적하는 예시

const domMetadata = new WeakMap(); // DOM이 제거되면 메타데이터도 삭제된다.

const element = document.getElementById("my-element");
domMetadata.set(element, { clicks: 0 });

element.addEventListener("click", () => {
  const meta = domMetadata.get(element);
  meta.clicks++;
});
```

#### WeakMap의 한계와 대안

1. 키만 약한 참조이기 때문에 값은 GC 대상이 아님. 즉, 값이 큰 데이터라면, 키가 사라져도 값은 메모리에 남을 수 있음.
2. 순회 불가능 : 캐시 전체를 조회할 수 없음.

##### 대안 : WeakRef + FinalizationRegistry (ES2021)

```js
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`${heldValue}가 GC되었습니다!`);
});

const obj = { data: "Large Object" };
registry.register(obj, "obj"); // obj가 GC되면 콜백 실행

const weakRef = new WeakRef(obj); // 약한 참조 발생
```

- weakRef : 객체에 대한 약한 참조 생성
- FinalizationRegistry : 객체가 GC될 때 콜백 실행 (GC 클린업 시 실행할 콜백 함수)

FinalizationRegistry란?

FinalizationRegistry는 JS의 기능으로, 객체가 GC될 때 콜백 함수를 실행할 수 있게 해줌. ES2021에서 도입되었으며, 메모리 관리와 관련된 고급 시나리오에서 활용됨.

But, 가비지 켈렉션은 직시 실행되지 않는다. JS의 가비지 켈렉션은 비결정적이며, 메모리 부족 상황이 발생할 때에만 실행된다.

역할

- 객체가 GC되면 등롣된 콜백을 호출
- 주로 리소스 정리(파일 핸들, 네트워크 연결 해제)에 사용

작동 조건

- 객체에 더이상 강한 참조(Strong Reference)가 없어야 함
- GC 실행 시점은 JS엔진에 의존적이므로 즉시 실행되지 않을 수 있음

클라이언트 단에서 무거운 작업은 수학적 작업, 그래픽 작업, 또는 용량이 큰 정적 파일 자원 (이미지, 영상, 파일) 전처리를 하는 상황..

이때 코드에서는 이를 변수에 담아서 사용할 것. 용량이 크기 때문에, 적절히 해제되는지 확인하는 것이 중요. 이때 FinalizationRegistry를 사용하여 해제되는지 확인한다.

```js
registry.register(fileObject);
registry.unregister(fileObject);
```

- register : 객체에 대한 참조를 등록
- unregister : 객체에 대한 참조를 제거

이렇게 하면 해당 객체의 GC여부를 관찰할 수 있다.

```js
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`객체 ${heldValue}가 GC되었습니다!`);
});

let obj = { key: "value" };
const ref = new WeakRef(obj);

registry.register(obj, "obj 참조");
obj = null; // 강한 참조 제거
```

#### 실제 예제 : 파일 핸들러 정리

시나리오 : 파일을 열고 사용 후 자동으로 리소스를 해제하는 코드

```js
const fs = require('fs');
const fileRegistry = new FinalizationRegistry((heldValue) => {
  console.log(`파일 ${heldValue}가 GC되었습니다!`);
  try {
    fs.closeSync(filePath);
  } catch (error) {
    console.error(`파일 ${filePath} 닫기 실패: ${error.message}`);
  }
});

aasync function openFile(path) {
  try {
    const fileHandle = await fs.promises.open(path, 'r');
    fileRegistry.register(fileHandle, path);
    return fileHandle;
  } catch (error) {
    console.error(`파일 ${path} 열기 실패: ${error.message}`);
    throw error;
  }
}

// 사용 예시
async function main() {
  try {
    let handle = await openFile("example.txt");
    // 파일 작업 수행
    const content = await handle.readFile();
    console.log("파일 내용:", content.toString());
    // handle에 대한 참조를 제거하면 GC 후 콜백 실행
    handle = null;
  } catch (error) {
    console.error("예외 발생:", error.message);
  }
}

main();
```

### Set

ES6에서 도입된 고유한 값(중복 없는 값)들의 컬렉션을 저장하는 자료형

1. 중복 불가 : 모든 값은 유일해야 함
2. 순서 보장 : 삽입 순서가 유지됨
3. 빠른 검색 : 값 존재 여부를 빠르게 확인 가능 O(1) (`[].includes()`(O(n))보다 훨씬 빠르다)
4. 크기 추적 : size 프로퍼티로 요소 개수 확인 가능

빠른 검색이 Set의 주 특징이다. 빠른 검색이 필요하다면, Set을 사용하자. 즉, 긴 리스트의 형태의 데이터에서 값의 존재 여부를 판별해야할 때, 굳이 Array로 사용하지 말고 Set을 사용하자.

단 배열은 index를 사용할 수 있다는 장점이 있다.

#### 가장 흔한 사용법 : 중복 제거

```js
// 배열로 초기화
const set = new Set([1, 2, 3]);

// 중복 제거된 배열 생성
const arr = [1, 2, 2, 3, 3, 3];
const uniqueArr = [...new Set(arr)]; // [1, 2, 3]
```

#### 실전 예시 : 태그 입력 인풋 컴포넌트

```js
"use client";
import { useState } from "react";

const TagInput = () => {
  const [tags, setTags] = useState < Set < string >> new Set(); // Set으로 관리하자!
  const [inputValue, setInputValue] = useState < string > "";

  const addTag = () => {
    if (inputValue && !tags.has(inputValue)) {
      setTags(new Set(tags).add(inputValue));
      setInputValue("");
    }
  };

  const removeTag = (tag: string) => {
    const newTags = new Set(tags);
    newTags.delete(tag);
    setTags(newTags);
  };

  return (
    <div>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyDown={(e) => e.key === "Enter" && addTag()}
      />
      <button onClick={addTag}>Add</button>
      <div>
        {Array.from(tags).map((tag) => (
          <span key={tag} onClick={() => removeTag(tag)}>
            {tag}
            <button onClick={() => removeTag(tag)}>X</button>
          </span>
        ))}
      </div>
    </div>
  );
};
```

#### 실전 예시 : 쇼핑몰 장바구니 데이터

```js
const Cart = () => {
  const [cartItems, setCartItems] = useState(new Set());

  const addToCart = (productId: string) => {
    // 이미 존재하는 상품은 추가하지 않음
    setCartItems(new Set(cartItems).add(productId));
  };

  const removeFromCart = (productId: string) => {
    setCartItems((prev) => {
      const newCart = new Set(prev);
      newCart.delete(productId);
      return newCart;
    });
  };

  return (
    <div>
      <h2> 장바구니 ({cartItems.size}개 상품)</h2>
      <ProductList onAddToCart={addToCart}/>
      <ul>
        {Array.from(cartItems).map((id) => (
          <CartItem key={String(id)} productId={id as string} onRemove={removeFromCart}/>
        ))}
      </ul>
    </div>
  );
};

export default Cart;
```

- 배열을 사용한다 : 중복을 허용하겠다, 중복이 있을 수 있는 리스트 데이터이다.
- Set를 사용한다 : 중복을 허용하지 않는다. 중복이 되어선 안되는 리스트 데이터이다.

이런 식으로 개발자의 의도를 다른 개발자에게 선명하게 보여주어야 한다. 좋은 코드는 자료형만 보아도 의도가 보인다!

##### lodash라는 모듈

배열의 중복 처리를 도와줌. 근데 이거 쓸바엔~ Set 쓰자고~

```js
import _ from "lodash";

_.uniqBy([1, 2, 2, 3, 3, 3], (item) => item); // [1, 2, 3]
```

#### 실전 예시 : 소셜 서비스의 좋아요 캐싱
