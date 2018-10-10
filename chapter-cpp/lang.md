### `using namespace std`

C++ 的头文件从 `<iostream.h>` 变为 `<iostream>` 的时候，为了区分同名的内容，把 `<iostream>` 里的东西放在 `std` namespace 里。然后为了老版本代码好迁移，引入了 `using namespace` 这个指令。（来源：_C++ Primer Plus, 5th edition_ 第二章）
