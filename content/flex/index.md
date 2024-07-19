---
emoji: ''
title: 'css: flex'
date: '2022-12-06 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---


## flex

- `flex: initial`
- `flex: auto`
- `flex: none`
- `flex: <positive-number>`

```jsx
보통은 flex-grow, flex-shrink, flex-basis 값을 각각 사용하지 않고 이 세 속성을 한번에 지정하는 flex 축약형을 많이 사용합니다. 
flex 축약형의 값은 flex-grow, flex-shrink, flex-basis 순서로 지정됩니다.
```

**flex 항목**을 `flex: initial`로 지정하면 `flex: 0 1 auto` 로 지정한 것과 동일하게 동작합니다. 이 경우, **flex 항목**들은 `flex-grow`가 0이므로 `flex-basis`값보다 커지지 않고 `flex-shrink`가 1이므로 **flex 컨테이너** 공간이 모자라면 크기가 줄어듭니다. 또, `flex-basis`가 `auto`이므로 **flex 항목**은 주축 방향으로 지정된 크기 또는 자기 내부 요소 크기 만큼 공간을 차지합니다.

`flex: auto` 로 지정하면 `flex: 1 1 auto`로 지정한 것과 동일하며, `flex:initial` 과는 주축 방향 여유 공간이 있을 때 **flex 항목**들이 늘어나서 주축 방향 여유 공간을 채우는 점만 다릅니다.

`flex: none`으로 지정하면 `flex: 0 0 auto`으로 지정한 것과 동일하며 **flex 컨테이너**의 크기 변화에도 **flex 항목** 크기는 변하지 않고 `flex-basis`를 `auto`로 지정했을 때 정해지는 크기로 결정됩니다.

이 축약형은 더 축약해서 `flex: 1` 이나 `flex: 2`처럼 쓸수도 있는데, 이는 `flex-grow` 만 지정하고 나머지는 1 0으로 사용한다는 뜻이다. 따라서 `flex: 2`는 `flex: 2 1 0`와 동일하게 처리됩니다.

### flex-direction

row, row-reverse, column, column-reverse

flex-box에는 주축과 교차축이 있는데 default 값은 row이므로 주축은 row를 기준으로 하며 교차축은 주축은 정확히 교차하는 교차축을 의미한다. 

여기에서 direction 값을 column으로 바꾸면 주축과 교차축에 값이 반대로 바뀐다.

### justify-content

주축에 대한 정렬이다.

flex-start(default), flex-end, center, space-between, space-around, stretch, space-evenly

### align-content

교차축에 대한 정렬로 flex-start, flex-end, center, space-between, space-around, stretch(default), flex-item 덩어리 전체가 교차축을 기준으로 정렬된다.

### align-items

개별 아이템 항목을 정렬할 때 사용하는 속성으로 교차축 기준으로 상하 가운데 설정할 수 있다.

### align-self

flex item 개별 속성을 정렬할 때 사용하는 속성으로 align items가 사용하는 값들을 인자로 받는다.

### flex-wrap

nowrap, wrap, wrap-reverse

flexbox에서는 default값이 nowrap이기 때문에 기본적으로 박스 크기가 넘칠 경우 박스 공간을 뚫고 나가는 식으로 레이아웃이 배치된다.

부모 컨테이너에 wrap속성을 사용할 경우 박스 공간 아래로 항목들이 자동으로 매치된다.

flex-shrink 값이 1로 지정되어 있으면 width 값에 상관 없이 부모컨테이너에 항목을 넘어가더라도 flex item이 자동으로 정렬된다.

### flex-basis

flexbox 에서 width 값 대힌 사용하여 값을 적용한다.

```jsx
flex-basis 속성은 항목의 크기를 결정합니다. 
이 속성의 기본값은 auto이며, 이 경우 브라우저는 항목이 크기를 갖는지 확인합니다. 
항목의 크기가 100 픽셀이므로 flex-basis의 값으로 100 픽셀이 사용됩니다.
flex 항목에 크기가 지정되어 있지 않으면, flex 항목의 내용물 크기가 flex-basis 값으로 사용됩니다. 
따라서 flex 컨테이너에서 display: flex 속성만을 지정하면 flex항목들이 각 내용물 크기만큼 공간을 차지하게 됩니다.
```

### flex-grow & flex-shrink

flex-item 항목을 늘려주거나 줄여줄 때 사용하며

flex: flex-grow / flex-shrink / flex-basis 형태로 축약해서 사용이 가능하다

```jsx
flex-grow값을 양수로 지정하면 flex 항목별로 주축 방향 크기가 flex-basis 값 이상으로 늘어날 수 있게 됩니다. 
위의 사진 예시에서 모든 항목의 flex-grow 값을 1로 지정하면 사용가능한 공간은 각 항목에게 동일하게 분배되며, 
각 항목은 주축을 따라 분배받은 값만큼 사이즈를 늘려 공간을 차지합니다.

첫 항목의 flex-grow 값을 2로 지정하고 나머지 두 개의 항목을 1로 지정한다면 
각 항목에 지정된 flex-grow 값의 비율에 따라 남은 공간이 분배됩니다. 
각 항목의 flex-grow 비율이 2:1:1 이므로 첫 항목에게 100 픽셀, 
두 번째와 세 번째 항목에게 50 픽셀씩 분배됩니다.
```

```jsx
flex-grow 속성이 주축에서 남는 공간을 항목들에게 분배하는 방법을 결정한다면 
flex-shrink 속성은 주축의 공간이 부족할때 각 항목의 사이즈를 줄이는 방법을 정의합니다. 
만약 flex 컨테이너가 flex 항목을 모두 포함할 만큼 넉넉한 공간을 갖고 있지 않고 
flex-shrink 값이 양수이면 flex 항목은 flex-basis에 지정된 크기보다 작아집니다. 
또한, flex-grow 속성과 마찬가지로 더 큰 flex-shrink 값을 갖는 항목의 사이즈가 더 빨리 줄어듭니다.
```

### order

flex-item 항목 순서를 변경할 수 있다.

### flex-flow

`flex-direction`속성과 `flex-wrap`속성을 [`flex-flow`](https://developer.mozilla.org/ko/docs/Web/CSS/flex-flow)라는 축약 속성으로 합칠 수 있습니다. 첫 번째 값은 `flex-direction`이고 두 번째 값은 `flex-wrap` 이다.