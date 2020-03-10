# Object(4) & Tell, Don't Ask

자기 상태(객체 상태)를 자기가 아는것.

## LSP(리스코프 치환 원칙)

- 성급한 추상화를 통해 문제가 일어남
  - abstract `c()` --> correct
  - abstract a(),b()`/*,not c()*/`
    - case1 : a(),b(),c()
    - case2 : a(),b(),c()
    - case3 : a(),b()
- downcasting이 일어나지 않게 하기 위해 원칙임 생김
  - 어쩔수 없는 케이스가 생겼다면
    - abstract a(),b(),c()
      - case1 : a(),b(),c()
      - case2 : a(),b(),c()
      - case3 : a(),b(),c(),`d()`  
    - 여러개의 case조건을 만듬으로서 `context 에러`가 생긴다.
  - `generic을 통해 이문제를 해결`


### 도메인 구조

```txt
                                |---->FrontEnd
                                |
           |----> Programmer ---|
           |         |          |---->BackEnd
Director---|         |
           |         |
           |        \|/         |---->ServerClient
           |---->  Paper     ---|
                                |---->Client
```

```java
public interface Paper{}

public interface Programmer{
    Program makeProgram(Paper paper);
}

public class Client implements Paper{
    Library libray = new ...
    Language language = new ...
    ...
    public setProgrammer(Programmer programmer){
        this.programmer=programmer
    }
}

public class FrontEnd implements Programmer{
    private Library libray;
    private Language language;
    @Override
    public Program makeProgram(Paper paper){
        if (paper instanceof Client){
            Client pb = (Client)paper;
             /**
            여기서 Paper가 Client로 downcasting하고 있으며, LSP를 위반하고 있다. and OCP도 위반
            
            BackEnd class인 경우:
            if (paper instanceof ServerClient){
                ServerClient pb = (ServerClient)paper;
            }
            */
            libray = pb.library
            language = pb.language
        }
        return makeFrontEndProgram();
    }
    private Program makeFrontEndProgram(){return new Program()}
}
```

## LSP위반 시행착오

```java
public class FrontEnd implements Programmer{
    private Library libray;
    private Language language;
    @Override
    public Program makeProgram(Paper paper){
        if (paper instanceof Client){ //1.LSP 위반
            Client pb = (Client)paper;
            libray = pb.library
            language = pb.language
        }
        return makeFrontEndProgram();
    }
    private Program makeFrontEndProgram(){return new Program()}
}
```

- LSP위반으로 OCP 위반된다.

### Don't Ask

```java
public class BackEnd implements Programmer{
    @Override
    public Program makeProgram(Paper paper){
        paper.setData(this);
        // 언어 시스템상 무슨 형을 보냈는지도 모름.(공변 관계)
        return makeFrontEndProgram();
    }
    ...
}
```

할리우드 원칙을 잘 지켰다 생각하지만, Client는 setData 메소드를 만들어야 하고, FontEnd 클래스에서도 똑같은 같은 코드가 들어간다.

--> Dry 원칙 위배

```java
public abstract class Programmer{
    public Program getProgram(Paper paper){ // template method
        paper.setData(this);
        return makeProgram(paper);
    }
    abstract Program makeProgram(Paper paper); //hook
}
```

`템플릿 메소드 패턴`(추상화)을 통해 `Dry 원칙`을 지켰다.

```java
public interface Paper{
    void setData(Programmer programmer);
}

public class Client implements Paper{
    ...
    public void setProgrammer(Programmer programmer){
        ...
    }
    @Override
    public void setData(Programmer programmer){
        if (programmer instanceof FrontEnd){ //DownCasting
            ...
        }
    }
}
```

하지만 문제는 미뤄졌을뿐...
설계를 잘한다 하더라고 코드로는 다운케스팅 된다. 현실에서는 `추상층이 포함할 수 있는 내용은 적은게 일반적`이기 때문이다.

```java
public class SeverClient implements Paper{
    ...
    public void setProgrammer(Programmer programmer){
        ...
    }
    @Override
    public void setData(Programmer programmer){
        if (programmer instanceof FrontEnd){ //DownCasting
            ...
        }else if(programmer instanceof Backend){
            ...
        }
        /**
        도메인 로직과 다르게 1:N 로직이 되어서
        N개의 케이스에 대해서 일반화된 처리해야 한다.
        더 난이도가 높아 버렸다.
        */
    }
}
```

다운케스팅을 할때도 SeverClient가 지는 책임이 더 무겁다.
1:N 로직이 되어 N개의 case를 일반화된 처리해 줘야 하기 때문이다.
이로서 문제가 더 심해졌다.

게다가 Client 클래스에서는 1개의 Case만 처리했다.
Class의 형태를 기준으로 코드를 짰다고 볼 수도 있다. 조심 해야 한다.

