[TOC]

---

## 1.2 DAO의 분리 :memo:

### - 관심사의 분리

- 관심이 한 군데에 집중되게 하는 것이 중요하다

  > 관심이 같은 것끼리 모으고, 관심이 다른 것은 따로 떨어져 있게 하는 것.

- 관심이 같은 것끼리 하나의 객체 안으로, 관심이 다른 것은 떨어져 서로 영향을 주지 않도록 분리



### - 커넥션 만들기의 추출

**UserDao의 관심사항**

1. DB와 연결을 위한 `Connection` 오브젝트 가져오기
2. `Statement`를 만들고 실행
3. 자원정리(사용한 리소스 닫기)

> add()라는 메소드 하나에서 세 가지 관심사항을 발견..! => :boom: **초난감상태!!** :boom:
>
> *** 예외처리에 관한 내용은 후에 다룰 것임



**중복 코드의 메소드 추출**

:heavy_check_mark: 중복된 코드 분리하기

`getConnection()`  독립적 메소드 만들기

> 각 메소드에서 분리한 `getConnection()` 메소드를 호출해서 커넥션 연결!

특정 관심사항이 담긴 코드 분리 --> 조금 난감한 DAO 만들기 성공!

> 기능에 영향 없이 구조만 변경 : 리팩토링[^refactoring]
>
> 공통 기능 담당 메소드 분리 : 메소드 추출[^extract method]

**리팩토링 :question:**

> 기존의 코드를 외부의 동작방식에 변화없이 내부 구조를 변경해서 재구성하는 작업 or 기술

예제코드 : 63p



### - DB 커넥션 만들기의 독립

**상속을 통한 확장**

**만약, DB 커넥션을 독자적으로 적용하고 싶다면?**

- UserDao에서 메소드의 구현 코드 제거

- `getConnection()` 을 추상 메소드로 

:point_right: DB 커넥션을 상속을 통해 서브 클래스로 분리

**UserDao.java**

```java
public void add(User user){
    Connection c = getConnection();
    /* add 구현 코드*/
}
public void get(User user){
    Connection c = getConnection();
    /* get 구현 코드*/
}
public abstract Connection getConnection(); // 추상메소드
```

> N사와 D사는 각각 `NUserDao`와 `DUserDao`에서 상속을 통해 확장(오버라이딩)
>
> -> Dao코드 수정 없이 DB연결기능을 새롭게 정의한 클래스 만들기 가능



### **디자인 패턴 :closed_book:**

#### 템플릿 메소드 패턴[^template method pattern]

> 슈퍼클래스에서 기본 로직의 흐름을 만듦
>
> 그 기능의 일부를 추상메소드, 오버라이딩 가능 메소드로 만듦
>
> 서브클래스에서 이 메소드를 필요에 맞게 구현

**UserDao.java**

```java
public abstract class UserDao {
    public User get(String email, String password) { ... }
    public int insert(User user) { ... }
    public abstract Connection getConnection() throws SQLException;
}
```

**CUserDao.java**

```java
public class CUserDao extends UserDao {
    	@Override
	public Connection getConnection() throws SQLException { 
        connection ...
    }
}
```

---

#### 팩토리 메소드 패턴[^factory method pattern]

> 서브클래스에서 구체적인 오브젝트 생성 방법을 결정

![팩토리 메소드 패턴](https://dhsim86.github.io/static/assets/img/blog/web/2017-06-06-toby_spring_01_object_dependency/01.png)

**DBConnection.java**

```java
public interface DBConnection {
	Connection getConnection() throws SQLException;
}
```

**MySQLConnection.java**

```java
public class MySQLConnection implements DBConnection {
	@Override
	public Connection getConnection() throws SQLException {
		connection 구현
}
```

**UserDao.java**

```java
public abstract DBConnection getDBConnection() throws SQLException;
```

**CUserDao.java**

```java
public class CUserDao extends UserDao {
	@Override
	public DBConnection getDBConnection() throws SQLException {
		return new MySQLConnection();
	}
}
```



위 두 메소드 단점

1. 상속사용 : 다른 목적으로 상속 적용 힘듦

2. 상하위 클래스의 밀접한 관계 : 슈퍼클래스 수정시 서브클래스 수정 or 슈퍼클래스의 변화 제약

3. 확장된 DB 커넥션 코드를 다른 DAO에 적용 불가능

   > `getConnection()` 메소드가 매 클래스마다 중복됨
