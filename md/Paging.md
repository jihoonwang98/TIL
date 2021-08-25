# Paging 1편



1편에서는 Paging 처리를 구현하는 코드를 작성하기 전에

변수들 간의 **관계식**과 **Test Code**를 작성해본다.





## 용어 정의

![](https://docs.google.com/drawings/d/sFa-NsUipdEeG7GrxpN7T4g/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=2057&h=444&w=601&ac=1)



![](https://docs.google.com/drawings/d/sBJWgjnPOgvc0O14uVCY-EA/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=174&h=288&w=601&ac=1)



![](https://docs.google.com/drawings/d/sBFVUbxif4moQ3mtC214vjw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=459&h=314&w=601&ac=1)



![](https://docs.google.com/drawings/d/sm70MnzKiRhL7Oxe-uN8qXQ/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=52&h=211&w=601&ac=1)







## 관계식 정의

아래의 값들은 이미 알고 있거나 정의되었다고 가정한다:

**totalPosts(모든 글 개수)**, **currentPage(현재 페이지 번호), postsPerPage(한 페이지당 표시할 글 개수), displayPageNum(한 번에 표시할 페이지 개수)**



페이징을 구현하기 위해 구해야 할 값:

**prev, next, startPage, endPage**

4가지!!





우선, **prev**는 **currentPage**로 구한 **startPage**가 1인 경우 *false*고, 나머지 경우는 모두 *true*.

**<u>prev = (startPage == 1) ? false : true</u>**



그리고, **next**는 **currentPage**로 구한 **endPage**가 **totalPages(모든 페이지 개수)**인 경우 *false*고, 나머지 경우는 *true*

**<u>next = (endPage == totalPages) ? false : true</u>**





즉, **prev**와 **next** 값은 **startPage, endPage, totalPages**에 의존하므로 

이 3가지 값을 먼저 구해보자.







### totalPages (총 page 개수)

**total page 개수**는 **total post 개수**와 **postsPerPage**의 값에 의존한다.



쉬운 예부터 들어보자면,



#### [EX1 나눠 떨어지는 경우]

**totalPosts = 10, postsPerPage = 2**이면,

**totalPages = 5**





#### [EX2 나눠 떨어지는 경우]

같은 원리로, **totalPosts = 300, postsPerPage = 10**이면,

**totalPages = 30**





#### [EX3 나눠 떨어지지 않는 경우]

자, 그러면 이제 **totalPosts**가 **postsPerPage**로 나눠 떨어지지 않는 경우는 어떨까?

**totalPosts = 13, postsPerPage = 3**이면,

**totalPages = 5**





#### 관계식 구하기

그럼 **totalPages**는 **totalPosts**와 **postsPerPage**를 이용해서 어떻게 나타낼 수 있을까?

바로 위의 예를 참고하면,

**totalPages = totalPosts / postsPerPage + 1** 인듯 보인다.

하지만 위의 식은 딱 <u>나눠 떨어지는 경우 성립하지 않는다</u>.





**postsPerPage**를 고정시켜 놓고 생각해보자.

**postsPerPage**가 5인 경우,

**totalPosts**가 1, 2, 3, 4, 5일 때 **totalPages**는 1이 나온다.

**totalPosts**가 6, 7, 8, 9, 10일 때 **totalPages**는 2가 나온다.

...



쭉 나열 해보면 1 ~ 5가 한 묶음, 6 ~ 10이 한 묶음, ... 해서 쭉 간다는 걸 알 수 있다.

그럼 얘네들을 어떻게 수학적으로 한 묶음임을 나타낼 수 있을까?

5개를 기준으로 한묶음씩 나뉘니깐 **나눗셈**을 활용해보자.



1, 2, 3, 4는 5로 나누면 몫이 0이지만,

5는 5로 나누면 몫이 1이 된다.

나누어 떨어지는 경우에 문제가 생긴다. 



이를 해결하기 위해 **totalPosts** 값에 1을 빼주면,

1, 2, 3, 4, 5는 각각 0, 1, 2, 3, 4에 대응되어

5로 나눈 몫이 모두 0이 되어 **"5로 나눈 몫이 0"**이라는 집합으로 묶을 수 있게 되었다.



6, 7, 8, 9, 10도 마찬가지로 전부 1씩 빼주면 **"5로 나눈 몫이 1"**이라는 집합으로 묶을 수 있게 된다.



이때, 5로 나눈 몫에 1을 더해주면 total 페이지 개수가 나오므로 이를 수식으로 정리하면,

**<u>totalPages =  ( (totalPosts - 1) / postsPerPage ) + 1</u>**이다. (이때,  /는 '몫' 연산자임에 주의)











### endPage (마지막 페이지 번호)



여기서 **endPage(마지막 페이지 번호)**란 <u>현재 페이지의 번호들 중 마지막 번호</u>란 뜻이다. **totalPages**(총 페이지 개수)와 헷갈리지 말자.



![](https://docs.google.com/drawings/d/sgcTc6qMS3UR_Ol0w0y3nJQ/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=419&h=211&w=601&ac=1)





**endPage**는 **currentPage**, **displayPageNum**, **totalPages** 세 가지 값에 의존한다.



예를 들어보자면,



#### [EX1]

**currentPage = 7, displayPageNum = 10, totalPages = 40**이면,

**endPage = 10**



#### [EX2]

**currentPage = 7, displayPageNum = 10, totalPages = 9**이면,

**endPage = 9**



#### [EX3]

**currentPage = 17, displayPageNum = 4, totalPages = 20**이면,

**endPage = 20**



#### [EX4]

**currentPage = 34, displayPageNum = 10, totalPages = 37**이면,

**endPage = 37**



#### [EX5]

**currentPage = 17, displayPageNum = 4, totalPages = 19**이면,

**endPage = 19**





복잡하니깐 **currentPage**와 **displayPageNum** 둘의 관계부터 일단 살펴보면,

**currentPage = 17, displayPageNum = 4**이면,

Page 번호: 1, 2, 3, 4가 한 번에 출력,

Page 번호: 5, 6, 7, 8이 한 번에 출력,

Page 번호: 9, 10, 11, 12가 한 번에 출력,

Page 번호: 13, 14, 15, 16이 한 번에 출력,

Page 번호: 17, 18, 19, 20이 한 번에 출력되므로

**endPage**는 20이라고 할 수 있다. (**totalPages**는 고려하지 않았을 때)



근데 여기에도 위에서 **totalPages**를 구할 때 발견했던 규칙이 또 있다.

이번 예시에서도 **currentPage**를 **displayPageNum**으로 나눈 몫에 따라 규칙이 생기지만 

나누어 떨어질때만 규칙이 깨지니깐 위에서처럼 **currentPage**에다가 1빼고 나눠주면 규칙성이 생긴다.

즉, 아래의 밑줄친 부분 말이다. 여기다가 **displayPageNum**을 곱해주면 우리가 구하는 **endPage** 관계식이 도출된다.



**endPage = (<u>( (currentPage - 1) / displayPageNum ) + 1 )</u> * displayPageNum **





그런데 만약 **totalPages**가 이 **endPage**보다 작은 경우(**[EX 2, 4, 5]가 여기에 해당**)는 어떡할까?

이럴 때만 위의 식으로 구한 **endPage** 대신 **totalPages**로 바꿔주면 된다.



즉, 아래의 코드와 같다.

```java
int endPage = (((currentPage-1)/displayPageNum)+1) * displayPageNum;

if(totalPages < endPage) 
    endPage = totalPages;
```







#### startPage (시작 페이지 번호)

**startPage**도 **endPage**와 비슷한 원리로 구할 수 있다.

관계식은 다음과 같다.

**startPage = ((currentPage-1)/displayPageNum) * displayPageNum + 1**









### 관계식 총정리

- **totalPages =  ( (totalPosts - 1) / postsPerPage ) + 1**

- ```java
  int endPage = (((currentPage-1)/displayPageNum)+1) * displayPageNum;
  
  if(totalPages < endPage) 
      endPage = totalPages;
  ```

- **startPage = ((currentPage-1)/displayPageNum) * displayPageNum + 1**

- **prev = (startPage == 1) ? false : true**

- **next = (endPage == totalPages) ? false : true**







### Test Codes



위에서 구한 관계식들로 **Test Code**를 작성해보자..

좀 읽기 힘들어서 리팩터링을 해야 할 것 같은데 나중에...

```java
package com.tistory.kgvovc.simpleTest;

import lombok.Getter;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import org.junit.Test;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

public class PagingTest {

    @Test
    public void testTotalPages() {
        assertEquals(totalPages(10, 2), 5);
        assertEquals(totalPages(300, 10), 30);
        assertEquals(totalPages(13, 3), 5);
        assertEquals(totalPages(17, 4), 5);
        assertEquals(totalPages(16, 4), 4);
        assertEquals(totalPages(15, 4), 4);
    }

    @Test
    public void testEndPage() {
        assertEquals(endPage(7, 10, 40), 10);
        assertEquals(endPage(7, 10, 9), 9);
        assertEquals(endPage(17, 4, 20), 20);
        assertEquals(endPage(34, 10, 37),37);
        assertEquals(endPage(17, 4, 19), 19);
        assertEquals(endPage(17, 4, 18), 18);
        assertEquals(endPage(17, 5, 20), 20);
        assertEquals(endPage(17, 5, 19), 19);
        assertEquals(endPage(1, 1, 1), 1);
        assertEquals(endPage(4, 3, 5), 5);
        new PageInfo(10, 2, 4, 3);
        assertEquals(endPage(2, 3, totalPages(10, 4)), 3);
    }

    @Test
    public void testStartPage() {
        assertEquals(startPage(7, 10), 1);
        assertEquals(startPage(17, 10), 11);
        assertEquals(startPage(18, 3), 16);
        assertEquals(startPage(17, 3), 16);
        assertEquals(startPage(32, 3), 31);
    }


    @Test
    public void testHasPrev() {
        assertFalse(hasPrev(startPage(7, 10)));
        assertFalse(hasPrev(startPage(2, 5)));
        assertFalse(hasPrev(startPage(9, 10)));
        assertFalse(hasPrev(startPage(7, 8)));
        assertFalse(hasPrev(startPage(1, 1)));

        assertTrue(hasPrev(startPage(3, 2)));
        assertTrue(hasPrev(startPage(2, 1)));
        assertTrue(hasPrev(startPage(32, 3)));
    }

    @Test
    public void testHasNext() {
        assertFalse(hasNext(endPage(5, 8, 7), 7));
        assertFalse(hasNext(endPage(7, 10, 9), 9));
        assertFalse(hasNext(endPage(32, 10, 40), 40));
        assertFalse(hasNext(endPage(32, 10, 38), 38));

        assertTrue(hasNext(endPage(5, 8, 34), 34));
        assertTrue(hasNext(endPage(1, 4, 8), 8));
    }


    @Test
    public void testAll() {
        printPages(new PageInfo(10, 2, 4, 3));
        printPages(new PageInfo(15, 2, 4, 3));
        printPages(new PageInfo(13, 4, 3, 3));
        printPages(new PageInfo(190, 4, 10, 10));
        printPages(new PageInfo(190, 11, 10, 10));
        printPages(new PageInfo(134, 11, 10, 10));
        printPages(new PageInfo(180, 8, 5, 7));
        printPages(new PageInfo(180, 15, 5, 7));
        printPages(new PageInfo(180, 36, 5, 7));
    }

    private int totalPages(int totalPosts, int postsPerPage) {
        return ((totalPosts - 1) / postsPerPage) + 1;
    }

    private int endPage(int currentPage, int displayPageNum, int totalPages) {
        int endPage = (((currentPage-1)/displayPageNum)+1) * displayPageNum;

        if(totalPages < endPage)
            endPage = totalPages;

        return endPage;
    }

    private int startPage(int currentPage, int displayPageNum) {
        return ((currentPage - 1) / displayPageNum) * displayPageNum + 1;
    }

    private boolean hasPrev(int startPage) {
        return (startPage == 1) ? false : true;
    }

    private boolean hasNext(int endPage, int totalPages) {
        return (endPage == totalPages) ? false : true;
    }

    private void printPages(PageInfo info) {
        int totalPages = totalPages(info.getTotalPosts(), info.getPostsPerPage());
        int startPage = startPage(info.getCurrentPage(), info.getDisplayPageNum());
        int endPage = endPage(info.getCurrentPage(), info.getDisplayPageNum(), totalPages);
        boolean hasPrev = hasPrev(startPage);
        boolean hasNext = hasNext(endPage, totalPages);

        if(hasPrev) {
            System.out.print("<< ");
        }
        for (int page = startPage; page <= endPage; page++) {
            System.out.print(page + " ");
        }
        if(hasNext){
            System.out.print(">> ");
        }
        System.out.println();
    }


    @Getter
    @Setter
    @RequiredArgsConstructor
    static class PageInfo {

        private final int totalPosts;      // 모든 글 개수
        private final int currentPage;     // 현재 페이지 번호
        private final int postsPerPage;    // 한 페이지당 표시할 글 개수
        private final int displayPageNum;  // 한 번에 표시할 페이지 개수
    }
}
```



어쨌든 잘 돌아간다.







### 직접 구현한 페이지



![](https://lh5.googleusercontent.com/gKjMXb9B5MGnAGJX5B_Xpji0JbQLs-eSYfS4_3nGbt-sjDxVd23n5vjBoLMDihOmL7J89lI-kmsUP3qiPT_HHg2chlZ6el25GlkoS5yulUVxU-wzUTkyleE74Ok26TVfuIbFKDsk)



![](https://lh4.googleusercontent.com/-8AqxsBXTfE-NShGid0wyKeUe7-MSCuhB6B_JnusB5fM_cmi0ClqDLn0F-xogmTVI5HliGGn2jZK4j0WxQPg1j4rldUaNWYbLPuAl00YMLR4H_30OWIPX6ET0ln3BN3BLhzFNjuF)



:grinning: