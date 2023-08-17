---
title: 剑指Offer | 从上到下打印二叉树 III
date: 2022-10-24
tags:
- 二叉树
- 广度优先搜索
categories:
- 数据结构与算法
- 剑指Offer
keywords:
description: 层序遍历
cover: https://s2.loli.net/2023/08/17/n2RJc3viyDadAht.jpg
---

# 题目描述

> 请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/

# 题目描述
## 层序遍历
``` C++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if (!root) return {};
        vector<vector<int>> res;
        queue<TreeNode*> q;
        q.push(root);
        bool isOrder = true;
        while (!q.empty()) {
            int curLevelSize = q.size();
            res.push_back(vector<int>(curLevelSize));
            for (int i = 0; i < curLevelSize; ++i) {
                TreeNode* node = q.front(); q.pop();
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
                if (isOrder) res.back()[i] = node->val;
                else res.back()[curLevelSize - 1 - i] = node->val;
            }
            isOrder = !isOrder;
        }
        return res;
    }
};
```
时间复杂度 O(n)，空间复杂度 O(n)