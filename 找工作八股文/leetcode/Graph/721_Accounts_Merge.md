```cpp
class Solution {
public:
    vector<int> parent;

    // 初始化并查集
    void init(int n) {
        parent.resize(n);
        for (int i = 0; i < n; i++) {
            parent[i] = i; // 每个节点的初始父节点是自己
        }
    }

    // 找到根节点
    int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]); // 路径压缩
        }
        return parent[x];
    }

    // 合并两个节点
    void unionSet(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX != rootY) {
            parent[rootX] = rootY; // 将一个根指向另一个根
        }
    }

    vector<vector<string>> accountsMerge(vector<vector<string>>& accounts) {
        int n = accounts.size();
        init(n); // 初始化并查集，大小为账户数量

        // 映射邮箱到账户索引
        unordered_map<string, int> emailToIndex;
        for (int i = 0; i < n; i++) {
            for (int j = 1; j < accounts[i].size(); j++) { // 从第 1 个邮箱开始
                string email = accounts[i][j];
                if (emailToIndex.find(email) == emailToIndex.end()) {
                    emailToIndex[email] = i; // 如果邮箱未被映射，直接映射到当前账户
                } else {
                    // 如果邮箱已经被映射，合并两个账户
                    unionSet(i, emailToIndex[email]);
                }
            }
        }

        // 合并后的邮箱按根节点分组
        unordered_map<int, vector<string>> indexToEmails;
        for (unordered_map<string, int>::iterator it = emailToIndex.begin(); it != emailToIndex.end(); ++it) {
            string email = it->first;
            int index = find(it->second); // 找到邮箱对应账户的根
            if (indexToEmails.find(index) == indexToEmails.end()) {
                indexToEmails[index] = vector<string>();
            }
            indexToEmails[index].push_back(email);
        }

        // 构建结果
        vector<vector<string>> result;
        for (unordered_map<int, vector<string>>::iterator it = indexToEmails.begin(); it != indexToEmails.end(); ++it) {
            int index = it->first;
            vector<string> emails = it->second;
            sort(emails.begin(), emails.end()); // 按字母顺序排序邮箱
            emails.insert(emails.begin(), accounts[index][0]); // 在最前面插入账户名
            result.push_back(emails);
        }

        return result;
    }
};
```
