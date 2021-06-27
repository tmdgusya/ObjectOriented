# 객체 지향적인 설계란?

**객체 지향적인 설계란 무엇일까?** 최근 **Domain Driven Design** 에 관한 프레임워크를 한번 사용해본뒤,
도메인 주도 설계에서는 **Domain 과 Context 의 흐름을 중심**으로 어플리케이션의 로직을 헤쳐나간다. 그렇담 
**객체지향은 무엇일까?** 아직은 초보 개발자의 생각을 Java 코드와 함께 적어보려고 한다.

- 우리가 작성해야할 **비즈니스 로직**은 아래와 같다
    1. **블로그의 포스트에는 댓글이 달릴 수 있다.**
    2. **블로그의 포스트에는 좋아요가 달릴 수 있다.**
    
그럼 위의 로직을 우리가 일상의 Context 로 가져와서 생각해보자! 

- **로치는 자신의 블로그를 킨 뒤, 로그인을 한다.** 로그인을 하자 블로그에서는 로치가 로그인을 했다는 사실을 알게 되었다.
어드민 권한을 검증받은 **로치는 블로그의 글 쓰기 버튼을 누른뒤, 글을 작성해 나간다.** 블로그에는 태그들과 글로 채워지고 하나의 포스팅으로 만들어진다.
  그 포스팅에 노을이 들어와서 댓글과 좋아요를 누르자. 포스트에는 댓글과 좋아요가 1개씩 추가 되었다.
  
자 그럼 위의 Context 의 흐름을 하나하나 분리해보자

    - 유저는 블로그에 로그인을 한다
    - 블로그에서는 로치가 로그인한 사실을 안다.
    - 유저는 어드민 권한을 검증받는다.
    - 어드민 권한이라면 블로그에서 글쓰기 기능이 가능하다.
    - 어드민 권한인 로치는 블로그에 글 쓰기 버튼을 누른다.
    - 로치는 포스트에 글을 작성한다.
    - 포스트에는 글이 작성되고, 블로그에 저장된다.
    - 다른 유저가 로그인 한다.
    - 블로그는 다른 유저가 로그인 한 사실을 안다.
    - 다른 유저는 로치의 포스트의 댓글을 단다.
    - 로치의 포스트에는 댓글이 달린다.
    - 다른 유저는 로치의 포스트에 좋아요를 누른다.
    - 로치의 포스트에는 좋아요가 달린다.

위의 흐름을 보았을때 로그인 하는 과정은 하나의 로직으로 단순하게는 아래코드 처럼 적어볼 수 있을것이다.

## 권한 검증

```java

UserRole role = Roach.getRole()

if(role === UserRole.ADMIN) {
    can Write!!
} else {
    😓Nope!!    
}

```

객체 지향적인 설계에서 위의 로직은 맞는 로직일까? 우리가 절차적으로 생각했을때 맞다는 생각이 들 수도 있다.
**저렇게 설계된 어플리케이션은 점점 저러한 분기문이 많아질테고, 우리는 코드를 유지보수하기가 힘들어 질것이다.**
또한 현재 User 객체는 하나의 Context 를 통해서 **자신의 관리자인지 아닌지를 수동적으로 판단되고 있다.**
객체 지향에서의 가장 중요한 관점은 **객체에게 할당된 권한은 객체에게 완전히 위임**하는 것이다. 그렇다면 위의 로직을
객체 지향 스럽게 바꿔보자

```java

if(Roach.isAdmin()) {
    //Can Write!!    
} else {
    //Nope!!    
}

```

자 위의 코드에 비해 훨씬 명확하고, **Roach 라는 User 의 Object 가 자신스스로 권한을 검증하고 있다.**
유저가 검증한 뒤 도출해주는 메세지를 통해서 우리는 **비즈니스로직에서 하나의 Context 를 생성해내고 있다.**
이 정도 예제라면 간단한 이해는 갔을꺼라 생각하고 다른 부분으로 넘어가 보자!

## 댓글 쓰기

댓글은 게시물에 적히는 객체 중 하나이다. 우리의 요구사항을 한번 이쯤에서 리마인드 시킬겸 객체 지향적인 시점으로 다시 분석해보자

1. 하나의 게시물에 댓글을 달 수 있다.
    - **게시물은 여러가지 댓글을 가질 수 있다.**
    - **댓글은 유저에 의해 적힌다.**
    - **댓글은 댓글의 내용을 저장할 수 있다.**
    - **댓글은 작성자를 기억할 수 있다.**
    
```java

Post RoachPost;
User Noeul;
Comment newComment;

```

일단 하나의 로직안에 Object 들이 위와 같이 존재할 수 있다. 아까 위에서 봤듯이 객체지향적인 설계는 
**객체들의 역할에 대한 책임을 다하고, 서로가 메세지를 주고 받으며 이어진다.** 우리는 어떻게 저 요구 사항을 
객체 지향적으로 수립할 수 있을까?

