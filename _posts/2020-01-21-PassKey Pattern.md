---
title: "PassKey pattern"
date: 2020-01-21
categories: issue
---

# Title : What is PassKey Pattern

**PassKey Pattern**

```c++
namespace util {
  template <typename T>
  class PassKey {
   private:
    // Avoid =default to disallow creation by uniform initialization.
    PassKey() {}

    friend T;
  };
}
```
