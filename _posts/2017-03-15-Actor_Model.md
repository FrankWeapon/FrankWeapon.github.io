---
layout: post
title: Actor Model (参与者模式)思想与实例
tags: concurrency
comments: true
---
- TOC
{:toc}

# Actor Model 设计思想
Actor Model（参与者模型）简化并发而产生，它将系统中的所有组件都视为 Actor（参与者），参与者之间通过发送消息来实现交互。每个 Actor 同时只能处理一个消息，也可以发送消息给其他参与者，而大量的参与者同时存在则构成了并行化。
Actor Model 中的消息为异步非阻塞式的消息传递，也就是说消息发送之后可以进行其他操作，当接收到接收方发送的返回消息时可以进行后续操作。因为 Actor 之间只能通过消息进行交互，也就是说不存在共享内存空间，这就省去了锁的操作。
Actor 通过隔离控制和计算实体来实现封装，相比使用锁来进行同步控制而言，简化了并行的实现难度。
参与者模式在1973年由 Carl Hewitt 等人发表的论文中提出[^1]，后来这种思想被应用于面向对象的并发编程语言中，其中最有名的当属 Erlang 以及 Scala。Apache 的 Akka 是实现了 Actor Model 的 library，支持 Java 以及 Scala。
在 Hadoop MapReduce v2 之后的版本中，Actor Model 就被用于其各个松耦合组件之间的通信。每个组件都维护其自己的状态机，由事件驱动状态机的变迁，在变迁发生时执行相关逻辑操作。
在通常的思路中，我们用线程来实现并发，用锁来控制共享内存中的同步互斥问题。然而，线程本身属于相对昂贵的资源，对于程序员的水平要求相对较高。对线程的不当使用将容易导致死锁问题的产生，而且大量线程的等待也对系统资源是一种浪费。
在 Actor Model 中使用基于事件驱动的编程模型，这种异步非阻塞的变成方式天生支持并发。且不需要涉及到对线程和锁的控制，可以使程序员关注于逻辑本身，降低了实现并发的难度。
不过，使用 Actor Model 进行编程需要思路上的转变，从顺序执行的程序转为事件驱动的模型可能会在一定程度上让程序本身更难以理解。
必须提到的是，对并行的控制（也就是同步）一定存在局部串行化。Actor Model 本身是一种抽象模型，对于使用者而言可以做到不使用锁，而锁将存在于更底层的位置来保证对并发的控制。
比如 Erlang 和 akka 对上层提供的 API 并不需要锁，而其复杂的并发控制分别将锁实现在编程语言级别和框架级别。也就是说 Actor Model 和线程不是同一个抽象级别的问题，Actor Model 提供了更高的抽象级别。

