---
marp: true
title: Ruby Thread
description: Introduction to Ruby Threads
theme: uncover
class: 
  - lead
  - invert
paginate: true
url: https://intro-to-ruby-threads.luongvo.now.sh/
_paginate: false
---

<style scoped>
h1 {
  color: red;
}


footer {
	bottom: 100px;
	font-size: 20px;
}
</style>

# <!--fit--> Ruby Thread
Introduction to multithreaded programming with Ruby

![bg cover brightness:.35](https://cmkt-image-prd.global.ssl.fastly.net/0.1.0/ps/4655842/1820/1120/m1/fpnw/wm1/b8euaxjektcya03kdhbpmc9y37w4d3mseruranh8zg9sk3nu6tmqf3dp2vdkfmlg-.jpg?1529918525&s=399873ce60d780679b7ac4027c9995f6)

<!--
_footer: Luong Vo ![w:20 h:20 invert](https://cdn2.iconfinder.com/data/icons/social-icons-circular-black/512/github-512.png) luong-komorebi
-->

---

# Ruby ?

---

# Thread ?

---

## Thread

- == *Thread of execution*
- != *Hardware thread*
- Provides context for CPU
- Can be scheduled independently

![bg left:33%](https://static1.squarespace.com/static/55e4e07ae4b0870543a32e52/t/5a97132653450a9a97553f32/1519850302862/IMG_8740.jpg?format=1500w)

---

A thread is a basic unit of CPU utilization, which comprises a thread ID, a program counter, a register set, and a stack.

---

### Hardware vs Software thread

A "hardware thread" is a physical CPU or core.

---
<!-- _backgroundImage: "linear-gradient(to bottom, #67b8e3, #4104ef)" -->
**How many thread can a 4 core CPU run at once?**

---

4 core CPU can genuinely support 4 hardware threads at once - the CPU really is doing 4 things at the same time

---

<!-- _backgroundImage: "linear-gradient(to bottom, #67b8e3, #4104ef)" -->
**Then we are all limited to 4 threads ??**

You are probably using Macbook with a Core i5-8259U...

---

<!-- _backgroundImage: "linear-gradient(to bottom, #ff050c, #8c0408)" -->
# NO!

---

1. Intel® Hyper-Threading Technology
2. Hardware thread can run multiple software threads
3. Hardware thread - OS / Software thread - Language

---

![](https://slideplayer.com/slide/3715174/13/images/4/Hardware+vs.+Software+X+F+in+out+Software+Thread+parallelism.jpg)

---

![](https://www.tutorialspoint.com/operating_system/images/one_to_one.jpg)

--- 

#### Thread Scheduling

Imagine reading a book,
then take a break and comeback.... 
![bg right](https://i.imgur.com/Q0jb28I.png)

---

## Ruby

![bg contain opacity:.5](https://cdn1.iconfinder.com/data/icons/cash-coin-essentials-colored/48/JD-01-512.png)

---

#### Thread in Ruby

```ruby
# Initialization
thr = Thread.new { puts "Whats the big deal" }

# status
thr.status # => "sleep", "running", 'etc'

# other operations
thr.kill # should not be used -> raise exception
thr.exit
thr.stop && thr.wakeup
thr.pass
thr.join
```

---

```ruby
Thread.main == Thread.current

# Your Ruby programm is always running in a main thread.
# When the main thread exits, all other threads are 
# immediately terminated and the process exits.
```

---

#### Maximum number of threads

```ruby
10_000.times do |i|
  begin
    Thread.new { sleep }
  rescue ThreadError
    puts "Your thread limit is #{i} threads"
    Kernel.exit(true)
  end
end

# it depends, mine is: 4094
```

--- 

### Thread provide contexts

Threads share an address space: context, variable and AST (code tree).

---

#### Ruby maps its threads to OS native threads

```ruby
100.times do
  Thread.new { sleep }
end

puts Process.pid # returns 26600
sleep

# run in console: `top -l1 -pid 26600 -stats pid,th`
``` 

![](http://wiki.expertiza.ncsu.edu/images/d/d5/Xxyyzz.jpg)

---

![](https://i.imgur.com/TNaxVrm.png)

---

#### Benefits of native threads

- Run on multiple processors
- Scheduled by the OS 
- Blocking IO operations don't block other threads

---

**Concurrency ? For free ?**
![bg contain brightness:0.5](http://ceftandcompany.com/wp-content/uploads/2011/04/ceft-and-company-ny-agency-nike-womans-apparel-advertising-3.jpg)

---

![bg](rgb(255,128,0))
## Race condition

---

#### Race condition

> Occurs when two or more threads can access shared data and try to change it at the same time. Because the thread scheduling algorithm can swap between threads at any time, you don't know the order in which the threads will attempt to access the shared data.

---

##### Example: File Upload
![bg right contain](https://i.imgur.com/t9tvScz.png)

---

```ruby
files = {
  'daddario.png' => '*image data*',
  'elixir.png' => '*image data*'
}

expected_count = 2

100.times do
  uploader = FileUploader.new(files)
  uploader.upload
  actual_size = uploader.results.size
  if actual_size != expected_count
    raise("Race condition, size = #{actual_size}")
  end
end

puts 'No race condition this time'
exit(0)
```

---

> Threads share an address space: context, variable and AST (code tree).

```ruby
  def results
    # Threads share AST, so here might be a 
    # race condition (one thread creates Queue
    # then another one creates it again)
    # To fix: move assignment to #initialize
    @results ||= Queue.new
  end
```

---

![bg left:33%](https://media.macphun.com/img/uploads/customer/blog/1548348087/15483505255c49f43d0c1ab5.02000369.jpeg?q=85&w=1680)

#### Good practices

1. Avoid lazy instantiation in multi-thread environment 
2. Avoid concurrent modifications
3. Protect concurrent modifications

---

![bg](rgb(255,128,0))
## Ruby Global Interpreter Lock

---

#### Common myth

> Ruby is incapable of using threads because of GIL

![bg opacity:0.25 brightness:0.5](https://previews.123rf.com/images/andreypopov/andreypopov1601/andreypopov160100278/50245634-confident-security-guard-making-stop-gesture-in-front-of-gate.jpg)

---

![bg contain](https://raw.githubusercontent.com/ifyouseewendy/ifyouseewendy.github.io/source/image-repo/RCIT-native_threads.png)

---

![bg contain](https://raw.githubusercontent.com/ifyouseewendy/ifyouseewendy.github.io/source/image-repo/RCIT-threads_with_GIL.png)

---

#### Facts
1. MRI allows concurrency but prevents parallelism
2. Every Ruby process and process fork has its own GIL
3. MRI releases GIL when thread hits blocking IO (HTTP request, console IO, etc.). Therefore, blocking IO could run in parallel
4. Other implementations (JRuby) don't have GIL

---

#### Reasons of GIL
1. Protect Ruby internal C code from race conditions (it is not always thread safe)
2. Protect calls to C extensions API
3. Protect developers from race conditions

---

![bg](rgb(255,128,0))
## CPU Bound vs IO Bound

---

IO bound tasks make sense to use threads
![bg right:66%](https://i.imgur.com/RgQVzZt.png)

---

![](https://i.imgur.com/8tBAaJF.png)

*There always will be a sweet spot between utilization and context switching and it is important to find it*


---

Computations-rich code on MRI runs better on 1 thread while on other implementations on `N = CPU cores` thread

![bg right:65% fit](https://i.imgur.com/2waD1MD.png)

---
![](https://i.imgur.com/VFFvG6G.png)

---

![bg](rgb(255,128,0))
## Thread Safety

---

### Thread safe codes:

1. Doesn't corrupt your data
2. Leaves your data consistent
3. Leaves semantics of your program correct

---

Example: Order a product
```ruby
Order = Struct.new(:amount, :status) do
  def pending?
    status == 'pending'
  end

  def collect_payment
    puts "Collecting payment..."
    self.status = 'paid'
  end
end

order = Order.new(100.00, 'pending')
5.times.map do
  Thread.new do
    if order.pending? # check
      order.collect_payment # set
    end
  end
end.each(&:join)
```

---

<!-- _backgroundImage: "linear-gradient(to bottom, blue, magenta)" -->
# Mutex

---

### Mutex

- Mutual exclusion, guarantees that no two threads enter the critical section of code at the same time 
- Until the owning thread unlocks the mutex, no other thread can lock it
- The guarantee comes from the OS

---

### Mutex tips

- Use mutex to read value.
- Remember: mutexes prevents parallel execution of critical sections.
- Make critical sections as small as possible.
- Deadlock may occur when one thread waiting for a mutex locked by another thread waiting itself for the first one. Use `mutex#try_lock`

---

![bg fit](https://i.imgur.com/jmhjXjN.png)
![bg fit](https://i.imgur.com/1Ft9Vjr.png)

---

# That's all. Thank you

![bg blur](https://media.giphy.com/media/8RxCFgu88jUbe/giphy.gif)

---

## Not yet, actually...

---

# <!--fit--> ALWAYS START WITH THE "WHY"

---

## Why at the same time?

Thread is just a terminology. The ultimate reason for its existence is our demand.

---

## Why Concurrency?

but not Parallelism

---

## Why Threads? 

but not Processes? 

---

## Why concurrency with threads ?

NODEJS ftw!

---

# Beyond Ruby Threads

1. Popular application & ecosystem (Bundler, Puma, Phusion Passenger,...)
2. Concurrency Models (Actors, CSP, STM, Guilds,...)
3. Concurrency Patterns (ThreadPools, Reactor,...)
4. Other languages concurrency implementations (Golang, Erlang,...)
 
---

<style scoped>
h1 {
  color: yellow
}
</style>

# <!--fit--> Think BIGGER!

<!--
_footer: Code: github.com/luong-komorebi/intro-to-ruby-threads/ \nSlide: https://intro-to-ruby-threads.luongvo.now.sh/
-->

![bg cover brightness:.25](https://kidskonnect.com/wp-content/uploads/2018/08/wright-brothers-facts.jpg)
