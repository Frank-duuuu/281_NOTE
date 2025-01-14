## climbing stairs, Leetcode 70
````C++
 int climbStairs(int n) {
        if(n == 0 || n == 1){return 1;}
        if(n == 2){return 2;}
        
        const size_t SIZE = n+1;
        vector<int> memo(SIZE,0);
        memo[0] = 1;memo[1] = 1;memo[2] = 2;
        for(size_t i = 3; i < n+1;i++){
            memo[i] = memo[i-1] + memo[i-2] ;
        }
        return memo[n];
    } 
};
````

## can jump out leetcode 55

* method I;(Problematic). 
times: < O(n*nums.size()), space: O(n)
use dp, vector<bool> memo. from back to the start and check whether ecery node's accessible range have true value(means able to reach the end)
 Problem : Too slow, somehow exceed the runtime limits in leetcode
 ````C++
 vector<bool> dp(nums.size(),false);
    dp[nums.size()-1] = true;
    
    for(int i = static_cast<int>(nums.size()-2); i >=0 ;i--){
        for(size_t j = i; j < min(static_cast<int>(nums.size()),nums[i] + 1 +i);j++){
            if(dp[j]){dp[i] = true;}
        }
    }
    
    
    return dp[0];
 ````
 
 * method II: 
 time: O(n) space:O(n)
 dynamic programming: vector<int> memo, traverse from the front, record the range each element can reach
 ````C++
 // Method II: dp from the front
    if(nums.empty()){return true;}
            if(nums[0] == 0){return nums.size() == 1;}
        vector<int> memo(nums.size(),nums[0]);
        for(size_t i = 1;i < memo.size();i++){
            memo[i] = max(memo[i-1] -1, nums[i]);
            //the maximum range this position can reach
            
            if(memo[i] == 0 && i!=nums.size()-1){return false;}
        }
            return true;
 ````
                                              
 * method III:
 dynamic programming from the back
 
 run from the back, chech whether each element can reach the good index(which can reach to the end)
 Time: O(n) space: O(1)
 ````C++
        if(nums.empty()){return true;}
        if(nums.size() == 1){return true;}
        if(nums[0] == 0){return nums.size() == 1;}
        
        int lastGoodIndex = nums.size() - 1;
        for(int i = static_cast<int>(nums.size() - 2);i >= 0;i--){
            if(nums[i] + i >= lastGoodIndex){
                lastGoodIndex = i;
            }//if
        }//for
    
    return lastGoodIndex == 0;
 ````                           

 `//leetcode 1155 throw dice to target sum`
 
 #### memo (suppose 6 faces)
 sum | 1 | 2 | 3 | 4 |5 |6 |7 |8
 --- | --- | --- | --- | --- | --- | --- | --- | ---
 0 dice| 0 | 0 | 0 | 0 | 0 | 0|0 | 0
 1 dice | 1| `1` | `1 `|`1` |`1 `|`1` | `0` | 0
 2 dice | 0 | 1 | 2 | 3 |4 |5 | 6 | `5` | 4
  
 
 `Tips` : 
 For every element in row, sum all upper left numbers(within the range all faces)
 
 For example, for highlighted number 5(target sum 9), we cannot roll from 1 dice target sum 7 as #cases is 0.
 but we can roll from 1 dice target sum 6 and roll another dice as 2, #cases is 1. sum up all cases resulting in 5
 
 ````C++
 int numRollsToTarget(int n, int k, int target) {
        const size_t num_row = size_t(n+1); const size_t num_col = size_t(target+1);
    vector<vector<long long int>> memo(num_row, vector<long long int>(num_col,0));
    
    //base case initialize the 1 dice situation
    for(int col = 1;col < min(k+1,static_cast<int>(num_col));col++){
        //for one dice, there is only 1 way for given number within the range
        memo[1][col] = 1;
    }
    
    //start from the row index 2
    for(int i = 2;i < num_row;i++){
        for(int j = 1;j < num_col;j++){
            //for each element sumation all situation
            for(int x = 1; x <= k && x < j; x++){
                memo[i][j] += memo[i-1][j-x];
            }//for each possibility
            
        }//col
        
    }//row
    
    
    //result is at the right bottom conenr
    return memo[n][target];
    }
 ````
      
 
 
 ## LeetCode 72 Edit distance
 
 
 
 <img width="520" alt="image" src="https://user-images.githubusercontent.com/81163933/174463563-5eb5d62e-b952-46ec-96b5-6c2f08aad64b.png">

 Suppose we want to convert s1 = bunny to s2 = banana
 
 ![IMG_0890](https://user-images.githubusercontent.com/81163933/174463556-0ca55c4f-b418-4226-ab0a-b81b02955840.jpg)

 * index means that how many characters I take from that string
 * suppose i = 2, j = 2, that block means number of  ways I have to convert string "bu" to string "ba"
 * Two cases: 
  1.s2[j] == s1[i] ->  memo[i][j] = min(memo[i-1][j-1], memo[i-1][j] + 1, memo[i][j-1] + 1);
  2.s2[j] != s1[i] -> memo[i][j] = min(memo[i-1][j-1] + 1, memo[i-1][j] + 1, memo[i][j-1] + 1);
 
 For example: 
 Suppose we are at [2][2], we want to calculate number of convertions from "bu" to "ba", 
  1.From [1][1], we already know number of convertion from b to b, for b to ba, we can add 1 insertion to insert a and thus need +1
  2.From [2][1], we already know number of convertion from bu to b, for bu to ba, we can add 1 insertion to insert a after we convert bu to b and thus    need +1
 3. From [1][2], we already know number of convertion from b to ba, for bu to ba, we can add 1 deletion, after we convert `b`u to `ba`u, we simply delete u and get ba.
 
 ````C++
 int minDistance(string word1, string word2) {
    if(word1.empty()){return static_cast<int>(word2.size());}
    if(word2.empty()){return static_cast<int>(word1.size());}
    
    vector<vector<int>> memo(word1.size() + 1, vector<int>(word2.size() + 1,0));
    //initialize the base case
    for(int col = 1; col < word2.size()+1;col++ ){
        memo[0][col] = col;
    }
    for(int row = 1; row < word1.size()+1;row++ ){
        memo[row][0] = row;
    }
    
    //traverse through the memo
    for(size_t i = 1;i < word1.size() + 1;i++){
        for(size_t j = 1;j < word2.size() + 1; j++){
            if(word1[i-1] == word2[j-1])
                memo[i][j] = min({memo[i-1][j] + 1,memo[i-1][j-1],memo[i][j-1] + 1});
            else
                memo[i][j] = 1 + min({memo[i-1][j],memo[i-1][j-1],memo[i][j-1]});
        }//for
    }//for
    
    return memo[word1.size()][word2.size()];
    
}
 ````
