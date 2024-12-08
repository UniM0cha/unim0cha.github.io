---
layout: post
title: 개발을 진행하면서 정한 나만의 Spring Boot 규칙들 (1. JPA 편)
categories:
  - 개발
  - 나만의 Spring Boot 규칙
tags:
  - Java
  - Spring boot
image: /assets/img/posts/5380b89e3e00cded741001488af5df12ad46f949408ae3d848ea6f10e7624fd5.jpeg
date: 2024-12-08 10:57 +0900
---

Spring Boot를 사용하여 개발을 진행하다보니 점점  
'이 정도는 아예 규칙으로 정해놓고 써야겠다!' 싶을 정도로  
휴먼 에러가 많이 발생하는 경우가 많았습니다.  
저 또한 그랬기에, 한번 정리하는 마음으로 휴먼 에러를 최대한 방지하기 위한  
저만의 Spring Boot 규칙과 그 이유에 대해서 설명해보고자 합니다.

## JPA 편

### 1. 엔티티 생성 시 `@AllArgsConstructor`보다 생성자를 직접 작성

`deletedAt 필드가 null이 아니면 삭제되었다`를 사용하여  
soft delete를 수행하는 간단한 Member 엔티티 입니다.

```java
@Entity
@Getter
@Table(name = "member")
@SQLDelete(sql = "UPDATE member SET deleted_at = current_timestamp WHERE id = ?")
@AllAgrsConstructor
public class Member extends UuidEntity {

  @Comment("삭제 일자")
  private LocalDateTime deletedAt;

  @Comment("이메일")
  @Column(nullable = false)
  private String email;

  @Comment("로그인 비밀번호")
  @Column(nullable = false)
  private String password;
}
```

이곳에서 깔끔한 코드로 객체를 생성하기 위해 간단하게 @AllArgsConstructor를 붙여주게 된다면  
deletedAt 필드는 엔티티 생명주기에 따라서 JPA가 직접 관리할 필드이지만,  
사용자가 직접 수정할 여지가 생기게 됩니다.

```java
Member member = new Member(/*deletedAt*/null , "이메일", "비밀번호");
```

그리고 이런 경우는 수없이 많이 생기죠. (ID, createdAt, createdBy, updatedAt, updatedBy...)  
따라서 엔티티의 경우엔 원하는 필드만 초기화 해주기 위해서 따로 생성자를 만들어주는 것이 좋았습니다.

```java
@Entity
@Getter
@Table(name = "member")
@SQLDelete(sql = "UPDATE member SET deleted_at = current_timestamp WHERE id = ?")
public class Member extends UuidEntity {

  // 나머지...

  public Member(String email, String password) {
    this.email = email;
    this.password = password;
  }
}
```

```java
Member member = new Member("이메일", "비밀번호");
```

### 2. 엔티티의 기본 생성자의 공개 범위를 Protected로 두어 직접 사용할 수 없도록 함

아까의 Member 엔티티입니다.

```java
@Entity
@Getter
@Table(name = "member")
public class Member extends UuidEntity {

  @Comment("이메일")
  @Column(nullable = false)
  private String email;

  @Comment("로그인 비밀번호")
  @Column(nullable = false)
  private String password;
}
```

사실 이 상태에서는 Spring Boot가 실행되지 않습니다.  
엔티티에는 기본 생성자가 꼭 필요합니다.  
그래서 보통은 `@NoArgsConstructor`를 붙여줍니다.

```java
@Entity
@Getter
@Table(name = "member")
@NoArgsConstructor
public class Member extends UuidEntity {
  // 나머지...
}
```

하지만 이러면 어느 곳에서든지 기본 생성자를 사용할 수 있게되죠.  
따로 규칙을 명시해두지 않았다면 다른 개발자분이 보고 @Setter를 붙이기 딱 좋은 모습입니다.  
<small style="opacity: 0.5;">Setter는 유명한 안티패턴이라는거 아시죠?</small>

```java
Member member = new Member();
member.setEmail("이메일");
```

따라서 특별한 경우가 아니면 `@NoArgsConstructor`에 `AccessLevel.PROTECTED`를 붙여줍니다.  
(private은 JPA도 사용 못합니다.)  
코드 자체에서 Setter가 아닌 생성자를 사용하도록 유도할 수 있습니다.

```java
@Entity
@Getter
@Table(name = "member")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member extends UuidEntity {
  // 나머지...
}
```

### 3. 양방향 매핑은 최대한 지양

객체간 순환참조는 개발을 하는 데에 있어서 악입니다. <small>진짜로.</small>  
그리고 엔티티의 양방향 매핑은 순환참조를 만들기에 딱 좋고, 예상치 못한 동작을 만들게 됩니다.

아무리 연관관계 편의 메서드를 사용한다고 하더라도  
엔티티간 멤버 필드의 불일치가 존재하기 마련입니다.

