# Object(2) & Type

Data Type에 대한 얘기가 아니라, 객체지향에서 Type은 `형`을 의미한다.

## Type

- Role : `형을 통해 역할을 묘사`함

- Responsibility : 형을 통해 로직을 표현함

- Message : 형을 통해 메세지를 공유함
  - String, int은 형이아니다. 값(불변, 복제 후 리턴)이다.
  - 우리가 보는 형은 참조 타입을 의미한다.

- Protocol : 객체간 계약을 형을 통해 공유함

### JVM 도용 가능한 Type : static, enum, class

- static: 단 한개 인스턴스 존재
singleton 대신사용
언어적인 static 초기화, 호출, 작동이 전부 쓰레드 안전이 아니다.
`JVM이 쓰레드 안전을 보장하지 않는다.`

동시성 문제에서, static context 버리고 instance context를 사용해야 한다.

- enum: 제한된 수의 인스턴스 존재(제너릭에 사용불가)
enum은 인스턴스는 제일먼저 생성, `생성 시점의 동시성 안전 문제는 해결된다.`
일반적인 인스턴스가 확정적이면 enum instance를 만들면 된다.

단점은 제너릭을 사용할 수 없다.형을 대체하거, 유연성은 떨어진다.

- class : 무제한의 인스턴스 존재
utility function(사용X) vs method(사용) :  this의 유무에 따라 다름

## Condition

1. 조건 분기는 결코 제거할 수 없다.
2. 조건 분기에 대한 전략은 두가지 뿐
   1. 내부에서 응집성 있게 모아두는 방식
      - 장:모든 경우의 수가 한눈에 파악가능
      - 단:분기가 늘어날 때마다 코드 변경
   2. 위부에 분기를 위임하고 경우의 수 만큼 처리기를 만드는 방식
      - 장: 분기가 늘어날 때마다 처리기만 추가하면 된다.
      - 단: 모든 경우의 수를 파악할 수 없다.

## Responsibility Driven

> 객체 만이 역할을 수행할 수 있고 Type(not Value) 만이 책임을 수행 할 수 있다.

### value = responsibility (책임이란건 가치 그 자체)

- 책임(working 함수)을 작은 책임으로 분할
- 여러 '책임'의 공통점을 모아 추상화 해놓는 것을 '역할(interface)'이라 할 수 있다
  - 연역적인 방법을 통해 역할을 만들고 귀납적인 방법을 통해 다른 사례를 더 만들 수 있음
  - 추상화 분기를 통해 if분기 만큼의 객체를 만들어 외부에 위임할 수 있음

```java
void func(){
    String v = 'c';
    Runnable run = null
    if (v.equals("a")){
        run = new Runnable(...)
    }else if(...){
        run = new Runnable(...)
    }
    processCondition((run)
}

void processCondition(Runnable condition){ //역할
    condition.run()
}

```

- 역할에 따라 협력이 정의됨
  - 책임을 가지고 협력을 정의하면 안됨.

### 값 객체, reference 객체

### generic class

### 정보 전문가 패턴

- 가장 많은 정보를 가지고 있는 객체에 많은 책임을 부여한다.
- 하지만 이점에서 개념과의 출돌이 일어 날 수 있다.
- ex. 스크린(객체).reservation()
  
- IDBA 정규화를 통해 객체의 역할과 책임을 분리할때 정보를 분리할 수 있다.
  - 1번, 2번 영화(10시 영화)(2 인스턴스)
  - 10시 영화(1번, 2번 영화)(1 인스턴스)

## Discount Type

```java
// DiscountCondition <- DiscountPolicy <- Movie
interface DiscountCondition{
    public boolean isSatisfiedBy(Screeing, int)
    public Money calculateFee(Money)
}
interface DiscountPolicy{ // mark interface
    interface AMOUNT extends DiscountPolicy{}
    interface PERCENT extends DiscountPolicy{}
    ...
}
abstract public class SequenceDiscount implements DiscountCondition{ // sequence에 대한 처리만 하겠다
    private final int sequence;
    SequenceDiscount(int sequence){
        this.sequence = sequence;
    }
    @Override
    public final boolean isSatisfiedBy(Screeing s, int audienceCount){
        return s.sequence == sequence;
    }
}
public class SequenceAmountDiscount extends SequenceDiscount implements DiscountPolicy.AMOUNT{
    //amount로직만 책임짐
    private final Money amount;
    public SequenceAmountDiscount(int sequence, Money amount){
        super(sequence)
        this.amount = amount
    }
    @Override
    public Money calculateFee(Money fee){
        return fee.minus(amount)
    }
}
/**
* DiscountCondition <- DiscountPolicy <- Movie
* 위 구조가 아니라 DiscountCondition <- Movie
* 구조가 되 버렸다. 도메인과 안맞다
*/

```

- iterator pattern(수동적 객체, lazy, 외부사정에 맞춰 나를 진행)
  - hasnext, next
  - 내가 바른 조건을 갖고 있는지, 액션이 무엇인지가 중요하다.
  - asyc, await(kotlin)
  - 발동 트리거 & 액션으로 2개 책임을 가짐