`결국에는, 문제는 이전 됐을 뿐 근본적으로 문제는 해소되지 않았다.`

### Generic

> 객체지향 의미로는, 추상형을 유지 하면서 구상형을 클라이언트가 결정히게주는 로직

if문과 차이점은 형으로 결정된다.
`내 코드(java)에 instanceof(downcasting)가 있다면 쓰면 된다!`

Runtime, Context 에러를 Compile Time Error의 문제를 옮길 수 있다.

```java
public interface Paper<T extends Programmer>{
    void setData(T programmer);
}

public class Client implements Paper<FrontEnd>{
    ...
    public void setProgrammer(FrontEnd programmer){
        ...
    }
    @Override
    public void setData(FrontEnd programmer){ //구상형
        programmer.setLibrary(library);
        programmer.setLanguage(langugae);
        //제약 범위내에서 움직임.
    }
}
```

generic이 해주는 것은 instanceof를 제거해 주는 것 까지다.
대신에, 구상 클래스의 용도를 제약했다. 런타임 에러 대신 컴파일 타임에 형 에러가 일어난다.

이 코드는 OCP, LSP도 어기지 않았다.

```java
public class SeverClient implements Paper<>{
    ...
    public void setProgrammer(Programmer programmer){
        ...
    }
    @Override
    public void setData(Programmer programmer){
        if (programmer instanceof FrontEnd){ //DownCasing
            ...
        }else if(programmer instanceof Backend){
            ...
        }
    }
}
```

하지만 이코드에서는

```java
public class SeverClient implements Paper<FrontEnd? or BackEnd?>{

```

이렇게 만들어야 하나? 1:N(paper:programmer) 구조여서 N을 구상하지 말고, 1을 구상하는게 좋다 라는 결론을 얻었다.

다시 Programmer가 책임을 지는 걸로 돌아가자!

## OCP와 제네릭을 통한 해결(Hollywood 원칙, Tell, Don't Ask)

```java
public interface Paper{}//marker interface

public class Client implements Paper{
    Library libray = new ...
    Language language = new ...
    protected Programmer programmer;
    ...
    public setProgrammer(Programmer programmer){
        this.programmer=programmer
    }
}

```

clone interface:
jvm native machine에 따라서 [serialize]를 구현하는걸 jvm 코드에 내장하기 위해서 clone은 marker interface다.

그럼 다시 롤백해서,

```java
public abstract class Programmer<T extends Paper>{ //변경
    public Program getProgram(T paper){
        setData(this);  //Tell!(Hollywood 원칙)
        return makeProgram(paper);
    }
    abstract void setData(T paper);
    abstract Program makeProgram();
}
```

programmer 쪽으로 paper를 알도록 수정.
setData의 책임을 Programmer가 가지고, [abstract]이니 구성은 자식 클래스한테 위임함

할리우드 원칙은 부모자식 관계에서도([has a 관계]를 가짐) 원칙을 지켜야함. 그래야 부모 객체에 여파가 없다.

```java
public abstract class BackEnd<T extends Paper> extends Programmer<T>{  // 수많은 backend 개발자가 있음
    protected Server server;
    protected Language lanuage;
    @Override
    protected Program makeProgram(){
        return new Program()
    }
}
```

generic을 쓰면 추상화가 깨지는게 아니라 제약이 걸린다. <T extends Paper> 씀으로서 if의 case만큼 클래스를 만들어야 한다. 
  
`형으로 풀어야 한다.`

BackEnd 클래스가 추상화 클래스이므로 setData `더 클라이언트 쪽으로 밀어냈다`
아까는 밀어내지 않고 협력 관계에 있는 (같은 책임 레벨을 갖는)도메인 레이어에 떠 넘겼다.

`if를 제거하기 위해서는 같은 클라이언트 방향으로 밀어내야한다.`

## 클라이언트의 변화

```java
public class Director{
    private Map<String, Paper> project = new HashMap<>();
    public void addProject(...){...}
    public void runProject(String name){
        if(!project.containkey(name)) throw new RuntimeException("no project");
        if(paper instanceof ServerClient){
            ...
            ServerClient project = (ServerClient)paper;
            Programmer frontEnd = new FontEnd<ServerClient>(){ //if 대신 형으로 나눔
                @Override
                void setData(ServerClient paper){
                    language = paper.frontEndLanguage;
                }
            };
            Programmer backEnd = new BackEnd<ServerClient>(){
                @Override
                void setData(ServerClient paper){
                    server = paper.server
                    language = paper.frontEndLanguage;
                }
            };
            ...
            deploy(name, project.getProgram(project));
        }else if(paper instanceof Client){
            ...
            deploy(name, project.getProgram(project));
        }
    }
    private void deploy(String name, Program...programs){}
    /**
    어떤 객체를 만들어 낼지는 디렉터가 선택했다.(하드코딩:ServerClient)
    if 대신 case 만큼 형을 만듬으로 해결
    */
}
```

