## 回文数
```cpp
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0) {
            return false;
        }

        long long reversed = 0;
        long long temp = x;

        while (temp != 0) {
            int digit = temp % 10;
            reversed = reversed * 10 + digit;
            temp /= 10;
        }

        return (reversed == x);
    }
};

```




```cpp
if (x < 0) {
            return false;
        }

        // 将整数转换为字符串
        string str = to_string(x);

        // 初始化左右指针
        int left = 0;
        int right = str.size() - 1;

        // 双指针比较
        while (left < right) {
            if (str[left] != str[right]) {
                return false; // 如果不相等，不是回文
            }
            left++;  // 左指针右移
            right--; // 右指针左移
        }

        return true;

```