```java

Comment writeByNoeul = Nouel.writeComment(new Comment("댓글은 어디에 있어야 할까?", Noeul))

```

노을은 댓글을 적을 수 있다. **노을은 댓글을 적어서 자신이 어떤 내용을 적었음을 댓글 객체에게 알려준다.**
댓글 객체는 **Noeul 이 이를 적었음을 알고, 내용또한 저장하게 된다.**
이제 내용과 작성자는 댓글 객체의 바운더리 안에 들어왔다.
그럼 댓글은 Post 에 저장되야 할 것이다. **하지만 Post 에 댓글이 마음대로 붙는건 객체지향 설계에 원칙이 아니다.**
**Post 안의 댓글은 Post 가 저장한다.** 따라서 아래와 같은 로직으로 확장될것이다.


```java

Comment commentWriteByNoeul = Nouel.writeComment(new Comment("댓글은 어디에 있어야 할까?", Noeul))

PostA.addComment(commentWriteByNoeul);
```

위의 로직대로 봤을때, 유저는 댓글을 적을 수 있지만, **댓글안의 내용을 적을 수 있는건 오직 댓글이 제공하는 생성자를 통해서**이다.
만약 댓글이 **유저와 메세징을 주고 받기 싫어 생성자를 private 로 바꾼다면, 유저는 댓글을 달 수 없다.**
이로써 **댓글은 유저와 소통하지만, 자신만의 관리 Pool 을 지닌 하나의 Object 이다.**
Comment 는 게시물에 바로 붙어야 할것 같지만, **자신이 저장 할 Comment 를 관리하는 것또한 Post 의 역할**이다. 
만약 아래와 같은 요구사항이 하나 생긴다고 해보자! 우리는 객체 지향적으로 어떻게 풀어낼 수 있을까?

- **이제부턴 게시물엔 "안경" 이라는 키워드의 댓글은 못달게 해주세요!**

자 이렇게 되면, Comment 객체와 Post 객체는 어떻게 함께 일해야 할까?
이걸 **검사하는 건 Comment 일까 Post 일까 한번 잘 생각해보자.** 만약 당신이 Comment 라고 생각해보자
User 가 글을 남길건 저장했두었는데, 갑자기 **Post 라는 검사관이 찾아와서 당신의 머리속에 안경이 있는지 훑어낸다.**
안경이 있다면, **Post 검사관이 당신에게 빨간딱지를 붙이며 앞으로 Post 에는 출입금지**입니다. 라는 스티커를 붙인다면 
좋겠는가? **이는 객체의 자율성을 침범할 수 있는 행위다.** 즉 위의 Context 를 코드로 짜면 아래와 같이 나올것이다.

```java

public void dont_add_안경쓴_댓글(comment) {
    if(comment.getContent().inClude("안경")) {
        return "안경쓴 댓글은 오지마!!!"    
    }
    commentList.add(comment)
}

```

이 보다는 Comment 가 자율적으로 자신이 안경을 썼는지 안썼는지 판단하고, 들어가지 않는 행위를 택해보자

```java

Comment comment = User.writeComment(new Comment("안경은 다빈치안경!", user))

if(comment.isInclude안경()) {
    throw new Exception("안경이라는 키워는 쓰지마세요!")    
} 

```

**Comment 가 자신이 안경을 가지고 있는지 판단하고, 안경이 들어있다면 알려준다** 
따라서 **Comment 에는 이제 안경이 들어올 수 없으며**, 유저 또한 경고문을 보고 댓글을 달때 안경을 쓰지 않도록
조심하게 되었다.

그리고 **Post 는 이제 자신이 관리하는 댓글들이 안경이 있는지를 검사할 차례**가 왔다.
하지만 ***Post 가 마음대로 댓글의 내용을 여는것이 아닌, 댓글이 자기 자신이 안경이 아님을 Post 에게 알려준다.**
따라서 Post 는 Comment 가 안경이 아님을 믿고, 안경이 적히지 않은 댓글들만 가져온다.

```java

public void delete_안경_댓글() {
    this.commentList.stream().filter(comment::isNot안경)
}

```


우리가 **절차지향적인 로직으로 이를 가져간다면, 유저의 content 를 검사하고 댓글을 만들고 이러한 과정들이
하나의 로직안에 담겨버릴 것**이다. 하지만 그렇게 되면, **같은 의미의 로직이 중복될 수 있고, 유지보수에 힘들어 질 수 있다.**
따라서 우리는 이를 **객체가 가져야할 권한이 아닌지를 분리**하고, **객체가 가져야할 권한이라면 객체에게 위임**한 뒤
**객체의 메세지로 하나의 로직에서의 Context 를 이어나가는 형태로 설계**한다. 이것이 객체지향 설계 방법이 아닐까?
