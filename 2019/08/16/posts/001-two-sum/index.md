# [LeetCode In Rust]001-Two Sum


<!--more-->

```rust
pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
    let map: HashMap<i32, usize> = nums.iter().enumerate().map(|(idx, &data)| (data, idx)).collect();

    nums.iter().enumerate().find(|(idx, &num)| {
        match  map.get(&(target - num)) {
            Some(&idx_in_map) => idx_in_map != *idx,
            None => false,
        }
    }).map(|(idx, &num)| vec![*map.get(&(target - num)).unwrap() as i32, idx as i32]).unwrap()

}

```



```rust
pub fn two_sum_v2(nums: Vec<i32>, target: i32) -> Vec<i32> {
    let map: HashMap<i32, usize> = nums.iter().enumerate().map(|(idx, &data)| (data, idx)).collect();

    for (i,&num) in nums.iter().enumerate() {
        match map.get(&(target - num) ) {
            Some(&x) => {
                if i != x {
                    return vec![i as i32, x as i32]
                }
            },
            None => continue,
        }
    }
    vec![]
}
```

