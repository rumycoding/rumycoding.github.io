---
title: Implementing BETA Semantics in CLox
date: 2025-02-23
tags:
- crafting interpreters
- programming language
- tech learning
categories:
- programming language
---
最近看了[Crafting Interpreters](https://www.craftinginterpreters.com/) 这本书, 准备记录一下其中一个challenge的实现过程。是[challenge 3](https://www.craftinginterpreters.com/superclasses.html#challenges)，关于在clox中实现BETA。

## Overview

In BETA semantics, method calls start at the top of the class hierarchy and walks downwards. A superclass method takes precedence over a subclass method. The `inner` keyword is introduced to allow superclass methods to call the overridden method in the subclass. An example looks like this:

```java
class Doughnut {
  cook() {
    print "Fry until golden brown.";
    inner();
    print "Place in a nice box.";
  }
}

class BostonCream < Doughnut {
  cook() {
    print "Pipe full of custard and coat with chocolate.";
  }
}

BostonCream().cook();
```

## What could be challenging?

Before implementing BETA semantics, when we call a method, we just look up the method in the class's method table and that's it. However, for BETA, we need to climb up the class inheritance tree to the top to see if we could get the method in any superclass. And that would be our starting point for calling the method.

What makes the story worse is that we could not throw the `inner` method away. We need to store it somewhere in the stack, so that when the top-level method calls `inner()`, we will be able to find it.

### Where to put the inner method?

When capturing a method, we also captured `this` using `ObjBoundMethod`. For `inner`, we could also store it in `ObjBoundMethod`, so that we have a way to store all the `inner` methods (till the innermost one) in the stack.

{% mermaid graph TD %}
  A["top level method calls inner()"] --"look up inner in the stack"--> B["ObjBoundMethod A (inner's method)"] --"read A.inner, place that in the stack"--> C["ObjBoundMethod B (inner's inner)"]
{% endmermaid %}

The new `ObjBoundMethod` looks like this (the fourth line is added):

```c
typedef struct {
   Obj obj;
   Value receiver; // this is actually a ObjInstance
   Value inner; // this is actually an ObjBoundMethod / ObjNative
   ObjClosure* method; // if this is nil, calling a native method (will be explained in Implementation session)
} ObjBoundMethod;
```

If a method does not have `inner` method, we could set its `inner` to be `ObjNative nil`, which will return nil.

We need to have a place to put `inner` in the stack. It could be the same as how we put `this` in the stack. If say BostonCream inherits Doughnut, the call stack for Doughnut cook will looks like this:

{% mermaid graph LR %}
  A["(Doughnut cook call frame) this"]--> B["inner(BostonCream's cook())"] -->C["arg 0"] --> D["arg 1..."]
{% endmermaid %}

## Implementation

When calling `invokeFromClass`:

1. Get method from class's method table
2. If class has super
   - if current class has the method, build a `ObjBoundMethod` and stored it in the stack
   - if current class does not have the method, which means `inner` is nil, put a nil method in the stack
   - Note that if `inner` is not nil method, we still need to build a `ObjBoundMethod` in the stack, so that when GC is happening, `inner` is not lost (later, we need to recursively call `invokeFromClass` to reach the top-level class of the inheritance tree. It is possible that the top-level class does not implement the method we want to call. In that case, we should make sure the most-top-level class that implement the method does not get lost)
3. call `invokeFromClass` using super
   - if call is successful, just return true
   - Otherwise, reset the `inner` to the original `inner` (this is possible as we enforced 2b.i.)
4. If in step 1. we get the method from current class method table, call that method
   - otherwise, return false, indicating calling the method is not successful.

```c
// Stack: receiver, inner, args...
static bool invokeFromClass(ObjClass* klass, ObjString* name,
                            int argCount, Value* inner) {
  assert(inner != NULL);

  Value method;
  Value receiver = peek(argCount + 1);
  bool gotMethod = tableGet(&klass->methods, name, &method);

  if (klass->superclass != NULL) {
    Value originalMethod = method;
    if (gotMethod) {
      // if inner is native, we do not need to bind it
      if (!IS_NATIVE(*inner)) {
        method = OBJ_VAL(newBoundMethod(receiver, *inner, AS_CLOSURE(method)));
      }
    } else {
      if (!IS_NATIVE(*inner)) {
        // if gotMethod is false, inner is no where in the stack, it might subject to gc
        // maybe we should allow the inner to be stored in a bound method that has nativenil as the method of the bound method
        method = OBJ_VAL(newBoundMethod(receiver, *inner, NULL));
      }
      else {
        // note here copy string will not allocate new memory, as the string is already in the string table
        assert(tableGet(&vm.globals, copyString(NILNATIVENAME, strlen(NILNATIVENAME)), &method)); 
      }
    }
    vm.stackTop[-argCount - 1] = method;
    if (invokeFromClass(klass->superclass, name, argCount, &method)) {
      return true;
    }
    method = originalMethod;
    vm.stackTop[-argCount - 1] = *inner; // as we set the stack back, bound method we created might be gc-ed
  }

  // superclass is null, or the method is not found in the superclass
  if (!gotMethod) {
    return false;
  }

  // inner is already on the stack
  return call(AS_CLOSURE(method), argCount);
}
```

## Optimization

One possible optimization is to create `ObjBoundMethod` only if the method is calling `inner`. We could store whether method is calling `inner` during compile time, and store that info inside `ObjFunction`. If we find a method we want to call, but it does not use `inner()`, then we could safely throw the `inner` in the stack away, and let GC to collect it.
