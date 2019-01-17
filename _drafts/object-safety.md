---
layout: post
title:  "Object safety에 대하여"
tags:   ["Rust"]
published: true
---

Rust에서 거의 모든 메서드 호출은 정적 디스패치로 이루어지는데, 다시 말해 어떤 코드를 실행할지를 컴파일 단계에서 대부분 결정한다는 뜻입니다.

하지만 어떤 메서드를 호출해야 하는지를 컴파일할 때는 알 수 없는 경우가 있습니다.

```rust

```

`dyn Trait`과 같은 타입의 값을 trait object라고 부릅니다

다른 언어에서는 보통 모든 객체가 동적 디스패치를 위한 정보를 값에 함께 달고 다니게 됩니다. C++처럼 [가상 테이블][] 같은 걸 만들거나, 또는 객체를 만들 때 썼던 클래스나 프로토타입에 대한 참조를 가지고 있다거나 하는 식으로요. 반면 러스트에서는 trait object만 동적 디스패치를 위한 정보를 포함하게 됩니다.

문제는 모든 트레이트로 트레이트 객체를 만들 수 있는 게 아니라는 점입니다.

----

트레이트 객체를 만들기 위해 필요한 조건을 object safety라고 부릅니다. 대충 요약하면 다음과 같습니다.

 - 트레이트의 모든 메서드는 `self`를 첫 번째 인자에 포함해야 합니다.
 - 자기 자신의 크기를 알아야 하는 타입이 메서드 인자나 리턴 타입에 있어선 안 됩니다.

요점은 구체적인 타입 정보를 실행 중에 아예 몰라도 동작할 수 있어야 object safe하다는 것입니다.

----

함수나 모듈 한두 개 정도의 작은 규모에서는 별다른 차이를 느끼지 않겠지만, 좀 더 큰 규모의 소프트웨어를 설계하게 되면 전통적인 객체 지향 디자인을 쓰기 힘들게 됩니다.

 - 트레이트 객체를 다시 원래의 값으로 바꿀 수 없습니다. 트레이트 객체가 되는 시점에서 원래 타입에 대한 정보를 모두 잃어버리기 때문입니다.
 - 제네릭에서 `T: A + B`처럼 둘 이상의 트레이트를 지정할 수 있는 것과는 달리, `dyn A + B`처럼 둘 이상의 트레이트를 요구할 수 없습니다. 단, `Send`, `Sync`와 같은 자동 트레이트(auto trait) 및 `'static`과 같은 lifetime 변수는 예외입니다.

다만 다운캐스트가 필요한 상황이라면 아예 [Any 트레이트][10]를 쓰는 방법은 마련되어 있습니다.

[10]: https://doc.rust-lang.org/std/any/index.html