---
title: MVC, MVP, MVVM 패턴 비교
date: 2024-01-11 23:30:00 +09:00
categories: [java, Design Pattern]
tags:
  [
    java, 
    자바, 
    DegisnPattern,
    디자인패턴
  ]
---

* * *

## MVC(Model-View-Controller) 패턴
* `모델(Model)`: 애플리케이션의 데이터와 비즈니스 로직을 담당한다.
* `뷰(View)`: 사용자에게 데이터를 보여주는 역할, 사용자 인터페이스를 표현하고 모델의 정보를 표시한다. 
* `컨트롤러(Controller)`: 모델과 뷰 사이의 상호 작용을 관리, 뷰에서 데이터를 받아 모델에게 전달한다.

뷰가 모델을 의존한다는 단점이있다.

## MVP(Model-View-Presenter) 패턴
* `모델(Model)`: 애플리케이션의 데이터와 비즈니스 로직을 담당한다.
* `뷰(View)`: 사용자에게 데이터를 보여주는 역할, 사용자 인터페이스를 표현하고 모델의 정보를 표시한다.
* `프리젠터(Presenter)`: 컨트롤러의 역할을 대신하며, 사용자 입력에 반응하여 모델을 업데이트하고 뷰를 갱신한다.

MVP 패턴은 뷰와 모델 사이의 의존성을 제거하여 더 효율적인 테스트와 유지 보수를 가능하게 한다.


## MVVM (Model-View-ViewModel) 패턴:
* `모델(Model)`: 애플리케이션의 데이터와 비즈니스 로직을 담당한다.
* `뷰(View)`: 사용자에게 데이터를 보여주는 역할, 사용자 인터페이스를 표현하고 모델의 정보를 표시한다.
* `뷰 모델(ViewModel)`: 사용자 인터페이스를 표현하는 데 필요한 데이터와 명령을 캡슐화한다.  
  뷰 모델은 뷰에 대한 데이터 바인딩을 관리하고 뷰와 모델 사이의 중간자 역할을 한다.

MVVM 패턴은 데이터 바인딩을 통해 뷰와 뷰 모델을 연결하여 뷰 모델의 변화를 자동으로 반영하고,   
뷰와 모델 간의 의존성을 제거한다. 이는 코드의 중복을 줄이고 유지 보수성을 향상시킨다.


## MVC vs MVP
MVC에서는 컨트롤러가 사용자 입력을 받고 모델을 업데이트하며, 그 결과를 뷰에 반영한다.   
그러나 MVP에서는 프리젠터가 사용자 입력을 받고 모델을 업데이트한 다음, 뷰를 갱신한다.  
이는 MVC와 달리 뷰가 모델에 직접 접근하지 않고 프리젠터를 통해 상호작용하므로, 테스트 용이성이 좋아진다.  



