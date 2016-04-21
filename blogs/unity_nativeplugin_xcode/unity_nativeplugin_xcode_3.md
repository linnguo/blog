#Unity NativePlugin实践三: 回调函数
目标: 在Native环境中(也就是C代码中)调用C#函数

C#中调用C函数, 需要在C#中声明一个签名匹配的函数, 并且使用[DllImport("dllName")]标识. 两个函数的对应是由平台来做的. 但是反过来, 在C中调用C#的函数, 平台不会自动的匹配同名同签名的函数, 方法是C#调用C的函数, 并且把C#编写的回调函数作为参数传递给C, C会把这个参数类型映射为函数指针.

`TestNativePlugin.h`

``` csharp
public class TestNativePlugin : MonoBehaviour {
	void Start () {
		CallVoidFunc(Print);
	}
	
	void Print(){
		Debug.LogError("In C# print");
	}
	
	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
	delegate void VoidFunc();
	
	[DllImport("Add")]
	static extern void CallVoidFunc(VoidFunc func);	
}	
```

`add.c`

``` c
typedef void (*VoidFunc)(void);

void CallVoidFunc(VoidFunc f)
{
    f();
}
```