- interface
  - 메소드 하나를 가지는게 중요함
    - 두개이상을 가지면 해체 불가능함
  - 안에 메소드 정의가 하나도 없는 interface가 가장 좋음
    - ex. Cloneable, Serializable(mark interface)
      - jvm branch마다 일을 다르게 함
      - anotation process

- 메세지(오직 1개의 인자)
  - 추상화 & 형 이어야함

### Discount Type(implement inversion)

```java
// DiscountCondition <- DiscountPolicy <- Movie
public class AmountDiscount implements DiscountPolicy.AMOUNT , SequenceDiscount{
    //amount로직만 책임짐
    private final Money amount;
    public SequenceAmountDiscount(Money amount){
        this.amount = amount;
    }
    @Override
    public Money calculateFee(Money fee){
        return fee.minus(amount);
    }
}
public class PercentDiscount implements DiscountPolicy.PERCENT , SequenceDiscount{
    //amount로직만 책임짐
    private final double percent;
    public SequenceAmountDiscount(double percent){
        this.percent = percent;
    }
    @Override
    public Money calculateFee(Money fee){
        return fee.minus(fee.multi(percent));
    }
}
/**
인터페이스 2개만 있어도 경우의 수가 폭증한다. 그래서 설계시 도메인이 중요하다.
*/


public class SequenceAmountDiscount extends AmountDiscount{
    private final int sequence;
    SequenceDiscount(Money amount, int sequence){
        super(amount)
        this.sequence = sequence;
    }
    @Override
    public final boolean isSatisfiedBy(Screeing s, int audienceCount){
        return s.sequence == sequence;
    }
}

public class SequencePercentDiscount extends PercentDiscount{
    private final int sequence;
    SequenceDiscount(double percent, int sequence){
        super(percent)
        this.sequence = sequence;
    }
    @Override
    public final boolean isSatisfiedBy(Screeing s, int audienceCount){
        return s.sequence == sequence;
    }
}
/**
isSatisfiedBy가 병합돼있는 경우의 수의 코드 중복이다.(상속의 한계) 이것을 제거할려면 전략 객체를 받아들일 수 밖에 없다.
코드를 자동생성하는 코틀린 같은 경우 class By를 통해 처리 가능하다.
*/


public class Movie<T extends DiscountPolicy & DiscountCondition>{
    private final Set<T> discountCondition = new HashSet<>()
    public Movie(..., T...conditions){
        ...
        this.discountContions.addAll((Arrays.asList(conditions)));
    }
    /**
    list를 쓰는 순간 값 context를 생각하는 것이다.
    객체 context를 쓰려면 set을 쓰는게 맞다.
    */

    Money calculationFee(Screening s, int audienceCount){   for(T condition: discountCondition){
            if(condition.isSatisfiedBy(s,audienceCount)){
                return condition.calculateFee(fee).multi((double)audienceCount);
            }
        }
        return fee;
    }
    /**
    Movie T형은 policy이면서 condition형만 올 수 있고, 이것만으로 if를 대체할 수 있음!(상속받은 클래스를 만듬으로서 각 if case를 처리 할 수 있음) - 이름이 조건처럼 써있음 class AB
    */
}
```

- generic을 사용해서 if문 없애버림
- if문 <-> generic 코드를 교체하는게 자연스러워함.

## Value Object

> 1. 불변
> 2. 리턴 새 객체를 return

```java
public class Money{ //Value Object
    public static Money of(Double amount){}
    private final Double amount; // 값 객체는 무조건 final
}

public class Screenint{  //Value Object
    private int seat;
    final int sequence;
    final LocalDateTime whenScreened;
    public Screening(...){
        ...
    }
    boolean hasSeat(int count){ //trigger
        return seat >= count;
    }
    void reserveSeat(int count){ // action
        if(hasSeat(count)) seat -= count;
        else throw new RuntimeException("no seat");
    }
    /**
    trigger & action이 보인다면
    외부상황에 맞춰 lazy하게 동작하게 하는 것 이다.
    */
}

public class Theater{
    ...
    void plusAmount(Money amount){
        this.amount = this.amount.plus(amount)
    }
    /**
    - 값 객체의 숙명(새객체이기 떄문에 재할당 필요)
    - 값객체는 포인터의 포인터를 깨먹고 있다.
    - this.amount가 절대 외부에 노출되면 안된다.
    - 값객체를 외부에 노출할때는 값으로 환원해서 노출 할 수 밖에 없다.
        - (double)this.amount.amount
    */
}

void main()
{
    Theater theater = new Theater(Money.of(100.0))
    Movie movie = new Movie<AmountDiscount>(
        "spiderman",
        Duration.ofMinutes(120L)
        Money.of(5000.0)
        new SequenceAmountDiscount(Money.of(100.0))
        //형으로 if문 대체
    );
    theater.addMovie(movie);
    /**
    Injection은 DI가 해주고
    우리가 할 일은 injection할 DI의 객체만 만들어주면 됨.
     -> 컴파일 x, json만 건들면 됨
    */
}
```

## 참고

[Serializable & Clonable](https://www.geeksforgeeks.org/marker-interface-java/)