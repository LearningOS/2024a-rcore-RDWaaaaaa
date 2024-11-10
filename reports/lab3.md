# Lab2总结

## 实现的功能
实现spawn，不经过fork，直接new了一个，没有复制父进程的地址空间。
它相较于fork+exec是更轻量级的。

## 问答题

1. 关于 p1 和 p2 的执行顺序
在 stride 算法中，每个进程有一个步长 stride 和一个传票数 pass，pass 值越小的进程越优先执行。当两个进程的 pass 都是 10，但 stride 值接近 BigStride（例如 p1.stride = 255，p2.stride = 250），每次执行后 pass 会增加各自的 stride。假设 p2 先执行一个时间片，p2.pass 将变为 10 + 250 = 260。理论上，下一个时间片应轮到 p1，因为它的 pass = 10 更小。

但由于 8 位无符号整数的溢出问题，260 会溢出为 4（即 260 - 256）。在这种情况下，p2.pass 变得比 p1.pass 小，p2 反而会再次被选中执行，因此实际情况是仍轮到 p2 而非 p1。

1. 为什么要求 STRIDE_MAX – STRIDE_MIN <= BigStride / 2
控制 stride 值的范围可以避免频繁的 pass 溢出。例如，对于 8 位无符号整数，如果 BigStride = 255，我们期望最大 stride 和最小 stride 的差距不超过 127，这样可以减少溢出带来的排序错误。当所有进程优先级均 >= 2 时，stride 值更小，使得进程之间的 pass 增长更平缓，不易因溢出导致执行顺序异常。

1. partial_cmp 函数补全代码
```rust
use core::cmp::Ordering;

struct Stride(u64);

impl PartialOrd for Stride {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        let max_stride = 255u64; // 假设我们使用 8 位无符号整数
        let diff = (self.0.wrapping_sub(other.0)) & max_stride;
        if diff < max_stride / 2 {
            Some(Ordering::Less)
        } else {
            Some(Ordering::Greater)
        }
    }
}

impl PartialEq for Stride {
    fn eq(&self, other: &Self) -> bool {
        false
    }
}
```

## 荣誉准则
1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

《你交流的对象说明》

2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

《你参考的资料说明》

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。
