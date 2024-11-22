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
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0) {
            return false;
        }

        string str = to_string(x);

        int left = 0;
        int right = str.size()-1;


        while(left < right){
            if(str[left] != str[right]){
                return false;
            }
            left ++;
            right --;
        }
        return true;
    }
};

```