디렉터가 `N`개의 프로젝트를 가지고 있어 디렉터 추상화는 불가능 (1(디렉터):N(페이퍼))
그래서 의존성 역전을 시켜야함.

--> Paper(가벼운쪽으로)로 가서 서비스를 제공하면 되겠다.

### 디렉터의 코드의 문제점 & 해결

```java
if(paper instanceof ServerClient){
```

OCP 위반! 이를 해소 하기 위해

1. 더큰 클라이언트로 이동
2. 형을 경우의 수만큼 생성(추상화를 통해)

```java
public interface Paper{
    Program[] run()
}

public abstract class ServerClient implements Paper{
    //코드는 같음
}

public abstract class Client implements Paper{
    //코드는 같음
}
/*
ServerClient, Client 코드의 변화는 abstract 뿐이다.
수행하는 방법에 따라 형만 더 만들꺼니 추상화만함.

무조건 형을 만들고 객체를 할당해야함. 거기에 합당한 케이스이기 때문이다.
**/

public class Director{
    private Map<String, Paper> project = new HashMap();
    public void addProject(...){...}
    public void runProject(String name){
        if(!project.containkey(name)) throw new RuntimeException("no project");
        deploy(name, project.get(name).run());
    }
    private void deploy(String name, Program...programs){}
}
```

디렉터보다 더 클라이언트인 Main까지 밀어야함.

Main까지 코드를 밀어내야지만 [DI]가 작동함.
코드가 잘 작성 됐다면 말단만 @autowide 해야한다.

```java
public class Main{
    public static void main(String[] args){
        Director director = new Director();
        director.addProject("A 프론트 개편", new Client(){
            //수행방법이 기술되 있다.
            //잘 만들었다면 if가 아닌 형을 만드는 것으로 구성 되있다.
            @Override
            public Program[] run(){
                FrontEnd frontEnd = new FrontEnd<Client>(){
                    @Override
                    void setData(Client paper){
                        library = paper.library;
                        language = paper.language;
                    }
                };
                setProgrammer(frontEnd);
                /**set은 클래스의 은닉성을 깨고 있다.*/
                return new Program[](frontEnd.getProgram(this));
            }
        });
        director.runProject("A 프론트 개편");
    }
}
```

여기서 추가적으로 봐야할 코드

```java
setProgrammer(frontEnd);
```

이 인터페이스는 ClientPaper가 외부에 자기의 속성을 갱신할 수 있는 메세지를 허용하려고 만든거지만, 
이제는 ClientPaper기 형으로 확장했지 때분에 외부용 메세지가 필요가 없다!

`보안성이 더 높아진다.` 클래스의 은닉성이 더 높아진다.

위 코드는

```java
public class Main{
    public static void main(String[] args){
        Director director = new Director();
        director.addProject("A 프론트 개편", new Client(){
            //수행방법이 기술되 있다.
            //잘 만들었다면 if가 아닌 형을 만드는 것으로 구성 되있다.
            @Override
            public Program[] run(){
                FrontEnd frontEnd = new FrontEnd<Client>(){
                    @Override
                    void setData(Client paper){
                        library = paper.library;
                        language = paper.language;
                    }
                };
                programmer = frontEnd;
                // 묻지않고 본인이 수행, 한층 더 은닉성이 올라감.
                // 보안수준이 protected로 올라감
                return new Program[](frontEnd.getProgram(this));
            }
        });
        director.runProject("A 프론트 개편");
    }
}
```

로 변경하면 된다.

정말 객체 설계를 잘했다면, public set,get 메소드가 외부에 노출될 일이 없다. 이미 설계가 잘 못 된걸 수도..

`관계가(1:N) 더 가볍느냐에 따라 구상을 거기로 넘겨야 한다.`

N쪽으로 가야 구성할게 1개로 가볍다.
만약 관계가 바뀌면 코드 배치를 달리 해야 한다.

반복 연습을 통해 코드 배치를 학습 해야 한다.

- if & down casting(LSP & OCP 위배) 처리:
  1. 가장 큰 클라이언트로(Main) 쭉 밀어야함.
  2. generic을 통해 형을 만듬(추상화), 책임을 구상클래스로 위임
  3. 형 생성시 속성을 외부에 할당(set) 받지 않고 자기가 할당, 한층 더 은닉성이 올라감.

## 참조

[has a 관계]:https://ddaisy.tistory.com/entry/UML-Class-Diagram-RelationshipsDependency-Aggregation-Composition-Inheritance-Realization

[serialize]:https://woowabros.github.io/experience/2017/10/17/java-serialize.html

[abstract]:https://opentutorials.org/course/1223/6062

[DI]:https://www.tutorialsteacher.com/ioc/introduction

[JAVA-pinpoint](https://www.javatpoint.com/java-string)
