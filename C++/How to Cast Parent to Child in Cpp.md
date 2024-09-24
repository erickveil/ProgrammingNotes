---
title: How to Cast Parent to Child in C++
layout: note
tags:
  - Cpp
---


Say I have the following classes:

```c++
class HNode {

	QString _ipAddress = "192.168.0.100";
	HNode* _parent;
	
public:
	
	QString getIP() { return _ipAddress; }
	bool isChassis() { return false; }
	HNode* getParent() { return _parent; }
	void setParent(HNode* parent) { _parent = parent; }

}
```

```c++
class HComposite: public HNode {

}
```

```c++
class HChassis: public HComposite {

public:
	bool isChassis() { return true; }

}
```

```c++
class WMeter {

	HNode *_node;
	
public:
	void setNode(HNode* node) { _node = node; }

}
```

Now, when WMeter is created, `WMeter::_node` is set something like this:

```
HNode rootNode;
HChassis chassisNode;
HNode moduleNode;
WMeter meterObj;

chassisNode.setParent(&rootNode);
moduleNode.setParent(&chassisNode);
meterObj.setNode(&moduleNode);
```

The hierarchy of parents that `meterObj` may have is uncontrollable in its depth. There may be any number of recursive calls to `getParent();` in order to find whatever `HNode` we're looking for.

But in this case, I want to make a method for `HChassis *WMeter::getChassis();`

I want to do something like this to navigate this parent hierarchy and get the `HNode` that is an `HChassis`, and return a pointer to that `HChassis`. The following code does not work:

```c++

HChassis* HNode::getChassis() {
	if (isChassis()) { return this; }
	return _parent->getChassis();
}
```

The problem here is that `return this` returns an `HNode*`, which doesn't match the return type.

How do you find and return an `HChassis*` to use in `WMeter` here?

To solve this, you can perform a dynamic cast on the `this` pointer within your `getChassis()` method to safely convert the `HNode*` to an `HChassis*`. The idea is to use `dynamic_cast` to check if the current node (`this`) is actually of type `HChassis`. If it is, you return the casted pointer. If it's not, you continue navigating up the parent hierarchy.

Here's how you can modify the `getChassis()` method:

```c++
HChassis* HNode::getChassis() {
    // Attempt to cast this HNode to an HChassis
    HChassis* chassis = dynamic_cast<HChassis*>(this);
    if (chassis) {
        // If the cast is successful, return the HChassis pointer
        return chassis;
    } else if (_parent) {
        // If not, recursively call getChassis on the parent
        return _parent->getChassis();
    }
    // If there's no parent and the cast was unsuccessful, return nullptr
    return nullptr;
}
```

1. **`dynamic_cast<HChassis*>(this)`**: This attempts to cast the current `HNode*` (`this`) to an `HChassis*`. If `this` is indeed pointing to an instance of `HChassis`, the cast will succeed and return a valid pointer to the `HChassis` object. If not, it returns `nullptr`.

2. **Recursion**: If the current node isn't an `HChassis` (i.e., `chassis` is `nullptr`), the method recursively calls `getChassis()` on its parent node (`_parent`). This process continues up the parent hierarchy.

3. **Return `nullptr`**: If no `HChassis` is found in the hierarchy, and there's no parent left to check, the method will return `nullptr`.

If any node in the hierarchy is an `HChassis`, it will be found and returned as an `HChassis*`. If no such node exists, the method will return `nullptr`, indicating that there is no `HChassis` in the hierarchy.
