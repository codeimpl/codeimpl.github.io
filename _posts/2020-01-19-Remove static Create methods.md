---
title: "Remove static Create methods"
date: 2020-01-19
categories: issue
---

# Title : Remove statice Create methods
 - issue number 939691

Per the blink-dev discussion (https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/iJ1bawbxbWs/vEdfT5QtBgAJ), we should remove static and trivial Create methods from the code base.

###The guideline is:

####**1. The auto-generated bindings consistently use Create().**
####**2. Don't mix Create() and a public constructor in one class.**

Newly added classes should follow the guideline.

For existing classes, rewriting per the guideline is a huge amount of work and bring few benefits. So, we scope the work to removing "redundant" Create() helpers. "redundant" means all Create() methods of the class are just wrapping MakeGarbageCollected / std::make_unique and not needed.

Note: You can use a PassKey pattern if you want to call MakeGarbageCollected() from a Create() method.

```c++
class MyClass {
 public:
  MyClass* Create(Type1 arg1, Type2 arg2, ...) {
    // This line can also be written as:
    // MyClass* instance = MakeGarbageCollected<MyClass>(util::PassKey<MyClass>(), arg1, arg2, ...);
    MyClass* instance = MakeGarbageCollected<MyClass>({}, arg1, arg2, ...);
    /* more logic here */
    return instance;
  }

  MyClass(util::PassKey<MyClass>, Type1 arg1, Type2 arg2, ...) {
  }
};
```


# Guide Line on Chromium 

## Don't mix Create() factory methods and public constructors in one class.
 하나의 클래스에서 Create() factory 함수와 public 생성자를 혼합하여 사용하지 마세요.

A class only can have either Create() factory functions or public constructors.
In case you want to call MakeGarbageCollected<> from a Create() method, a
PassKey pattern can be used. Note that the auto-generated binding classes keep
using Create() methods consistently.

하나의 클래스는 오직 Create() factory 함수와 public 생성자 중 하나만 갖을 수 있다.
만약 당신이 Create() 함수에서 MakeGarbageCollected<>를 호출 하고 싶다면,
PassKey pattern을 사용할 수 있다.자동 생성 된 바인딩 클래스는 Create () 메소드를 
일관되게 사용합니다.


**Good:**
```c++
class HistoryItem {
 public:
  HistoryItem();
  ~HistoryItem();
  ...
}

void DocumentLoader::SetHistoryItemStateForCommit() {
  history_item_ = MakeGarbageCollected<HistoryItem>();
  ...
}
```

**Good:**
```c++
class BodyStreamBuffer {
 public:
  using PassKey = util::PassKey<BodyStreamBuffer>;
  static BodyStreamBuffer* Create();

  BodyStreamBuffer(PassKey);
  ...
}

BodyStreamBuffer* BodyStreamBuffer::Create() {
  auto* buffer = MakeGarbageCollected<BodyStreamBuffer>(PassKey());
  buffer->Init();
  return buffer;
}

BodyStreamBuffer::BodyStreamBuffer(PassKey) {}
```

**Bad:**
```c++
class HistoryItem {
 public:
  // Create() and a public constructor should not be mixed.
  static HistoryItem* Create() { return MakeGarbageCollected<HistoryItem>(); }

  HistoryItem();
  ~HistoryItem();
  ...
}
```