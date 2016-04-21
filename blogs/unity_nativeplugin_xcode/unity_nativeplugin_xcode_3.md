#Unity NativePlugin实践三: 回调函数
目标: 在Native环境中(也就是C代码中)调用C#函数

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