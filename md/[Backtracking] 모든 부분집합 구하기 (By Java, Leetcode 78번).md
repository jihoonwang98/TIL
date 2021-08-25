# 모든 부분집합 구하기 (By Java, Leetcode 78번)



> 문제 출처: https://leetcode.com/problems/subsets/



정수 배열 `nums`가 주어졌을 때, `nums`의 **power set**을 구하는 문제.



**Ex1:**

```
Input: nums = [1,2,3]
Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```



**Ex2:**

```
Input: nums = [0]
Output: [[],[0]]
```





## 첫 번째 풀이 (Cascading)



![](https://leetcode.com/problems/subsets/Figures/78/recursion.png)





### 풀이 IDEA

[기존의 결과를 이용하는 풀이]



![](https://docs.google.com/drawings/d/sUWhtTWIwATDAsW-DS6JAjg/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=366&drawingRevisionAccessToken=d3ZKC7NjFmmnPA&h=334&w=601&ac=1)



위의 그림을 보면 알 수 있듯이,

요소 n개의 모든 subset들을 원소로 가지는 리스트를 L(n)이라고 하면

L(n) = L(n-1) + L(n-1).forEach(list -> list.add(n번째 요소))이다.



즉, DP처럼 n-1번째 결과를 통해 n번째 결과를 만들어낼 수 있는 것이다.





이를 코드로 작성해보면 아래와 같다.

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> output = new ArrayList();
        output.add(new ArrayList<Integer>());

        for (int num : nums) {
            List<List<Integer>> newSubsets = new ArrayList();
            for (List<Integer> curr : output) {
                newSubsets.add(new ArrayList<Integer>(curr) {
                    {add(num);}
                });
            }
            for (List<Integer> curr : newSubsets) {
                output.add(curr);
            }
        }
        return output;
    }
}
```



위의 코드가 잘 이해가 안 간다면 아래 그림을 보자.

아래 그림은 `int[] nums = new int[] {1, 2, 3};`인 경우다.





![](https://docs.google.com/drawings/d/sQwylGiLxjvMsx-3fwi_iCw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=220&drawingRevisionAccessToken=sVCU5F2H6vJqqg&h=358&w=601&ac=1)





![](https://docs.google.com/drawings/d/sAMwz3LFaMPaRdU9-GtO34Q/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=34&drawingRevisionAccessToken=cNR79iYiZGRREQ&h=358&w=601&ac=1)





![](https://docs.google.com/drawings/d/sMDdjrT-6bMGsc0KAykIwVQ/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=67&drawingRevisionAccessToken=Mdre7OX3YKEolg&h=358&w=601&ac=1)







### 시간 복잡도 

$$
O(N*2^N)
$$



증명은 잘 모르겠다.

N은 새로운 ArrayList를 만들고 내용물을 복사하는 과정에서 곱해지는 것 같다.









## 두 번째 풀이 (Backtracking)



> **Backtracking**이란?
>
> 백트래킹이란 모든 경우의 수를 따져서 solution을 찾는 방법인데,
>
> 찾다가 solution이 될 가능성이 없는 경우,
>
> 해당 case를 버리고 그 전 step에서 다시 조치를 취한다. 



![](https://leetcode.com/problems/subsets/Figures/78/combinations.png)



Power set의 정의를 이용한 풀이다.



> **[Power set의 정의]**
>
> Power set is all possible combinations of all possible *lengths*, from 0 to n.





```java
class Solution {
    List<List<Integer>> output = new ArrayList();
    int n, k;

    public void backtrack(int first, ArrayList<Integer> curr, int[] nums) {
        // if the combination is done
        if (curr.size() == k) {
            output.add(new ArrayList(curr));
            return;
        }
        for (int i = first; i < n; ++i) {
            // add i into the current combination
            curr.add(nums[i]);
            // use next integers to complete the combination
            backtrack(i + 1, curr, nums);
            // backtrack
            curr.remove(curr.size() - 1);
        }
    }

    public List<List<Integer>> subsets(int[] nums) {
        n = nums.length;
        for (k = 0; k < n + 1; ++k) {
            backtrack(0, new ArrayList<Integer>(), nums);
        }
        return output;
    }
}
```







## 세 번째 풀이



![](https://leetcode.com/problems/subsets/Figures/78/bitmask4.png)





```java
class Solution {
  public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> output = new ArrayList();
    int n = nums.length;

    for (int i = (int)Math.pow(2, n); i < (int)Math.pow(2, n + 1); ++i) {
      // generate bitmask, from 0..00 to 1..11
      String bitmask = Integer.toBinaryString(i).substring(1);

      // append subset corresponding to that bitmask
      List<Integer> curr = new ArrayList();
      for (int j = 0; j < n; ++j) {
        if (bitmask.charAt(j) == '1') curr.add(nums[j]);
      }
      output.add(curr);
    }
    return output;
  }
}
```





## 네 번째 풀이



```java
public class SubsetAlgorithm {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> list = new ArrayList<>();
        // Arrays.sort(nums);
        backtrack(list, new ArrayList<>(), nums, 0);
        return list;
    }

    private void backtrack(List<List<Integer>> list , List<Integer> tempList, int [] nums, int start){
        list.add(new ArrayList<>(tempList));
        for(int i = start; i < nums.length; i++){
            tempList.add(nums[i]);
            backtrack(list, tempList, nums, i + 1);
            tempList.remove(tempList.size() - 1);
        }
    }
}
```

