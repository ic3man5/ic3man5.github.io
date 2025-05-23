---
layout: post
title:  "Modern Error Handling in Rust and C++23"
date:   2025-05-22 15:52:00 -0400
categories: rust c++
---

Introduction
-------------

How a programming language handles errors is something that defines it. Interpreted languages tend to rely on raising exceptions like [Python][Python-homepage] or [java][Java-homepage]. Others like to handle errors manually with return values or pointers like [C][C-wiki] or [Go][Go-homepage]. 

There is a third choice of error handling that is becoming more popular with modern languages like [Rust](Rust-homepage), [Haskell][Haskell-homepage], [Zig][Zig-homepage], and even [C++][C++-homepage].

The Result Pattern
-------------

The result pattern is a return value that represents either success or failure. The pattern forces you to handle the failure and not ignore it, which typically results less error-prone code. Rust has the [`std::result::Result`](https://doc.rust-lang.org/std/result/enum.Result.html) enum and C++23 has introduced the [`std::expected`](https://en.cppreference.com/w/cpp/utility/expected) class. Below is example code that checks if a value is 0, 1, or invalid if not either of the two and returns the result pattern.

Rust example:
===
{% highlight rust %}

fn check_value(value: &i64) -> Result<bool, String> {
    match value {
        0 => Ok(false),
        1 => Ok(true),
        _ => Err(format!("Value {value} is invalid!")),
    }
}

fn main() {
    let value: i64 = 3;
    let result = check_value(&value);
    match result {
        Ok(b) => println!("{value} is {b}!"),
        Err(msg) => println!("Error: {msg}"),
    };
}
{% endhighlight %}

`Error: Value 3 is invalid!`

C++ example
===

{% highlight c++ %}
#include <iostream>
#include <format>
#include <cstdint>
#include <expected>

auto check_value(const uint64_t& value) -> std::expected<bool, std::string> {
    if (value == 0) {
        return false;
    } else if (value == 1) {
        return true;
    } else {
        return std::unexpected(std::format("Value {} is invalid!", value));
    }
}

auto main(int argc, char* argv[]) -> int {
    const int64_t value = 3;
    if (const auto result = check_value(value); result.has_value()) {
        std::cout << std::format("{} is {}\n", value, result.value());
    } else {
        std::cout << std::format("Error: {}\n", result.error());
    }

    return 0;
}
{% endhighlight %}

`Error: Value 3 is invalid!`

Why It Matters
---

You maybe asking yourself, why does this matter and what problem does it solve? To answer that we can look at this simple example of opening a file with traditional C++ APIs:

{% highlight c++ %}
#include <fstream>
#include <iostream>

auto main() -> int {
    // Open the file
    std::fstream my_file("file.txt", std::ios::in);
    // Read the first line of the file
    std::string line;
    std::getline(my_file, line);
    std::cout << "first line: " << line << "\n";

    return 0;
}

{% endhighlight %}

`first line: `

This might be a simple enough program but it comes with some flaws. First we aren't checking if the file was actually opened. This is easy to miss since we are creating an object and C++ constructors can't return values. The only language feature we can use here is exception handling in the constructor which is highly frowned upon in C++. Second, `std::getline` is called on a file descriptor that isn't valid. This also fails silently.

So how do we fix this? In C++ our only option using the standard library is to make calls to make sure its open and valid. There are calls to make the standard library adapt `std::expected` but that would be very unlikely due to complexity and compatibility reasons.

Lets take a look at how a modern language like rust uses the result pattern in the standard library:

{% highlight rust %}
use std::fs::File;
use std::io::BufReader;
use std::io::prelude::*;

fn main() {
    let file = File::open("test.txt").expect("Failed to open file");
    let reader = BufReader::new(file);
    if let Some(first_line) = reader.lines().next().and_then(|line| line.ok()) {
        println!("First line: {}", first_line);
    }
}
{% endhighlight %}

{% highlight bash %}
thread 'main' panicked at main.rs:6:39:
Failed to open file: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
{% endhighlight %}


[C-wiki]: https://en.wikipedia.org/wiki/C_(programming_language)
[Go-homepage]: https://go.dev/
[Rust-homepage]: https://www.rust-lang.org/
[C++-homepage]: https://en.cppreference.com/w/
[Python-homepage]: https://python.org
[Java-homepage]: https://www.java.com/en/
[Haskell-homepage]: https://www.haskell.org/
[Zig-homepage]: https://ziglang.org/