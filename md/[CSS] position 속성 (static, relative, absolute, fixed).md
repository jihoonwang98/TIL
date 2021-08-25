# [CSS] position 속성 (static, relative, absolute, fixed)



`position`은 레이아웃을 배치하거나, 객체를 위치시킬 때 사용하는 css 속성이다.

`position` 속성은 상속되지 않으며, `top`, `bottom`, `left`, `right`의 위치를 같이 설정할 수 있다.



### `position` 속성값 종류

- `static` (기본값): 위치를 지정하지 않을 때 사용한다.
- `relatvie`: 위치를 계산할 때 `static`의 원래 위치부터 계산한다.
  - 다른 요소들이 `relative` 요소는 `static` 속성을 가지고 있다고 생각하고 위치함
- `absolute`: 원래 위치와 상관없이 위치를 지정할 수 있다. 단, 가장 가까운 부모 요소를 기준으로 위치가 결정된다. 근데 이 부모 요소가 `fixed`, `absolute`, `relative`한 `position`을 가지는 부모여야 한다. 만약 그런 부모가 없으면 defalut로 body를 부모로 가진다.
  - 다른 요소들이 `absolute` 요소는 아예 무시하고 위치함
- `fixed`: 원래 위치와 상관없이 위치를 지정할 수 있다. 하지만 상위 요소에 영향을 받지 않기 때문에 화면이 바뀌더라도 고정된 위치를 설정할 수 있다. 브라우저 화면의 상대 위치를 기준으로 위치가 결정된다.