# [JPA - Best Practice] Step-01 Account 생성, 조회, 수정 API 간단히 만들기



> 참고: https://github.com/cheese10yun/spring-jpa-best-practices/blob/master/doc/step-01.md





### 중요 포인트

- 도메인 클래스 작성
- DTO 클래스를 이용한 Request, Response
- Setter 사용 안하기





## 1. 도메인 클래스 작성: Account Domain



**Account 클래스**

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Account {

    @Id @GeneratedValue
    private long id;

    @Column(name = "email", nullable = false, unique = true)
    private String email;

    @Column(name = "first_name", nullable = false)
    private String fistName;

    @Column(name = "last_name", nullable = false)
    private String lastName;

    @Column(name = "password", nullable = false)
    private String password;

    @Column(name = "address1", nullable = false)
    private String address1;

    @Column(name = "address2", nullable = false)
    private String address2;

    @Column(name = "zip", nullable = false)
    private String zip;

    @Column(name = "created_at")
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    @Column(name = "updated_at")
    @Temporal(TemporalType.TIMESTAMP)
    private Date updatedAt;

    @Builder
    public Account(String email, String fistName, String lastName, String password, String address1, String address2, String zip) {
        this.email = email;
        this.fistName = fistName;
        this.lastName = lastName;
        this.password = password;
        this.address1 = address1;
        this.address2 = address2;
        this.zip = zip;
    }

    public void updateMyAccount(AccountDto.MyAccountReq dto) {
        this.address1 = dto.getAddress1();
        this.address2 = dto.getAddress2();
        this.zip = dto.getZip();
    }
}
```



### (1) 제약 조건 맞추기

- 칼럼에 대한 제약조건을 생각하며 작성하는 것이 바람직함.
- `nullable`, `unique` 조건 등 해당 DB의 schema와 동일하게 설정하는 것이 좋다.



### (2) 생성 날짜, 수정 날짜 값 설정 못하게 하기

- 일단, setter 메서드를 모든 멤버 필드에 대해서 설정하지 않음.
- Builder에도 생성&middot;수정 날짜를 제외한다.

- `@CreationTimestamp`나 `@UpdateTimestamp` 애너테이션을 이용해서 날짜를 자동으로 입력하게 하거나, DB에서 자동으로 입력하게 설정하는 편이 좋다.



### (3) 객체 생성 제약

- `@NoArgsConstructor(access = AccessLevel.PROTECTED)` 애너테이션을 통해 객체를 기본 생성자로 만들지 못하게 함.
- 대신 `@Builder` 애너테이션을 통해 객체를 생성하게 강제함.





## 2. DTO 클래스를 이용한 Request, Response



**Account DTO 클래스**

```java
public class AccountDto {

    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class SignUpReq {
        private String email;
        private String fistName;
        private String lastName;
        private String password;
        private String address1;
        private String address2;
        private String zip;

        @Builder
        public SignUpReq(String email, String fistName, String lastName, String password, String address1, String address2, String zip) {
            this.email = email;
            this.fistName = fistName;
            this.lastName = lastName;
            this.password = password;
            this.address1 = address1;
            this.address2 = address2;
            this.zip = zip;
        }

        public Account toEntity() {
            return Account.builder()
                    .email(this.email)
                    .fistName(this.fistName)
                    .lastName(this.lastName)
                    .password(this.password)
                    .address1(this.address1)
                    .address2(this.address2)
                    .zip(this.zip)
                    .build();
        }

    }

    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class MyAccountReq {
        private String address1;
        private String address2;
        private String zip;

        @Builder
        public MyAccountReq(String address1, String address2, String zip) {
            this.address1 = address1;
            this.address2 = address2;
            this.zip = zip;
        }

    }

    @Getter
    public static class Res {
        private String email;
        private String fistName;
        private String lastName;
        private String address1;
        private String address2;
        private String zip;

