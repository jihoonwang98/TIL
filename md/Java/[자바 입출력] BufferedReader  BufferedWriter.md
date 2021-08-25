# [자바 입출력] BufferedReader / BufferedWriter



## 1. BufferedReader와 BufferedWriter

BufferedReader와 BufferedWriter는 이름처럼 버퍼를 이용해서 읽고 쓰는 함수입니다.

이 함수는 버퍼를 이용하기 때문에 이 함수를 이용하면 입출력의 효율성이 좋아집니다.



![](C:\Users\user\Desktop\1.PNG)

- 버퍼링 없이 키보드가 눌릴 때마다 눌린 문자의 정보를 목적지로 바로 이동시키는 것보다

  중간에 메모리 버퍼를 둬서 데이터를 한데 묶어서 이동시키는 것이 더 효율적이고 빠르다.



![](C:\Users\user\Desktop\2.PNG)





## 2. BufferedReader

BufferedReader의 `readLine()` 메서드를 사용하면 데이터를 라인 단위로 읽을 수 있다.

`readLine()` 메서드의 반환 타입은 `String`으로 고정되기 때문에 

`String`이 아닌 다른 타입으로 입력을 받으려면 Type Casting이 꼭 필요하다.



만약, 줄 단위로 읽지 않고 공백 단위로 데이터를 읽고 싶다면 어떻게 해야할 까?

이때는 `StringTokenizer`의 `nextToken()` 메서드를 이용하거나 

`String` 클래스의 `split()` 메서드를 사용한다.



### 예시

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

int n = Integer.parseInt(br.readLine());
br.close();
```



## 3. BufferedWriter

`BufferedWriter` 함수 또한 버퍼를 이용하기 때문에 성능 면에서 더 좋다.



`System.out.println()`처럼 문자열 출력과 개행을 동시에 해주지 않기 때문에

개행을 하려면 `write()` 메서드에 '\n'을 넣어주거나 `newLine()` 함수를 사용해야 한다.



### 예시

```java
BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

bw.write("hello\n");
bw.newLine(); // 개행
bw.write("I am writing\n");
bw.flush();
bw.close();
```













