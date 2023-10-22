---
layout: post
title:  "Movement of Robots"
date:   2023-10-23
desc: "Calculating final state of a set of moving robots optimally"
keywords: "C++,Leetcode,DSA,Easy"
categories: [prog]
permalink: /prog/mor/
tags: [C++,Leetcode,Easy]
icon: icon-code
---

## Problem Statement

Some robots are standing on an infinite number line with their initial coordinates given by a 0-indexed integer array nums and will start moving once given the command to move. The robots will move a unit distance each second.

You are given a string s denoting the direction in which robots will move on command. 'L' means the robot will move towards the left side or negative side of the number line, whereas 'R' means the robot will move towards the right side or positive side of the number line.

If two robots collide, they will start moving in opposite directions.

Return the sum of distances between all the pairs of robots d seconds after the command. Since the sum can be very large, return it modulo `10^9 + 7`.

Note:

* For two robots at the index i and j, pair (i,j) and pair (j,i) are considered the same pair.
* When robots collide, they instantly change their directions without wasting any time.
* Collision happens when two robots share the same place in a moment.
  * For example, if a robot is positioned in 0 going to the right and another is positioned in 2 going to the left, the next second they'll be both in 1 and they will change direction and the next second the first one will be in 0, heading left, and another will be in 2, heading right.
  * For example, if a robot is positioned in 0 going to the right and another is positioned in 1 going to the left, the next second the first one will be in 0, heading left, and another will be in 1, heading right.

## Intuition

The first intuition was all collision events that can occur are actually non-events when looking at the collective robot system. **In simpler terms, instead of assuming the robots collide and change directions, you can just assume the robots don't collide and "pass through" each other.**

## Approach

* Keeping the above intuition in mind, we can assume each robot is either moving `d` distance to the left or right depending on the direction of it's movement.
* So after `d` seconds we can say the ith robot will move to either `nums[i]+d` if direction is `R` or `nums[i]-d` if it's `L`.
* If you sort the final positions and consider distance between each consecutive robot, this value will be considered exactly once when calculating distance of any pair with one robot to it's left and one to it's right.
* Going by the above observation, consecutive distance `f[i] - f[i-1]` (`f` represents the final positions) will be considered once in `i*(n-i)` pairs where n is the total number of robots. So we can add `(f[i] - f[i-1])*(i*(n-i))` for each consecutive distance.

## Source

```c++
class Solution {
private:
    int MOD = 1000000007;
public:
    int sumDistance(vector<int>& nums, string s, int d) {
        long long int dist = 0LL;
        vector<long long int> f;
        // Calculate final positions
        for(int i=0;i<nums.size();++i) {
            if(s[i] == 'R') f.push_back(nums[i] + d);
            else f.push_back(nums[i] - d);
        }
        // Sort final positions
        sort(f.begin(), f.end());
        for(int i=1;i<f.size();++i) {
            // Add contribution of each consecutive distance
            dist += ((f[i] - f[i-1])*((i*(f.size()-i))%MOD))%MOD;
            dist %= MOD;
        }
        return dist;
    }
};
```

## Complexity

* Overall time complexity: `O(NlogN)`
* Space complexity: `O(N)` extra space to store final positions