        public Res(Account account) {
            this.email = account.getEmail();
            this.fistName = account.getFistName();
            this.lastName = account.getLastName();
            this.address1 = account.getAddress1();
            this.address2 = account.getAddress2();
            this.zip = account.getZip();
        }
    }
}
```



### (1) DTO 클래스의 필요성

**API에 직접 엔티티를 노출시키는 경우 생기는 문제점**

- 데이터 안정성

  - [Request를 엔티티로 받는 경우 문제점]

    정보 변경 API에서는 firstName, lastName 두 속성만 변경할 수 있다고 하자.

    그런데 만약 Account 클래스를 RequestBody로 받게 된다면 

    email, password 등 Account 클래스의 모든 속성값들을 컨트롤러를 통해 넘겨받게 되고 

    <u>예상치 못한 데이터 변경</u>이 발생할 수 있다.

  - [Response를 엔티티로 나타내는 경우 문제점]

    Response 타입이 Account 클래스일 경우 계정의 모든 정보가 노출되게 된다.

    JsonIgnore 속성들을 두어 막을 수도 있지만 바람직하지 않은 방법이다.

- 엔티티가 변경될 경우 API 스펙이 바뀐다.



**DTO 클래스를 통해 요청을 받고 응답을 하면 생기는 장점**

- 요구사항이 명확해짐.

  - 위의 MyAccountReq 클래스는 계정 마이 페이지에서 변경할 수 있는 값들로 address1, address2, zip 속성이 있다.

    요구 사항이 이 세 가지 속성에 대한 변경이어서 해당 API가 어떤 값들을 변경할 수 있는지가 명확해진다.





### (2) 컨트롤러에서의 DTO

```java
@RestController
@RequestMapping("accounts")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @RequestMapping(method = RequestMethod.POST)
    @ResponseStatus(value = HttpStatus.CREATED)
    public AccountDto.Res signUp(@RequestBody final AccountDto.SignUpReq dto) {
        return new AccountDto.Res(accountService.create(dto));
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    @ResponseStatus(value = HttpStatus.OK)
    public AccountDto.Res getUser(@PathVariable final long id) {
        return new AccountDto.Res(accountService.findById(id));
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    @ResponseStatus(value = HttpStatus.OK)
    public AccountDto.Res updateMyAccount(@PathVariable final long id, @RequestBody final AccountDto.MyAccountReq dto) {
        return new AccountDto.Res(accountService.updateMyAccount(id, dto));
    }
}
```



![](https://camo.githubusercontent.com/faa44ccc032224cfa3bfbfd904635abb4214b628f955b680bbf8067751e13696/68747470733a2f2f692e696d6775722e636f6d2f706268647063562e706e67)

- 위에서 언급했듯이 Request 값과 Response 값이 명확해져서 API 또한 명확해진다.
- 위 그림처럼 swagger API Document를 사용하면 Request와 Response 값이 자동으로 명세되는 장점도 있다.





## 3. Setter 사용 안하기

- JPA에서는 영속성이 있는 객체에 Setter 메서드를 이용해 DB DML이 가능하다.
  - 무분별하게 모든 필드에 대해 Setter 메서드를 작성했을 경우, 의도치 않은 데이터 변경이 일어날 수 있다.
  - 또한, 데이터 변경이 일어났을 때 추적할 포인트들이 많아진다.
  - 하지만, DTO 클래스를 기준으로 데이터 변경이 일어난다면 변경 추적 포인트를 좁히기 쉽다.



```java
// setter 이용 방법 (비추천!!!!)
public Account updateMyAccount(long id) {
    final Account account = findById(id);
    account.setAddress1("변경...");
    account.setAddress2("변경...");
    account.setZip("변경...");
    return account;
}


// Dto 이용 방법
public Account updateMyAccount(long id, AccountDto.MyAccountReq dto) {
  final Account account = findById(id);
  account.updateMyAccount(dto);
  return account;
}

// Account 클래스의 일부
public void updateMyAccount(AccountDto.MyAccountReq dto) {
  this.address1 = dto.getAddress1();
  this.address2 = dto.getAddress2();
  this.zip = dto.getZip();
}
```

- DTO 클래스를 이용해 데이터 변경을 하는 것이 훨씬 더 직관적임.
  - MyAccountReq 클래스에는 3개의 필드가 있으니 오직 3개의 필드만 변경이 가능하다는 것을 알 수 있다.

- 또한, **객체 자신을 변경하는 것은 언제나 자기 자신이어야 한다.**
  - `account.updateMyAccount(dto)` 