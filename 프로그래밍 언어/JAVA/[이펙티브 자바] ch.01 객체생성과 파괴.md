

## Item 1 정적 팩토리 메서드

- 자바의 기본 객체 생성 전략은 생성자
- 하지만 정적 팩토리 메서드(Static Factory Method) 방식도 존재
            
      class Boolean{
      ... 중략
      public static Boolean valueOf(boolean b){
      return b? Boolean.TRUE : Boolean.FALSE
        }
      }
    
- 만약 생성자 패턴이라면
	> Boolean x = new Boolean(y);
- 하지만 정적 팩토리 매서드이므로
	> Boolean x = Boolean.valueOf(y);
### Naming Convention
- of~~(재료로 볼수 있으면, 패러미터로 받을경우)
- from~(재료로 볼수 없으면, 객체로 받을경우)
- valueOf~~(from of와 유사)
- instance, getInstance (같은 인스턴스임을 보장하진 않음)
- create, newInstance(새로운 인스턴스 반환)

### 실 사용 예시
- DTO ->  ENTITY 매핑상황에서 ENTITY에 해당 매핑 메소드를 스태틱 팩토리 메소드로 만들어 볼 수 있음

> Dto convert to Entity Naming Convention
>  Input되는 객체의 경우 Form
>  Output 되는 객체의 경우 Dto

