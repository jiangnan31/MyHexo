title: leetCode系列<一>
date: 2014-10-22 20:32:12
categories: Code
tags: LeetCode
---
## 1. Two Sum

```java
import java.util.HashMap;

/**
 * 
* @ClassName: TwoSumTest
* @add date  2011-03-13
* @Description: Given an array of integers, find two numbers such that they add up to a specific target number.
* The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.
* You may assume that each input would have exactly one solution.
* Input: numbers={2, 7, 11, 15}, target=9
* Output: index1=1, index2=2
* @author jiangnan04 
* @date 2014-10-22 下午08:44:15 
*
 */
public class TwoSumTest {

   
    public int[] twoSum(int[] numbers, int target) {
    	 HashMap<Integer, Integer> map = new HashMap<Integer, Integer>();  
         int n = numbers.length;  
         int[] result = new int[2];  
         for (int i = 0; i < numbers.length; i++)  
         {  
             if (map.containsKey(target - numbers[i]))  
             {  
                 result[0] = map.get(target-numbers[i]) + 1;  
                 result[1] = i + 1;  
                 break;  
             }  
             else  
             {  
                 map.put(numbers[i], i);  
             }  
         }  
         return result; 
    }
	public static void main(String[] args) {
		TwoSumTest t1 = new TwoSumTest();
		//int []numbers={2, 7, 11, 15};
		int []numbers={0, 4, 3, 0};
		//int [] numbers={5,75,25};
		int target=0;
		int[] reslut = t1.twoSum(numbers,target);
		System.out.println(reslut[0]+"  "+ reslut[1]);

	}

}
```