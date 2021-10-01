# RUSTLINGS学习笔记

## 1. move_semantics2

这道题才让我感到RUST和C之间巨大的差异。对于Vector的引用，编译器直接禁止了对其使用 * 运算符，因为Vector不能copy（更细节的我还不懂）。我尝试过很多方法，无一例外都需要使用 * ，所以都失败了。查看hint后，提示给出了清晰的解决方法：

1. Make another, separate version of the data that's in `vec0` and pass that
   to `fill_vec` instead.
2. Make `fill_vec` borrow its argument instead of taking ownership of it,
   and then copy the data within the function in order to return an owned
   `Vec<i32>`
3. Make `fill_vec` *mutably* borrow its argument (which will need to be
   mutable), modify it directly, then not return anything. Then you can get rid
   of `vec1` entirely -- note that this will change what gets printed by the
   first `println!`

第三种仅仅是让编译通过，所以我选择第二种实现。

```
fn main() {
    let mut vec0 = Vec::new();
    let mut vec1 =  fill_vec(& vec0);

    // Do not change the following line!
    println!("{} has length {} content `{:?}`", "vec0", vec0.len(), vec0);

    vec1.push(88);

    println!("{} has length {} content `{:?}`", "vec1", vec1.len(), vec1);
}

fn fill_vec(vec: & Vec<i32>) ->  Vec<i32>  {
    //let mut vec = vec;
    let mut vec=my_copy(vec);
    vec.push(22);
    vec.push(44);
    vec.push(66);

    vec
}

fn my_copy(vec: & Vec<i32>) -> Vec<i32>{
    let mut new_vec = Vec::new();
    for i in vec {
        new_vec.push(*i);
    }
    new_vec
}
```

这里的for循环也应该好好思考一下，i仍然是引用而不是向量的值。

## 2. move_semantics5

RUST对于引用的操作限制非常严格，而且最开始我也理解错了题目的意思。想着既然同时出现了两个引用，那么这两个引用一定是immutable的，但是后面引用是被修改的，又要求一定是mutabel。

其实这道题考的是修改代码行的位置，使得先修改，再引用，就能避免原引用被回收掉的问题。

## 3. errors3

若设置main的返回值为Result<i32, ParseIntError>，则会产生下面的报错：

```
error[E0277]: `main` has invalid return type `Result<i32, ParseIntError>`
  --> exercises/error_handling/errors3.rs:10:14
   |
10 | fn main() -> Result<i32, ParseIntError> {
   |              ^^^^^^^^^^^^^^^^^^^^^^^^^^ `main` can only return types that implement `Termination`
   |
   = help: consider using `()`, or a `Result`

```

这时，可以模仿errors5.rs的写法，将main的返回值改成Result<(), ParseIntError>，就可以通过；

或者直接把**let cost = total_cost(pretend_user_input)?;**改成**let cost = total_cost(pretend_user_input).unwrap();**

## 4. generics2

这道题我卡在了如何在结构中定义泛型的地方，看编译器的报错一直看不懂，对照圣经才发现定义写错了。。。

```
struct Wrapper<T> {
    value: T,
}

impl<T> Wrapper<T> {
    pub fn new (value: T) -> Self {
        Wrapper { value }
    }
}
```

## 5. generics3

编译器出现了下面这样的报错，看情况应该是编译器无法确定泛型T能否被打印。幸运的是，编译器提示了应该怎么做，并且成功通过。编译器真的绝绝子！

```
error[E0277]: `T` doesn't implement `std::fmt::Display`
  --> exercises/generics/generics3.rs:24:52
   |
24 |             &self.student_name, &self.student_age, &self.grade)
   |                                                    ^^^^^^^^^^^ `T` cannot be formatted with the default formatter
   |
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: this error originates in the macro `$crate::__export::format_args` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider restricting type parameter `T`
   |
21 | impl<T: std::fmt::Display> ReportCard<T> {
   |       ^^^^^^^^^^^^^^^^^^^

```

## 6. option1



## 7. option2

圣经里if let控制流那个章节读的太简略了，导致看到题目都没怎么看明白。if let做的其实就是match option的工作，下面的那个while let也可以同等的理解，但是要注意的是array中的每一个元素也都是option，所以这里是有一个嵌套的。
