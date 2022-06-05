---
layout: post
title: "Tailwind CSS 맛보기 후기"
author: "dongpark"
category: [post, main]
---

클론코딩 강의 메뉴중에 최근 핫하게(사실 대두된지는 좀 된걸로 기억..) 떠오르는 tailwind CSS 맛보기를 강의 내용 기준으로 4시간 정도 진행해 보았다.
소문대로 간결하고 부트스트랩급 간편함을 제공한다.

### CLASS 기반 스타일 컴포넌트

![](/assets/tailwindcss1.png)

``` 
<div className="py-4 px-2 space-y-2 border-b-2 last:border-b-0">
  <div className="w-full aspect-video bg-slate-300" />
  <div> Let's try potatos </div>
</div>
```

위와 같은 클래스 선언 으로도 실용성 있는 화면 구성이 가능 했다. 더 나아가 진성 디자인 고자인 나 조차도 직관 적이라 러닝 커브가 상당히 낮은편 이였다.

사실 지나가는 CSS 컴포넌트중 하나일거라 생각하고 가볍게 들은감이 많았었다. 실제로 실습해보고 나서는 생각이 상당히 바뀌었다.
당장 생각나는 장점만 해도 이미 클래스가 정의되어 있기 때문에 팀간 컨벤션을 맞추지 않고도 정형화된 화면 개발이 가능하다는 것?
나같은 백엔드 한량도 괜찮은 화면 개발이 가능하게 되었다는점? 앞으로도 클래스 기반 화면 개발이 잘 정착했으면 좋겠다.

### 부모 클래스 및 이벤트 컴포넌트 제공

```
className={
    cls(
                "flex flex-col items-center space-y-2 ",
                router.pathname === "/"
                  ? "text-orange-500"
                  : "hover:text-gray-500 transition-colors"
)}
```

위에서 보는 hover가 생각하는 그 hover가 맞다. 단순히 디자인 요소 컴포넌트를 클래스로 축약해서 제공해주는 것뿐만 아니라. 
XXX: 형식으로 이벤트나 실제 부모클래스를 상속해주는 기능을 제공해준다.

### JIT(Just in Time) 컴파일링

강의자 께서 강의 내내 강조했던 부분이였는데 사실 tailwind CSS는 하나의 큰 CSS 파일이라고 볼수 있지만,
사실 3.0 버전으로 넘어오면서 선언한 클래스들만 CSS 요소로 컴파일링 된다고 거듭 강조했다.
성능 기반으로 생각해도 아주 센세이션한 발전이다. 프론트 진영은 혁신주기가 점점 짧아지는것 같아 부럽기도 하다.
