# Memory 布局



## Memory 中保留的插槽

Solidity 保留了 4个 32-byte 的插槽，具体的字节范围（包括两端边界）如下所示：

0x00-0x3f（64字节）：(scratch space) 哈希方法的暂存空间
0x40-0x5f（32字节）：当前分配的内存大小（也被称为：可用内存指针）
0x60-0x7f（32字节）：零值插槽

可以在代码语句中使用暂存空间(scratch space), 比如在(inline assembly)中。零插槽用作动态内存数组的初始值，不应写入（空闲内存指针头最初指向0x80）。

Solidity总是将新对象放在空闲内存指针上，内存永远不会被释放（这在将来可能会改变）。

[TODO 这里放一张图]

Solidity 内存数组红的元素总是会占用32字节的倍数的内存（即使是`bytes1[]`也是如此的，但是不适用于`bytes`和`string`的情况）。多维数组是指向内存数组的指针。动态数组的长度存储在数组的第一个插槽中，后面跟着数组元素。

[TODO 这里放一张图]

> **注意**
>
> Solidity 中的有些操作需要操作大于64字节的临时内存区域，因此无法放入暂存空间 (scratch space)。此时他们会被放到空闲指针指向的位置，但是由于他们的生命周期很短，实际的内存指针不会被更新。这个指针有可能会被清零（zeroed out），也可能不会，所以不可以认为空闲内存指针一定会指向零值内存区。
>
> 虽然使用 `msize`来获得一个绝对归零的内存区域似乎行得通，但是在不更新空闲内存指针的情况下非暂时地使用这样的指针可能会产生意想不到的结果，所以请不要这么做。

## 与Storage 内部存储布局的差异

如上所述，`Memory` 中的布局与 `Storage` 中的布局不同。下面是一些例子。

### 数组存储布局的差异


下面的数组在存储器中占用32字节（1个插槽），但在内存中占用128字节（4个项目，每个项目32字节）。

```solidity
uint8[4] a;
```

### Struct 存储布局的差异

下面的 `struct S` 在 `storage` 中占用96字节（32字节的3个插槽），但在 `Memory` 中占用128字节（每个32字节的4个 item）。

```
struct S {
    uint a;
    uint b;
    uint8 c;
    uint8 d;
}
```



## Reference

[1] https://blog.ethereum.org/2015/06/26/state-tree-pruning/