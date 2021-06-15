# Non-const static data member must be initialized out of line

```c++
// Wrong Example
class MyInteractor
{
	static int a= 0;
}
```
报错：Non-const static data member must be initialized out of line

改正：非const类型的静态变量必须要拿到类外初始化。

```c++
// Right Example
static int a= 0;
class MyInteractor 
{


}
```

