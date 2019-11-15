# [LeetCode In Rust]026-Remove Duplicates From Sorted Array


<!--more-->

```rust
pub fn remove_duplicates(nums: &mut Vec<i32>) -> i32 {
    if nums.is_empty() { return 0 }
    let mut idx = 0;
    for i in idx .. nums.len() {
        if nums[i].gt(&nums[idx]) {
            idx += 1;
            nums.swap(i,idx);
        }
    }
    (idx + 1) as i32
}
```