[복잡한 로직을 가진 Dto 생성시 용도 명시를 위해 사용](https://github.com/jinia91/blog/blob/df95ea987a1b8c91ea0b119b6f8ad9793bc4f2e4/src/main/java/myblog/blog/category/dto/CategoryForMainView.java#L26)


### 장점?
1. 객체 생성메소드의 이름을 가질수 있음
	만약 비지니스 로직상 여러 생성자를 오버로딩하는 상황이 나온다면 정적 팩토리 메소드를 사용하는 것이 더 좋은 코드 스타일이 될 것
	
2. 호출될 때마다 인스턴스를 새로 생성하지 않을때도 유용
	- 대표적으로 싱글톤패턴을 생각해볼 수 있음
		- [싱글톤 객체 생성 예시](https://github.com/jinia91/MVC2-Member_Board/blob/74a2df660c9695c0d208996cdcbf9030a1fa9d44/src/main/java/service/MemberService.java#L16)
	- 경량화 패턴(flyweight)에서도 사용됨
3. 반환타입의 하위 타입 객체도 반환할 수 있음!
	- 반환타입을 상위 클래스나 인터페이스로 반환하는 팩토리 메소드, 스태틱 팩토리 메소드 방식으로 인해 보다 유연한 객체 생성이 가능
	- [팩토리 메소드 예시](https://github.com/jinia91/blog/blob/df95ea987a1b8c91ea0b119b6f8ad9793bc4f2e4/src/main/java/myblog/blog/member/auth/UserInfoFactory.java#L10)

### 단점
1. 정적 팩토리 메소드만 제공할 경우 하위 클래스를 만들수가 없음
	- protected 생성자를 만들면 되지않나???
2. 협업 개발자들이 인지하기 어려움


## Item 2 선택적 매개변수가 많다면 빌더 고려

- 객체를 생성할때, 선택적 매개변수가 많을경우 객체 생성을 위한 팩토리메소드든 생성자든 엄청 많이 만들어야 하는 단점이 존재

- 이를 해결하기위해 JBeans setter를 사용할수도 있지만 이럴경우 객체의 일관성이 무너지며 setter를 전부 열어야하므로 비지니스 로직에 문제가 생길 수 있음
	- 또한 JBeans의 경우 불변객체를 만들수 없음

### Builder 패턴으로 해결하자

- 빌더 패턴 코드 예시


	  class Example<T> {
		   	private T foo;
		   	private final String bar;
   	
	   	private Example(T foo, String bar) {
	   		this.foo = foo;
	   		this.bar = bar;
	   	}
   	
	   	public static <T> ExampleBuilder<T> builder() {
	   		return new ExampleBuilder<T>();
	   	}
   	
	   	public static class ExampleBuilder<T> {
	   		private T foo;
	   		private String bar;
   		
	   		private ExampleBuilder() {}
   		
	   		public ExampleBuilder foo(T foo) {
	   			this.foo = foo;
	   			return this;
	   		}
   		
	   		public ExampleBuilder bar(String bar) {
	   			this.bar = bar;
	   			 return this;
	   		}
   		
   		
	   		public Example build() {
   			return new Example(foo, bar);
	   		}
	   	}
	  }

- 원 클래스는 내부 클래스인 빌더 클래스만 생성가능하고 빌더 클래스에서 자신을 반환하는 패러미터 주입 메소드를 작성해 체인으로 빌더 객체를 쌓도록 함
- 이후 마지막으로 build를 통해 모든 패러미터를 한번에 주입한 원클래스의 객체를 생성하고 반환하므로서 선택적 패러미터들을 한번에 주입하여 객체 생성이 가능케함!

### 롬복을 사용하면 아주 간편하게 빌더 생성가능!

- [롬복을 사용한 빌더패턴 작성예시](https://github.com/jinia91/blog/blob/df95ea987a1b8c91ea0b119b6f8ad9793bc4f2e4/src/main/java/myblog/blog/member/doamin/Member.java#L48)
- [빌더 패턴 사용 예시](https://github.com/jinia91/blog/blob/df95ea987a1b8c91ea0b119b6f8ad9793bc4f2e4/src/main/java/myblog/blog/member/service/Oauth2MemberService.java#L60)


## Item 3 싱글턴 패턴

- [싱글턴 패턴 정리](https://github.com/jinia91/TIL/blob/master/Design/%EC%8B%B1%EA%B8%80%ED%86%A4%20%ED%8C%A8%ED%84%B4.md)

- 객체를 들고있는 상수 멤버를 private 지정하냐 안하냐의 차이?
	- 보다 유연한 설계를 위해서는 private을 설정하고 반환 메서드를 public으로 여는방식이 좀더 나음
	- 지연생성 싱글턴 패턴 예시 
	
			  public class Speaker{
			  //final이 아니므로 동적 변경 가능
			  private static Speaker instance;
			  
			  private Speaker(){}
			  // 동적 변경된 상황에서 다시 싱글턴객체를 재 생성시 혹은 지연 생성 전략으로 적용 가능
			  public static Syncronozied Speaker getInstance{
			  if(instance == null){
			  instance == new Speaker();
				  }
			  return instance;

				  }
			  }

## Item 4 인스턴스화를 막으려면 private생성자를 사용하자

- 정적 메서드나 정적 필드만 담은 클래스를 만드는 경우라면 객체 생성이 불필요하므로 private 생성자를 명시하여 사용토록하자. 유지보수 차원에서 private 생성자가 없다면 상속이나 객체 생성으로 무분별한 리소스 낭비가 발생할 수 있음!

- but 객체지향적 설계가 아니므로 그렇게 권장하지 않음. 차라리 싱글턴 패턴으로 만들어 정적 메서드를 사용하거나 정적 필드를 사용하는것이 보다 바람직한 설계이지 않을까?


## Item 5 의존성 주입

###  의존성?
클래스간에 연관관계(의존관계)를 갖는것을 의미
A 클래스의 코드변화가 B클래스에 영향을 주는 관계

### 의존성 주입?
클래스간의 연관관계(의존관계)에 있어서 인터페이스간의 느슨한 연결만 존재하고 구체적인 구현체는 외부에서 주입되는 방식
	
- 인터페이스간 느슨한 연결을 통해 DIP를 지킬수 있고
- 런타임 시점에서 구체적 구현체를 외부 주입으로 받음으로서 OCP를 준수할 수 있다.


**1. Property Injection**

변수값을 프레임워크의 외부 설정으로 주입하는 방식
- [실제 사용 예시](https://github.com/jinia91/blog/blob/df95ea987a1b8c91ea0b119b6f8ad9793bc4f2e4/src/main/java/myblog/blog/member/service/Oauth2MemberService.java#L28)

**2. Construct Injection**

일반적으로 스프링에서 가장 많이 사용되는 방식


## Item 6 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체는 매번 재생성하기보다는 싱글톤 객체나 불변객체를 만들어 활용하자

	- ex) String을 일반 객체로 만들지 말고(**절대**) 리터럴객체로 만들것!

### 비용이 비싼 객체는 캐싱해서 사용하자
- 비용이 비싼 객체인지 파악하는것은 사실 어려움...
- ***메소드를 까보고 내부 구조를 탐색하는 습관을 들이자***
- **EX) String.matches**

	  static boolean isEmailValid(String s){
	  return s.matches("정규 표현식")}
      // 해당 메소드의 경우 메소드가 호출될때마다 mathes안에서 
	  정규표현식이 매번 새로 컴파일된 후 Pattern 객체가 되어 판정메소드가 작동함
	  따라서 메소드를 호출할때마다 불필요한 일회성 객체가 재생성됨

	  ->
	  private static final Pattern EMAIL = 
	  Pattern.compile("정규 표현식");

		이처럼 패턴 객체 자체를 불변객체로 만들어 사용하면 불필요한 객체 생성을 막을 수 있음

> 실제 백준 문제에서 정규 표현식을 사용할일이 생겨 단순 matches사용과 compile한 static 멤버 변수로 비교해봄
> [관련문제](https://www.acmicpc.net/problem/3613)
> 
|| 메모리 |시간  |
|-|--|--|
|matches| 11692 | 84ms |
|compile| 11656 | 80ms |

유의미한 차이는 아닌거같은데...?


### 박싱 타입 vs primitive 타입
- primitive 타입으로만 계산하는것과 오토박싱이 뒤섞인 로직간에는 일반적으로 10배가까이 성능차이가 남
- 오토박싱의 비용을 생각할때, 성능에 민감한 로직이라면 사용하지 않는것이 맞다.

#### 하지만 박싱타입이 유용한 케이스도 존재
- Null Case
	- 비지니스 로직상 null이 의미를 갖는경우
		- price null != price 0
		- 가격이 아직 정해지지 않음을 나타낼수도 있음
- Optional하게 값을 받아야하는경우를 잘 생각해보자

## Item 7 다쓴 리소스는 해제해서 메모리를 관리하자

- GC에만 의존하는 프로그래밍은 필연적으로 성능저하를 낳음

- JVM, GC, 메모리구조에 대한 보다 깊은 공부를 할 필요성

## Item 8 Finalizer ,Cleaner 사용하지말자!

- 실무에서도 쓸일은 거의 없음...

## Item 9 try - with -resource 로 자동 리소스 해제를 쓰자


- 예시
 
		try (DataOutputStream dos = new DataOutputStream(new FileOutputStream("d_data.txt", true));  
		       DataInputStream dis = new DataInputStream(new FileInputStream("d_data.txt"))) {  
		        
		      // 파일에 출력하기  
		  dos.writeUTF("사람이름");  
		      dos.writeInt(20);  
		      dos.writeChar('M');  
		      dos.writeBoolean(false);  
		      dos.writeDouble(3.141592);  
		        
		      // 파일에서 값을 읽어오기  
		  while(true) {  
		         System.out.println("이름은 " + dis.readUTF() +   
		                        ", 나이는 " + dis.readInt() +   
		                        ", 성별은" + dis.readChar() +   
		                        ", 결혼은 " + dis.readBoolean() +   
		                        ", IQ는 " + dis.readDouble());  
		      }  
		   } catch (FileNotFoundException e) {  
		      e.printStackTrace();  
		   } catch (EOFException e) {  
		      System.out.println("파일 읽기 완료!!!");  
		   } catch (IOException e) {  
		      e.printStackTrace();  
		   }  
		}


- try (리소스) 를 명시해주면 try 구문이 끝날때 괄호안의 리소스를 자동해제시켜줌!
- 리소스 객체 안에는 AutoCloseabe 인터페이스를 상속받아 close 메소드를 구현해내서 리소스를 닫을때 전처리로직을 구현해놓으면 사용가능
	