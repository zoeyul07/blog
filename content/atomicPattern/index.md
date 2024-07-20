---
emoji: ''
title: 'atomic pattern'
date: '2023-04-05 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

## atomic design

1. Atoms

- UI의 기본적인 블록으로서, 버튼, 텍스트, 아이콘 등과 같은 작은 컴포넌트를 의미한다.

1. Molecules

- Atom들을 결합한 것으로서, 검색창이나 로그인 폼 등과 같은 상대적으로 복잡한 컴포넌트를 의미한다.

1. Organisms

- Molecule들과 Atom들을 결합한 것으로서, 헤더나 푸터 등과 같은 전체 페이지의 레이아웃을 의미한다,.

1. Templates

- 페이지 레이아웃을 정의하는데 사용되는, 여러 개의 Organisms와 Molecules를 결합한 것을 의미한다.

1. Pages

- 실제로 사용자가 볼 수 있는 웹 페이지를 의미합니다. Template을 사용하여 구성되며, 필요한 데이터와 기능을 갖추고 있다.

### **Molecule과 Organism 을 나누는 기준의 주관성**

molecule은 atom으로 구성되어 SRP에 따라 1가지 책임을 지고, organism은 atom, molecule, organism으로 구성되어 서비스에서 Layout을 기준으로 나눌 수 있는 영역을 갖는다.

작성한 컴포넌트에 컨텍스트가 있는 경우에는 organism으로, 컨텍스트 없이 UI 적인 요소로 SRP를 지킬 수 있다면 재사용하기 쉬운 molecule로 작성한다. 그래서 molecule의 컴포넌트 네이밍은 컨텍스트 없이 주로 UI 적인 네이밍이 포함된다. 반면에 organism 네이밍은 컨텍스트가 포함된다.

### **Organism을 나누는 기준**

organism은 UI에서 명확한 영역을 갖는다. 이 명확한 영역에 대한 정의는 주관적이라 여러 가지 방식으로 해석될 수 있다.

극단적으로는 큰 영역을 organism이라고 할 수 있고, 비슷한 유형의 책임을 그루핑하여 구분할 수 있다. 

공통된 컨텍스트를 묶어 organism 컴포넌트로 표현하면 적당한 책임을 가진 컴포넌트를 작성할 수 있다. 

## Foundation, Elements, Modules and the Prototype

Here is the high-level overview of how I like to structure and implement the individual parts:

- **Foundation**: This is the basic layer of design tokens such as colors, typography, spacings, iconography and their like. Basically the non-component basics I have my trouble with when defined and categorized as atoms. Assigning them an explicit category and naming it foundation makes clear that this affects every piece of the system.
- **Elements**: The “basic building block” components everyone thinks of when talking about atoms. Concretely they map to customized implementations of single HTML elements, like headings and buttons. But also these kinds of elements, that do not make any sense in HTML when used standalone, like list items. In this case a list would be the most non-dividable form and hence the element.
- **Modules**: Everything that can contain other components. I am defining components as a collective term for elements or modules – the distinction being whether or not they can contain other parts. In atomic design terms this is the group of molecules and organisms, avoiding the strict categorization. As you might have guessed, this distinction also does not find its expression in the file system: There is no folder hierarchy, just a single flat components directory which contains a single folder for each component.
- **Prototype**: In the end product people do not want design tokens, components or templates. They want concrete pages which should be assembled of all these parts. The prototype is our section in the pattern library/design system documentation where everything comes together. Here the templates containing the components are married with the (sample) data to form pages everyone understands. The prototype is also the basis for testing and validating ideas and features.

### Reference

[아토믹 디자인을 활용한 디자인 시스템 도입기 | 카카오엔터테인먼트 FE 기술블로그](https://fe-developers.kakaoent.com/2022/220505-how-page-part-use-atomic-design-system/)

[Effective Atomic Design](https://kciter.so/posts/effective-atomic-design)

[next.js 폴더나 컴포넌트 구조를 어떻게 짜시나요?](https://careerly.co.kr/qnas/594)

[Stop Using Atomic Design Pattern](https://jbee.io/react/stop-using-atomic-design/)

[Atomic Design Methodology | Atomic Design by Brad Frost](https://atomicdesign.bradfrost.com/chapter-2/)

[The Atomic Workflow | Atomic Design by Brad Frost](https://atomicdesign.bradfrost.com/chapter-4/#step-2-prepare-for-screenshotting)

[Atomic Design is messy, here's what I prefer](https://dennisreimann.de/articles/atomic-design-is-messy.html)

[The src folder in NextJs - Js Craft](https://www.js-craft.io/blog/the-src-folder-in-nextjs/)

[합성 컴포넌트로 재사용성 극대화하기 | 카카오엔터테인먼트 FE 기술블로그](https://fe-developers.kakaoent.com/2022/220731-composition-component/)

[Create A Design System with Storybook, Typescript, and Next.js](https://roewynumayam.com/blog/create-a-design-system-with-storybook-typescript-nextjs)
