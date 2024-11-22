## two sum  没啥好说的
暴力解法
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        for(int i = 0; i < nums.size(); i ++){
            for( int j = i + 1; j < nums.size(); j ++){
                if(nums[i] + nums[j] == target){
                    return{i, j};
                }
            }
        }
        return {};
    }
};


```
