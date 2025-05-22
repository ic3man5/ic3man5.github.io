---
layout: post
title:  "Modern Error Handling in Rust and C++23"
date:   2025-05-22 15:52:00 -0400
categories: rust c++ zig
---

Introduction
-------------

How a programming language handles errors is something that defines it. Interpreted languages tend to rely on raising exceptions like [Python][Python-homepage] or [java][Java-homepage]. Others like to handle errors manually with return values or pointers like [C][C-wiki] or [Go][Go-homepage]. 

There is a third choice of error handling that is becoming more popular with modern languages like [Rust](Rust-homepage), [Haskell][Haskell-homepage], [Zig][Zig-homepage], and even [C++][C++-homepage].

The Result Pattern
-------------

The result Pattern is a return value that represents either success or failure. The pattern forces you to handle the failure and not ignore it, which typically results less error-prone code. Rust has the [`std::result::Result`](https://doc.rust-lang.org/std/result/enum.Result.html) enum and C++23 has introduced the [`std::expected`](https://en.cppreference.com/w/cpp/utility/expected) class. 

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

C++ Example
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





[C-wiki]: https://en.wikipedia.org/wiki/C_(programming_language)
[Go-homepage]: https://go.dev/
[Rust-homepage]: https://www.rust-lang.org/
[C++-homepage]: https://en.cppreference.com/w/
[Python-homepage]: https://python.org
[Java-homepage]: https://www.java.com/en/
[Haskell-homepage]: https://www.haskell.org/
[Zig-homepage]: https://ziglang.org/