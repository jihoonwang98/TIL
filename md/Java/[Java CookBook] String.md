# String



**String, StringBuilder의 차이**

![](https://lh4.googleusercontent.com/qghGTuQc3bA5fDc1PD_-udHCq2YFcVNz0qM4RFFSkfQrJ42PlZO9Tyo7DJCaE90M2hRm-6-w8ZO_KKsIRQWq1TkePHYI14EKfNn_jYquP0e1-41R92ORyV6JPIUve5G1tSvQuJPO)



**String**은 immutable하고, **StringBuilder**는 mutable하다.

따라서 String 연산을 할 때 차이가 생기는데, 아래의 예제를 보자.



![](https://docs.google.com/drawings/d/sOEXnVUW5d_dKOATz2aGPMg/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=199&h=346&w=394&ac=1)



**String**은 <u>불변</u> 객체이기 때문에 concatenation을 하면 새로운 String 객체를 생성한다.





![](https://docs.google.com/drawings/d/sw7aG3OyFx3UAlcJz87Wacg/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=65&h=201&w=394&ac=1)





그러나, **StringBuffer**는 <u>변할 수 있는 mutable</u> 객체이기 때문에 

객체를 직접 조작 가능하다.







## Substring 만들기

`substring()` 메서드는 해당 문자열을 직접 변형하는 것이 아니라

**<u>새로운 String 인스턴스를 생성</u>**하는 것임에 주의...

위에서도 알 수 있듯이 **String**은 **immutable**하다.



그리고, index는 *zero-based*(0부터 시작)임에 주의한다.





### Method Signature

- **문자열.substring(start)**
  - start index부터 맨 끝까지 추출
- **문자열.substring(start, last)**
  - start index부터 **<u>last index 전</u>**까지 추출
  - [start, last)를 추출하는 거임



```java
public class SubStringDemo {
    public static void main(String[] args) {
        String a = "Java is great.";
        System.out.println(a);
        String b = a.substring(5);
        System.out.println(b);
        String c = a.substring(5, 7);
        System.out.println(c);
        String d = a.substring(5, a.length());
        System.out.println(d);
    }
}

/* output */
Java is great.
is great.
is
is great.
```







## String Tokenizing하기 (split)



### (1번) 정규 표현식과 **`split()`** 메서드 이용

```java
public class Tokenizing_Split {
    public static void main(String[] args) {
        String str = "abc dfs asdf emrwe sd";
        for (String word : str.split(" ")) {
            System.out.println(word);
        }
    }
}
```

(나중에 정규 표현식을 공부할 때 더 자세히 포스팅하겠다...)







### (2번) StringTokenizer 클래스의 `hasMoreTokens()`와 `nextToken()` 메서드 이용



1. StringTokenizer 클래스를 Tokenizing 하고 싶은 String으로 초기화
2. `hasMoreTokens()`와 `nextToken()` 메서드로 tokenizing된 String들 추출







#### 단어 기준(공백 기준) 분리



**Method Signature**

```java
new StringTokenizer(String strToTokenize)
```



```java
public class Tokenizing_Tokenizer {
    public static void main(String[] args) {
        StringTokenizer st = new StringTokenizer("Hello World Of Java");

        while (st.hasMoreTokens()) {
            System.out.println("Token : " + st.nextToken());
        }
    }
}
/* output */
Token : Hello
Token : World
Token : Of
Token : Java
```



- 생성자에 String만 넘겨주면 European language들에서 "word boundary"로 생각되는 기준으로 잘린다.

  - 무슨 말이냐면 일반적으로 생각할 수 있는 "단어" 기준으로 잘린다는 말이다.

  - ```java
    StringTokenizer st = new StringTokenizer("Hello \tWorld    Of\n Java");
    ```

    위와 같이 넘겨줘도 위의 예제와 똑같이 나온다.





#### 지정한 문자 기준 분리



**Method Signature**

```java
new StringTokenizer(String strToTokenize, String delimiters)
// 두번째 파라미터인 delimiters에 tokenizing 기준 문자들을 나열한 string을 넘겨주자
```



```java
StringTokenizer st2 = new StringTokenizer("Hello, World|of|Java", ", |"); 
// 위에서 기준 문자는 총 세 개(',', ' ', '|')다. -> 공백도 포함임

while (st2.hasMoreElements()) {
    System.out.println("Token : " + st2.nextElement());
}

/* output */
Token : Hello
Token : World
Token : of
Token : Java
```





![](https://docs.google.com/drawings/d/sijmgJkEIbnFGuG4iSCIT-g/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=277&h=518&w=601&ac=1)





여기서 주의할 점은

**delimiter가 연속적으로 위치**하면, 

그 사이 값이 **null**로 나오지 않고 그냥 함께 버려진다는 것이다.

무슨 말이냐면,

위의 예에서

Hello <u>**null**</u> World of Java로 나오지 않고

Helo World of Java로 나온다는 뜻이다.





이런 예시를 들어보자.

`FirstName|LastName|Company|PhoneNumber`로 인적사항이 표시되어 있는 String이 있는데

우리는 위의 String을 tokenizing 해서 각각의 정보를 얻고 싶다.

그런데 여기서 무직인 사람은 세번째 Company에 입력이 안되어 있다고 하자.



그렇다면 `FirstName|LastName||PhoneNumber`와 같은 형식으로 String이 작성되고,

이것을 StringTokenizer에 넣어서 '|' 기준으로 tokenizing 하면,

위에서처럼 delimiter가 연속으로 붙어있기 때문에 그냥 같이 버려지고

FirstName, LastName, PhoneNumber 세 개의 값만 나오게 된다.

이러한 **문제점**은 아래에서 해결해보도록 하자.

<a href="#solution">[GO TO SOLUTION]</a>

문제점을 해결하기 전에 알아야 할 Method Signature가 있으니 이것부터 공부해보자.





**Method Signature**

```java
new StringTokenizer(String strToTokenize, String delims, boolean returnDelims);
// 세번째 파라미터 returnDelims가 true이면 delim도 같이 리턴한다.
```



```java
StringTokenizer st = new StringTokenizer("Hello, World|of|Java", ", |", true);
while(st.hasMoreElements()){
	System.out.println("Token: " + st.nextElement());
}
/* output */
Token: Hello
Token: ,
Token:  
Token: World
Token: |
Token: of
Token: |
Token: Java
```



세번째 인자를 **true**로 넘겨주면 아래의 그림과 같이 동작한다.



![](https://docs.google.com/drawings/d/s-D5syxGh3_ipkXBMXP4R4w/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=32&h=507&w=601&ac=1)







<h4 id="solution">
    SOLUTION
</h4>

```java
public class StrTok {
    public final static int MAXFIELDS = 4;
    public final static String DELIM = "|";


    public static String[] process(String line) {
        String[] results = new String[MAXFIELDS];

        StringTokenizer st = new StringTokenizer(line, DELIM, true);

        int i = 0;
        while(st.hasMoreTokens()) {
            String s = st.nextToken();
            if (s.equals(DELIM)) {
                if(++i >= MAXFIELDS)
                    throw new IllegalArgumentException("Input line " + line + " has too many fields");

                continue;
            }
            results[i] = s;
        }
        return results;
    }

    public static void printResults(String input) {
        System.out.println("Input: " + input);
        String[] outputs = process(input);

        int i = 0;
        for (String s : outputs) {
            System.out.println("Output " + i++ + " was: " + s);
        }
        System.out.println();
    }

    public static void main(String[] args) {
        printResults("A|B|C|D");
        printResults("|B|C|");
        printResults("A||C|D");
        printResults("|||");
        printResults("FirstName|LastName||PhoneNumber");
    }
}


/* output */
Input: A|B|C|D
Output 0 was: A
Output 1 was: B
Output 2 was: C
Output 3 was: D

Input: |B|C|
Output 0 was: null
Output 1 was: B
Output 2 was: C
Output 3 was: null

Input: A||C|D
Output 0 was: A
Output 1 was: null
Output 2 was: C
Output 3 was: D

Input: |||
Output 0 was: null
Output 1 was: null
Output 2 was: null
Output 3 was: null

Input: FirstName|LastName||PhoneNumber
Output 0 was: FirstName
Output 1 was: LastName
Output 2 was: null
Output 3 was: PhoneNumber
```







## StringBuilder로 String 이어 붙이기

**String**을 concatenation 하는 가장 쉬운 방법은 "+" operator를 이용하는 것이다.

컴파일러는 암묵적으로 **StringBuilder**를 만들어서 그 안의 **append()** 메서드를 이용할 수 있도록 최적화를 해주기도 한다.



> 참고: String 최적화 JDK 1.5
>
> https://gist.github.com/benelog/b81b4434fb8f2220cd0e900be1634753



컴파일러가 이 일을 처리하게 냅두지 말고 우리가 직접 해보자.





**StringBuilder**는 기본적으로 character들의 collection을 나타낸다.

**StringBuilder**는 **String**과 비슷하지만, 맨 위의 그림에서 나타나듯이,

**String**은 *immutable*(불변)하지만,

**StringBuilder**는 *mutable*하다.





> 참고로,
>
> **StringBuilder**와 **StringBuffer**의 차이점은
>
> **StringBuilder**는 synchronization(동기화)이 적용되지 않아 thread 환경에서 위험하고,
>
> **StringBuffer**는 synchronization(동기화)이 적용되어 thread 환경에서 안전하다.
>
> 
>
> 공통점은 두 클래스 모두 변경가능(mutable)하다.
>
> 두 클래스는 모두 **AbstractStringBuilder**라는 같은 부모 클래스의 자식 클래스고,
>
> 대부분의 메서드가 비슷하다.





아래의 예제를 보자.

```java
public class StringBuilderDemo {

    public static void main(String[] args) {
        String s1 = "Hello" + ", " + "World";
        System.out.println(s1);

        StringBuilder sb2 = new StringBuilder();
        sb2.append("Hello");
        sb2.append(',');
        sb2.append(' ');
        sb2.append("World");

        String s2 = sb2.toString();
        System.out.println(s2);


        // Now do the above all over again,
        // but in a more concise (and typical "real-world" Java) fashion
        System.out.println(
                new StringBuilder()
                        .append("Hello")
                        .append(',')
                        .append(' ')
                        .append("World")
        );
    }
}
```



- StringBuilder의 append 메서드는 StringBuilder 자신을 리턴하기 때문에 

  연쇄적으로 메서드를 호출할 수 있다.

  - append(), delete(), deleteCharAt(), insert(), replace, reverse() 등의 메서드 모두 자기 자신을 리턴한다.







두 번째 예제는 StringBuilder로 String을 CSV 형태로 만드는 예제다.

```java
public class StringBuilderCommaList {

    public static void main(String[] args) {
        // Method using regexp split
        StringBuilder sb1 = new StringBuilder();
        for (String word : "say hello to my lil friends".split(" ")) {
            if (sb1.length() > 0) {
                sb1.append(", ");
            }
            sb1.append(word);
        }
        System.out.println(sb1);

        // Method using a StringTokenizer
        StringTokenizer st = new StringTokenizer("say hello to my lil friends");
        StringBuilder sb2 = new StringBuilder();

        while(true) {
            sb2.append(st.nextToken());
            if (st.hasMoreTokens()) {
                sb2.append(", ");
            } else {
                break;
            }
        }
        System.out.println(sb2);
    }
}
```







## String 내부의 Character 접근



String을 글자 단위로 접근하려면 **charAt()** 메서드를 사용한다.

```java
public class StrCharAt {
    public static void main(String[] args) {
        String str = "hello!";
        for (int i = 0; i < str.length(); i++) {
            System.out.printf("Char %d is %c\n", i, str.charAt(i));
        }
    }
}
/* output */
Char 0 is h
Char 1 is e
Char 2 is l
Char 3 is l
Char 4 is o
Char 5 is !
```





```java
String str2 = "hello world!";
for (char c : str2) {
    System.out.println(c);
}
```

위와 같이 Enhanced For Loop을 사용할 때, 

`(char ch: String)`꼴을 사용하면 돌아갈 것 같지만, 안된다.



대신에 아래와 같이 String.toCharArray()를 사용해야 한다.

```java
String str2 = "hello world!";
for (char c : str2.toCharArray()) {
    System.out.println(c);
}
```