특히 **삭제가 아닌 연관관계를 해제**할 때 발생합니다.  
제가 경험했던 현상을 간단하게 보여드리겠습니다.

아래는 Client(고객사, 일)와 Member(고객, 다)를 양방향 매핑으로 연결한 예시입니다.

```java
@Entity
@Table(name = "member")
public class Member extends UuidEntity {

  @Comment("이메일")
  private String email;

  @Comment("로그인 비밀번호")
  private String password;

  @Comment("고객사")
  @ManyToOne(fetch = FetchType.LAZY)
  private Client client;
}
```

```java
@Entity
@Table(name = "client")
public class Client extends BaseEntity {

  @Comment("이름")
  private String name;

  @OneToMany(mappedBy = "client", orphanRemoval = true, cascade = CascadeType.ALL)
  private List<Member> members;
}
```

자동 영속화를 위해 `CascadeType.ALL`과 간편한 삭제를 위해 `orphanRemoval`을 주었습니다.  
<small style="opacity: 0.5;">(이거에 대해서는 내용이 많이 복잡합니다! 잘 모르겠으면 일단 넘어가주세요.)</small>

1번 멤버를 A고객사에서 B고객사로 변경하려고 합니다.  
그러면 Member에서는 Client B를 조회한 후 client 필드에 B를 할당해주기만 하면 됩니다.

```java
member.changeClient(clientB);
```

그리고 양방향 매핑시에는 Client 쪽에서는 멤버 필드의 무결성을 위해  
**Client A의 List에서는 Member를 제거하고**  
Client B의 List에서는 Member를 더해주어야 합니다.

```java
clientA.removeMember(member);
clientB.addMember(member);
```

이때 Client A의 List에서 Member를 제거하는 동작이,  
orphanRemoval에 의해서 "이동"이 아닌 "삭제"로 판단될 때가 있습니다.  
(물론 영속화 과정을 꿰고 있다면 괜찮겠지만 조금만 복잡해져도 알기가 어렵습니다.)  
이동만 하려 했는데 삭제가 되어버리는 것이죠.

그리고 이 현상은 컴파일 시점에 잡아낼 수도 없으며, 예측하기 어려울 뿐만 아니라,  
심지어 데이터베이스를 사용하지 않는 단위 테스트에서도 잡아내기 어렵습니다.

물론 실무에서는 이 예시 말고도 예측하기 어려운 상황은 더 많습니다.  
따라서 저는 특히나 양방향 매핑을 사용하지 않으려고 합니다.  
**데이터베이스의 구조와 엔티티의 구조가 가장 유사할 때** 오류를 최소화 할 수 있다고 생각합니다.

> "그럼 쿼리 메서드와 엔티티를 통한 간편한 조회가 힘들텐데요!"

하실 수도 있습니다. 이해합니다.  
실제로 양방향 매핑을 사용하다가 단방향으로 전환하면 꽤 귀찮습니다.  
구체적으로 이런게 안됩니다.

```java
// Client의 모든 Member 가져오기
List<Member> members = client.getMembers();
```

```java
public interface ClientRepository extends JpaRepository<Client, Long> {
  // Client의 Member 중 특정 이메일을 가진 Member가 존재하는 Client 찾기
  Optional<Client> findByMembers_Email(String email);
}
```

하지만 모두 Repository를 사용해서 대체할 수 있습니다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  List<Member> findAllByClient(Client client);
}

// Client의 모든 Member 가져오기
List<Member> members = memberRepository.finAllByClient(client);
```

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  Member findByEmail(String email);
}

// Client의 Member 중 특정 이메일을 가진 Member가 존재하는 Client 찾기
Member member = memberRepository.findByEmail("이메일");
Client client = member.getClient();
```

물론 Repository로만 처리하기엔 복잡한 상황이 있을 수 있습니다.

따라서 저는 간단한 쿼리문을 작성할 때에는 쿼리메서드를 사용하고,  
검색과 같이 조건이 많은 복잡한 쿼리가 필요할 때는  
고민 말고 JPQL 또는 QueryDSL을 사용하여 쿼리메서드를 만듭니다.  
그 중에서도 JPQL을 권장합니다.

QueryDSL은 아직 빌드도구 간 연동 오류가 많이 발생합니다.  
웬만하면 JPQL로 될텐데, 정 안된다면 Native Query를 사용합니다.  
QueryDSL로 되는거면 JPQL로도 될 것입니다.  
오히려 QueryDSL 문법을 학습하기보단 SQL과 구조가 비슷한 JPQL이 더 학습에 도움이 될 것입니다.

---

원래는 한 게시물에 모두 적으려 했으나, JPA만 해도 꽤 길어졌네요.😅  
다음 게시물에서 Spring boot에 대한 규칙도 정리해보겠습니다.
