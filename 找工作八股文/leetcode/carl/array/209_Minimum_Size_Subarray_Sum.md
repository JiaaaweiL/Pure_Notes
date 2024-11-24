```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
       
        int result = INT32_MAX;
        int sum = 0; 
        int left = 0;

        for (int right = 0; right < nums.size(); right++) {
            sum += nums[right]; // 增加右边界的值
            while (sum >= target) { // 如果当前窗口的和满足条件
                result = min(result, right - left + 1); // 更新最小长度
                sum -= nums[left]; // 移动左边界
                left++;
            }
        }

        return result == INT32_MAX ? 0 : result; // 如果没有子数组满足条件，返回 0

    }
};







```
