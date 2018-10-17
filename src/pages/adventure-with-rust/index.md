---
title: Adventure with Rust
date: "2018-10-14"
---

I have found some unusual motivation to learn Rust programming language. Here's what happened...

<!-- end-excerpt -->

## Problem

I don't play video games very often, but when I do, I like using my __Razer Naga 2014__ mouse. It has 12 extra buttons on it's side, which really improve my gaming experience. The only problem I've had with this mouse is that Razer does not provide any official application for managing those buttons for Linux platform. The mouse works, but side buttons are bounded to keys 1..9, 0, -, =, and it's not possible to change that binding.

I've decided to write a program that is able to remap my Razer Naga's buttons. I had some experience writing such software in __Python__, but this time I've decided to use __Rust__ language, because:

* it is faster than Python,
* it compiles into single binary file, so distribution will be easier,
* I want to learn it.

## Rust

Rust is an object oriented programming language with support for functional programming. It is fast - [in some benchmarks even faster than C](https://benchmarksgame-team.pages.debian.net/benchmarksgame/faster/rust.html). It's compiler is sometimes annoying, but it guarantees memory and thread safety. Rust doesn't use __Garbage Collector__ to for memory management. Instead it uses kind of __Scope-based resource management__ to reclaim memory, which makes it more deterministic than in languages using Garbage Collection.

In my opinion Rust has __steep learning curve__. Even experienced developers have to fight with it's compiler from time to time. However Rust was 3 times in a row __#1__ most loved programming language in Stack overflow developer surveys.

## Solution

To solve this problem I've used [evdev-rs](https://github.com/ndesh26/evdev-rs) and [rust-uinput](https://github.com/meh/rust-uinput) libraries.

evdev-rs is a wrapper for C library [libevdev](https://www.freedesktop.org/software/libevdev/doc/latest). I've used it to read event's from Razer's side buttons.

With rust-uinput I was able to create a virtual keyboard.

Final flow looks like this:

```read events from naga buttons``` -> ```map into expected keys``` -> ```write as events to virtual keyboard```

Core part of my rust code looks like this:

```rust
pub fn map_events(naga: Naga, input_device: &mut Device) {
    loop {
        let event = naga.next_event();
        match event {
            Ok((_read_status, input_event)) => process_event(input_event, input_device),
            Err(e) => {
                println!("Err: {}", e);
                return;
            }
        }
    }
}

fn process_event(event: InputEvent, input_device: &mut Device) {
    match event.event_code {
        EV_KEY(key) => {
            let mapped_key = keymap::map_key(key).unwrap();
            match event.value {
                1 => input_device.press(&mapped_key).unwrap(),
                0 => input_device.release(&mapped_key).unwrap(),
                _ => {}
            }
        }
        EV_SYN(_) => input_device.synchronize().unwrap(),
        _ => {}
    }
}
```

It seemed to be working properly during my gaming session! :)

## Summary

I wanted to play a little bit on my linux machine and thanks to this, I've found some motivation to learn more Rust. The best way for me to learn a new programming language is to write a project in it. I'm still struggling with the rust compiler, but I've made some steps forward. The language seems to be very powerful and I would like to have it in my toolbox.

Full source code is available in my github repository [jpodeszwik/razer-naga-2014-key-remap](https://github.com/jpodeszwik/razer-naga-2014-key-remap).
