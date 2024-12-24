---
layout:     post
title:      "Sprint Planning: result analysis"
date:       2022-11-19 12:00:00 +0100
categories: agile
---

# Introduction

Two month ago I published a blog post about [velocity and accuracy in agile environments](/agile/2022/09/08/sprint-planning-velocity-accuracy) which included an experiment. Once it was finished I planned to analyse the results. This blog post will try to cover that.

# Data

The work was divided in several tasks and four of them were assigned to me. From those tasks two had subtasks. My idea is to first show the data regarding the  tasks, and then drill down into the subtasks.

## Tasks

The data of the tasks, where time is counted in hours.

| Estimated time | Time spent | Deviation |
| -------------- | ---------- | --------- |
| 4              | 5          | 25%       |
| 4              | 6          | 50%       |
| 42             | 58,5       | 39,29%    |
| 26             | 33,5       | 28,85%    |

As you can see the all the tasks incurred in a big deviation.

## Subtasks

The data from the subtasks of the third task.

| Estimated time | Time spent | Deviation |
| -------------- | ---------- | --------- |
| 8              | 8,50       | 6,25%     |
| 6              | 5          | -16,67%   |
| 8              | 4          | -50%      |
| 4              | 4          | 0%        |
| 8              | 9,50       | 18,75%    |
| 4              | 5,50       | 37,5%     |
| 4              | 4          | 0%        |
| Unexpected     | 18         | NA        |

In general term, the deviation is high. On top of that, there's an unexpected task.

The same data but from the subtasks of the fourth task.

| Estimated time | Time spent | Deviation |
| -------------- | ---------- | --------- |
| 4              | 2          | -50%      |
| 8              | 7          | -12,5%    |
| 4              | 4          | 0%        |
| 2              | 2          | 0%        |
| 4              | 12         | 0%        |
| 4              | 6,50       | 62,5%     |

This time the deviation is negative for the first two tasks and zero for the next three.

In general terms , although some of the subtasks (5) were completed on time, the deviations of the rest are higher than expected (double digits).

# Conclusion

The estimation improved as the project progressed, so we could say that knowledge improves the estimation. As an example, in the last task there is only one subtask with more time than estimated, and in the rest either the deviation is zero or negative. Moreover, not having done the task splitting would have been worse, as doing this exercise helped me understand what I had to do. Also, there was just one unexpected task. In general, increasing granularity helps, but once I get to estimates around eight hours I think I can stop splitting the work as the time invested in doing the initial analysis is higher than the time spent doing the actual work. Finally, I didn't take into account the time for the reviews and the fixes for that feedback.
