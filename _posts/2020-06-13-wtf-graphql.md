---
layout: post
title: "GraphQL이 뭐길래?"
author: "dongpark"
category: post
---
## What is GraphQL?

- GraphQL 은 페이스북에서 만든 쿼리 언어
- SQL과 목적은 같지만 사용주체는 다른편?
    - sql은 **데이터베이스 시스템**에 저장된 데이터를 효율적으로 가져오는 것이 목적
    - gql은 **웹 클라이언트**가 데이터를 서버로 부터 효율적으로 가져오는 것이 목적
- HTTP POST 메서드와 웹소켓 프로토콜을 활용
- Rest-API와 다르게 정의한 스키마에 맞춰서 클라이언트가 입맛에 맞게 데이터베이스를 꺼내 쓸 수 있다.

## JPA와 Graph-ql의 콜라보레이션

환경설정 및 스키마

- application.yml

```yaml
spring:
  ##############################################
  ####### JDBC DATASOURCE
  ##############################################
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://localhost:3306/graph-ql?serverTimezone=Asia/Seoul&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
    username: root
    password: '비밀번호'

  jpa:
    hibernate:
      ddl-auto: update
    generate-ddl: false
    database: mysql
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MariaDB53Dialect

graphql:
  spqr:
    gui:
      enabled: true
```

- member.graphqls

```graphql
schema {
    query : Query
    mutation: Mutation
}

type Query {
    members(count: Int, offset: Int) : [Member]
    member(index: Int): Member
}

type Mutation {
    register(id : String, password: String, name: String, phoneNumber: String, gender: String) : Member
}

type Member {
    memberIndex : Int
    password: String
    identify : String
    name : String
    gender : String
}
```

1차도전

- [Member.java](http://member.java)
    
    ```java
    public class Member {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @GraphQLQuery(name = "memberIndex")
        long memberIndex;
    
        ///아이디
        @Column(length = 30, unique = true)
        @GraphQLQuery(name = "identify")
        String identify;
    
        ///비밀번호
        @Column(length = 255)
        @GraphQLQuery(name = "password")
    		String password;
    ```
    
    *각 컬럼마다 GQL에서 쓰이는 컬럼이라고 명시해준다*
    
- [MemberQuery.java](http://memberquery.java)
    
    ```java
    @AllArgsConstructor
    public class MemberQuery implements GraphQLQueryResolver {
    
        private MemberRepository memberRepository;
    
        public List<Member> getMembers(int count, int offset){
            return memberRepository.findAll();
        }
    
        public Optional<Member> getMember(Long memberIndex){
            return memberRepository.findById(memberIndex);
        }
    ```
    
- MemberMutation.java
    
    ```java
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    @GraphQLMutation(name = "MemberMutation")
    public class MemberMutation implements GraphQLMutationResolver {
    
    MemberRepository memberRepository;
    
        public Member register(String id, String password, String name, String phoneNumber, String gender) {
            password = parseSha256(password);
            Member member = Member.builder().identify(id).name(name).password(password).phoneNumber(phoneNumber).gender(gender).build();
            return memberRepository.save(member);
        }
    
        public String withdraw(long memberIndex){
            Optional<Member> optional = memberRepository.findById(memberIndex);
            if(optional.isPresent()){
                Member member = optional.get();
                member.setWithdrawTime(LocalDateTime.now());
                memberRepository.save(member);
                return "회원 탈퇴가 정상적으로 완료되었습니다.";
            }else{
                return "유효 하지 않은 회원 입니다.";
            }
        }
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ```
    
- 장점 : 스키마에서 선언한 쿼리와 뮤테이션이 메소드명과 일치하기만 하면 사용이 가능해서 이용하기 손쉽다.
- 문제점 : 모든 도메인이 address/graphql 로 끝난다 → 구별이 안간다 → 각 도메인별로 각각의 서비스 레이어를 분리할수 없다

2차도전

- 각 도메인에 맞게 DataFetcher 클래스를 구현하여 각각 매핑하는 방법을 이용한다
- [AllMemberDataFetcher.java](http://allmemberdatafetcher.java)
    
    ```java
    @Component
    public class AllMemberDataFetcher implements DataFetcher<List<Member>> {
    
        private final MemberRepository memberRepository;
    
        public AllMemberDataFetcher(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    
        @Override
        public List<Member> get(DataFetchingEnvironment dataFetchingEnvironment) {
            return memberRepository.findAll();
        }
    }
    ```
    
- [MemberDataFetcher.java](http://memberdatafetcher.java)
    
    ```java
    @Component
    public class MemberDataFetcher implements DataFetcher<Member> {
    
        private final MemberRepository memberRepository;
    
        public MemberDataFetcher(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    
        @Override
        public Member get(DataFetchingEnvironment dataFetchingEnvironment) {
            long memberIndex = dataFetchingEnvironment.getArgument("memberIndex");
            return memberRepository.findById(memberIndex).orElse(null);
        }
    }
    ```
    
- [MemberProvider.java](http://memberprovider.java)
    
    ```java
    @Component
    public class MemberProvider implements MemberDetails {
    
        private MemberRepository memberRepository;
        private AllMemberDataFetcher allMemberDataFetcher;
        private MemberDataFetcher memberDataFetcher;
    
        @Value("classpath:graph-ql/member.graphqls")
        Resource resource;
    
        private GraphQL graphQL;
    
        @Autowired
        public MemberProvider(MemberRepository memberRepository, AllMemberDataFetcher allMemberDataFetcher, MemberDataFetcher memberDataFetcher) {
            this.memberRepository = memberRepository;
            this.allMemberDataFetcher = allMemberDataFetcher;
            this.memberDataFetcher = memberDataFetcher;
        }
    
        @PostConstruct
        private void loadSchema() throws IOException {
            File file = resource.getFile();
    
            TypeDefinitionRegistry typeDefinitionRegistry = new SchemaParser().parse(file);
            RuntimeWiring runtimeWiring = buildRuntimeWiring();
            GraphQLSchema graphQLSchema = new SchemaGenerator().makeExecutableSchema(typeDefinitionRegistry, runtimeWiring);
            graphQL = GraphQL.newGraphQL(graphQLSchema).build();
        }
    
        private RuntimeWiring buildRuntimeWiring() {
            return RuntimeWiring.newRuntimeWiring()
                    .type("Query", typeWiring -> typeWiring
                            .dataFetcher("members", allMemberDataFetcher)
                            .dataFetcher("member", memberDataFetcher)).build();
        }
    
        @Override
        public ExecutionResult execute(String query) {
            return graphQL.execute(query);
        }
    }
    ```
    
- MemberController.java
    
    ```java
    @RestController
    @RequestMapping(value = "/api")
    @AllArgsConstructor
    public class MemberController {
    
        private MemberProvider memberProvider;
    
        @PostMapping(value = "/member")
        public ResponseEntity<Object> getAllMembers(
                @RequestBody String query
        ) {
            ExecutionResult executionResult = memberProvider.execute(query);
            return new ResponseEntity<>(executionResult, HttpStatus.OK);
        }
    }
    ```
    
- 딱 봐도.. 각 도메인마다 구현해야 할게 너무 많아보인다 → 반대로 기존에 쓰던 Hibernate와 결합할 수 있어보인다.

![alt text](https://s3.ap-northeast-2.amazonaws.com/dongpark.land.image/361638261336993.png)


