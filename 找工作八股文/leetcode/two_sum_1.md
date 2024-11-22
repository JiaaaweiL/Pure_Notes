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
哈希表解法
num_map.count(complement) 是查找key存在与否，如果存在，用num_map[complement]返回index


```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> num_map;
        for(int i = 0; i < nums.size(); i++){
            num_map[nums[i]] = i;
        }

        for(int i = 0; i < nums.size(); i++){
            int complement = target - nums[i];
            if(num_map.count(complement) && num_map[complement] != i){
                return {i, num_map[complement]};
            }

        }
        return {};
    }
};

```
