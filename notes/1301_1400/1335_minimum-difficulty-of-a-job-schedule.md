# 1335. Minimum Difficulty of a Job Schedule

You want to schedule a list of jobs in `d` days. Jobs are dependent (i.e, to work on the $i$<sup>th</sup> job, you have to finish all the jobs $j$ where $0 \leq j < i$).

You have to finish **at least** one task every day. The difficulty of a job schedule is the sum of difficulties of each day of the d days. The difficulty of a day is the maximum difficulty of a job done on that day.

You are given an integer array `jobDifficulty` and an integer `d`. The difficulty of the $i$<sup>th</sup> job is `jobDifficulty[i]`.

Return *the minimum difficulty of a job schedule*. If you cannot find a schedule for the jobs return `-1`.


**Example:**

> **Input:** `jobDifficulty = [7, 3, 9, 6, 4, 2, 9]`, `d = 3`
> 
> **Output:** `19`
> 
> **Explanation:** First day you can finish the first job, total difficulty = $7$.
>
> Second day you can finish the second job, total difficulty = $3$.
>
> Second day you can finish all other jobs, total difficulty = $9$.
> 
> The difficulty of the schedule $= 7 + 3 + 9 = 19$.


## Recursion/DP/DFS/Brute Force

The problem is effectively splitting an array into several segments, and try to minimise the sum of the maximum elements in each segment. To brute force it, we try all possible splits:

$[\underset{7}{\underbrace{7}}|\underset{3}{\underbrace{3}}|\ \underset{9}{\underbrace{9\ 6\ 4\ 2\ 9}}], [\underset{7}{\underbrace{7}}|\ \underset{9}{\underbrace{3\ 9}}\ |\ \underset{9}{\underbrace{6\ 4\ 2\ 9}}], [\underset{7}{\underbrace{7}}|\ \underset{9}{\underbrace{3\ 9\ 6}}\ |\ \underset{9}{\underbrace{4\ 2\ 9}}], \ldots, [\underset{9}{\underbrace{7\ 3\ 9\ 6\ 4}}\ |\underset{2}{\underbrace{2}}|\underset{9}{\underbrace{9}}]$

However, the #. of nested loops is equal to the #. of splits/days. Without knowing this beforehand, it's hard to write the code.[^nested loops] E.g. we may want `for i in xxx: for j in yyy` if we have two days, and want `for i in xxx: for j in yyy: for k in zzz` if we have three days. One solution is divide and conquer: we recursively split jobs into jobs to do today and jobs to be done later. To fix notation, let $dfs(i, d)$ be the optimal job difficulty when we have $d$ days remaining and have to finish all jobs from the $i$<sup>th</sup> onwards.

[^nested loops]: If you really want, you can. See [https://stackoverflow.com/q/64573917](https://stackoverflow.com/q/64573917).

* base case: $\displaystyle dfs(i, 1) = \max_{j\geq i} jobDifficulty_j$, i.e. when we only have one day left, we have to finish all remaining jobs, and the job difficulty for that day is the maximum difficulty of all remaining jobs
* recurrence relation: $\displaystyle dfs(i, d) = \max_{i\leq j\leq n_{jobs} - d} jobDifficulty_j + \min_{i\leq j\leq n_{jobs} - d} dfs(j, d - 1)$. That is, we have $d$ days left, and have to finish the $i$<sup>th</sup> job to the last job. Let's split the jobs into $i$<sup>th</sup> to $j$<sup>th</sup> job, and $j$<sup>th</sup> to the last job, and we do $i$<sup>th</sup> to $j$<sup>th</sup> job today, and do the remaining later. Today's job difficulty is the max. difficulty of $i$<sup>th</sup> to $j$<sup>th</sup> job, and the optimal job difficulty for the rest of days is $dfs(j, d - 1)$. We try all possible $j$’s and find the lowest job difficulty. Please note the upper bound of $j$ is #. of jobs minus days remaining. That is, if we have four days remaining, we must have at least four jobs untouched, so that all days can have at least one job scheduled, which is a requirement of this problem.


### Implementation

```c
int dfs(int* jobDifficulty, int jobDifficultySize,
        int start_job_idx, int days_remaining, int** dfs_cache) {
    // Check if the current DFS has already been computed in previous calls
    if (dfs_cache[start_job_idx][days_remaining] > INT_MIN) {
        return dfs_cache[start_job_idx][days_remaining];
    }

    // If only one day is left, have to finish all jobs today
    int min_diff = INT_MIN;
    if (days_remaining == 1) {
        for (int i = start_job_idx; i < jobDifficultySize; i++) {
            if (jobDifficulty[i] > min_diff) {
                min_diff = jobDifficulty[i];
            }
        }
        dfs_cache[start_job_idx][days_remaining] = min_diff;
        return min_diff;
    }

    // If we have two or more days left, split all jobs into jobs to be done
    // today and jobs for tomorrow or later. Loop over all possible choices
    // for today's last job
    int today_diff = INT_MIN;
    int future_diff;
    min_diff = INT_MAX;
    for (int i = start_job_idx;
         i < jobDifficultySize - days_remaining + 1;
         i++) {
        if (jobDifficulty[i] > today_diff) {
            today_diff = jobDifficulty[i];
        }
        future_diff = dfs(jobDifficulty, jobDifficultySize,
                          i + 1, days_remaining - 1, dfs_cache);
        if (future_diff + today_diff < min_diff) {
            min_diff = future_diff + today_diff;
        }
    }
    dfs_cache[start_job_idx][days_remaining] = min_diff;
    return min_diff;
}

int minDifficulty(int* jobDifficulty, int jobDifficultySize, int d) {
    // #. of job shall be greater than #. of day. Otherwise, some days will get
    // zero job and such schedule is not valid
    if (d > jobDifficultySize) {
        return -1;
    }

    // Init. DFS cache
    int** dfs_cache = malloc(jobDifficultySize * sizeof(*dfs_cache));
    for (int i = 0; i < jobDifficultySize; i++) {
        dfs_cache[i] = malloc((d + 1) * sizeof(*dfs_cache[i]));
        for (int j = 0; j < d + 1; j++) {
            dfs_cache[i][j] = INT_MIN;
        }
    }

    // Start with the first job, with d days remaining
    int ans = dfs(jobDifficulty, jobDifficultySize, 0, d, dfs_cache);
    free(dfs_cache);
    return ans;
}
```

We shall cache all `dfs` calls' results and reuse them if possible. Otherwise will lose a lot of time. E.g. $[7\ |\ 3\ 9\ |\ 6\ 4\ 2\ 9]$ and $[7\ 3\ |\ 9\ |\ 6\ 4\ 2\ 9]$ both use `dfs(3, 1)`, so we only have to calc. this once.


### How is this recursion related to DFS?[^dp to tree]

[^dp to tree]: [https://leetcode.com/problems/minimum-difficulty-of-a-job-schedule/solutions/924611/dfs-dp-progression-with-explanation-o-n-3d-o-nd](https://leetcode.com/problems/minimum-difficulty-of-a-job-schedule/solutions/924611/dfs-dp-progression-with-explanation-o-n-3d-o-nd).

(We don't have to think this as a DFS; simply treating this as a DP is totally fine.)

Tree is a very good way to visualise all possible splitting:

<img src="1335.svg" width="75%">

So if we have three days, then we DFS to nodes with depth $2$ and get the node with smallest difficulty. But again, no need to think this as a tree.