# 同步问题编程实例(哲学家就餐问题)
这里对哲学家就餐问题分别使用锁的机制以及 Actor Model 进行实现，以此表现 Actor Model 的抽象能力。
这里参考文章 *Locks, Actors, And Stm In Pictures*[^2]，原文中详细介绍了Lock, Actor 再加上 Transaction 共三种方案在思想上的不同，并用卡通图进行辅助说明，在这里不做翻译只进行关键部分的转述。
Actor Model 的实现使用框架celluloid[^3]，算法采用 Ruby 实现。
## 哲学家就餐问题描述
1965年，Dijkstra提出并解决了一个他称之为哲学家就餐的同步问题。该问题可简单描述为：5个哲学家围坐在一个圆桌上，每两个哲学家之间都有一只筷子，哲学家平时进行思考，只有当他们饥饿时，才拿起筷子吃饭。只有同时获得左右两只筷子他们才可以开始吃饭，吃饭后放下筷子继续思考。
关键问题是，是否可以给出一段描述其行为的程序且保证不会发生死锁。 这里使用带有 Waiter 的算法。
## Locks
```ruby
require 'thread'

class Waiter
  def initialize
    @mutex = Mutex.new
  end

  def can_eat? philosopher
    left = philosopher.left_fork
    right = philosopher.right_fork
    @mutex.synchronize do
      if !left.used && !right.used
        left.used = true
        right.used = true
        return true
      else
        return false
      end
    end
  end

  def stop_eating philosopher
    @mutex.synchronize do
      philosopher.left_fork.used = false
      philosopher.right_fork.used = false
    end
  end
end

class Philosopher
  attr_accessor :left_fork, :right_fork

  # 创建哲学家时指定其对应左右筷子，开始为思考状态
  def initialize(name, left_fork, right_fork, waiter)
    @name = name
    @left_fork = left_fork
    @right_fork = right_fork
    @waiter = waiter
    think
  end

  # 思考一定时间后处于饥饿状态，试图进食
  def think
    puts "Philosopher #@name is thinking..."
    sleep(rand())
    puts "Philosopher #@name is hungry..."
    dine
  end

  # 等待判断是否可以进食，进食完成后继续思考
  def dine
    while !@waiter.can_eat?(self)
      think
    end
    puts "Philosopher #@name eats..."
    sleep(rand())
    puts "Philosopher #@name belches"
    @waiter.stop_eating(self)
    think
  end
end

n = 5

forks = []

class Fork
  attr_accessor :used
  def initialize
    @used = false
  end
end

(1..n).each do |i|
  forks << Fork.new
end

threads = []
waiter = Waiter.new

# 为每个哲学家创建线程，为其分配筷子（圆桌)
(1..n).each do |i|
  threads << Thread.new do
    if i < n
      left_fork = forks[i]
      right_fork = forks[i+1]
    else
      # special case for philosopher #5 because he gets forks #5 and #1
      # and the left fork is always the lower id because that's the one we try first.
      left_fork = forks[0]
      right_fork = forks[n]
    end
    Philosopher.new(i, left_fork, right_fork, waiter)
  end
end

# 迭代启动线程
threads.each {|thread| thread.join}
```
## Actors
```ruby
require 'rubygems'
require 'celluloid'

class Waiter
  include Celluloid
  # 设置标记表示筷子可用状态
  FORK_FREE = 0
  FORK_USED = 1
  attr_reader :philosophers
  attr_reader :forks
  attr_reader :eating

  def initialize(forks)
    @philosophers = []
    @eating = []
    @forks = Array.new(forks, FORK_FREE)
  end

  def welcome(philosopher)
    @philosophers << philosopher
    # 异步发送事件，使哲学家开始思考
    philosopher.async.think
  end

  def hungry(philosopher)
    pos = @philosophers.index(philosopher)

    left_pos = pos
    right_pos = (pos + 1) % @forks.size
    # 判断哲学家左右两侧筷子是否可用
    if @forks[left_pos] == FORK_FREE && @forks[right_pos] == FORK_FREE
      @forks[left_pos] = FORK_USED
      @forks[right_pos] = FORK_USED
      # 当前哲学家加入进食队列并发送事件
      @eating << philosopher
      philosopher.async.eat
    else
      # 暂时不能进食，继续思考
      philosopher.async.think
    end
  end

  # 使某哲学家进食后释放筷子资源
  def drop_forks(philosopher)
    pos = @philosophers.index(philosopher)
    left_pos = pos
    right_pos = (pos + 1) % @forks.size
    @forks[left_pos] = FORK_FREE
    @forks[right_pos] = FORK_FREE
    @eating -= [philosopher]
    philosopher.async.think
  end
end

class Philosopher
  include Celluloid
  attr_reader :name
  attr_reader :waiter

  def initialize(name, waiter)
    @name = name
    @waiter = waiter
    # 向 Waiter 注册自身
    waiter.async.welcome(Actor.current)
  end

  def think
    puts "#{name} is thinking"
    sleep(rand)
    puts "#{name} gets hungry"
    waiter.async.hungry(Actor.current)
  end

  def eat
    puts "#{name} is eating"
    sleep(rand)
    puts "#{name} burps"
    waiter.async.drop_forks(Actor.current)
  end
end

names = %w{Heraclitus Aristotle Epictetus Schopenhauer Popper}
waiter = Waiter.new(names.size)
# 实例化所有哲学家，initialize方法中发送事件, 整个程序将被驱动开始运行
philosophers = names.map {|name| Philosopher.new(name, waiter)}
# 程序运行结束，进程休眠
sleep
```

[^1]: Carl Hewitt; Peter Bishop and Richard Steiger. A Universal Modular Actor Formalism for Artificial Intelligence. IJCAI. 1973
[^2]: http://adit.io/posts/2013-05-15-Locks,-Actors,-And-STM-In-Pictures.html
[^3]: https://github.com/celluloid/celluloid
