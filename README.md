# nlua
A project to conjoin lua with C#.

Building
---------

	msbuild NLua.sln

Examples
---------

You can use/instantiate any .NET class without any previous registration or annotation. 
```csharp
	public class SomeClass
	{
		public string MyProperty {get; private set;}
		
		public SomeClass (string param1 = "defaulValue")
		{
			MyProperty = param1;
		}
		
		public int Func1 ()
		{
			return 32;
		}
		
		public string AnotherFunc (int val1, string val2)
		{
			return "Some String";
		}
		
		public static string StaticMethod (int param)
		{
			return "Return of Static Method";
		}
        }

```

* Using UTF-8 Encoding:

NLua runs on top of [aeonlua](https://github.com/cartheur/aeonlua) binding, it encodes the string using the ASCII encoding by default.
If you want to use UTF-8 encoding, just set the `Lua.State.Encoding` property to `Encoding.UTF8`:

```csharp

using (Lua lua = new Lua())
{
	lua.State.Encoding = Encoding.UTF8;
	lua.DoString("res = 'Файл'");
	string res = (string)lua["res"];

	Assert.AreEqual("Файл", res);
}

```

Creating Lua state:

```csharp
	using NLua;
	
	Lua state = new Lua ()

```

Evaluating simple expressions:
```csharp
	var res = state.DoString ("return 10 + 3*(5 + 2)")[0] as double;
	// Lua can return multiple values, for this reason DoString return a array of objects
```

Passing raw values to the state:

```csharp
	double val = 12.0;
	state ["x"] = val; // Create a global value 'x' 
	var res = (double)state.DoString ("return 10 + x*(5 + 2)")[0];
```


Retrieving global values:

```csharp
	state.DoString ("y = 10 + x*(5 + 2)");
	double y = (double) state ["y"]; // Retrieve the value of y
```

Retrieving Lua functions:

```csharp
	state.DoString (@"
	function ScriptFunc (val1, val2)
		if val1 > val2 then
			return val1 + 1
		else
			return val2 - 1
		end
	end
	");
	var scriptFunc = state ["ScriptFunc"] as LuaFunction;
	var res = (int)scriptFunc.Call (3, 5).First ();
	// LuaFunction.Call will also return a array of objects, since a Lua function
	// can return multiple values
```

##Using the .NET objects.##

Passing .NET objects to the state:

```csharp
	SomeClass obj = new SomeClass ("Param");
	state ["obj"] = obj; // Create a global value 'obj' of .NET type SomeClass 
	// This could be any .NET object, from BCL or from your assemblies
```

Using .NET assemblies inside Lua:

To access any .NET assembly to create objects, events etc inside Lua you need to ask NLua to use CLR as a Lua package.
To do this just use the method `LoadCLRPackage` and use the `import` function inside your Lua script to load the Assembly.

```csharp
	state.LoadCLRPackage ();
	state.DoString (@" import ('MyAssembly', 'MyNamespace') 
			   import ('System.Web') ");
	// import will load any .NET assembly and they will be available inside the Lua context.
```

Creating .NET objects:
To create object you only need to use the class name with the `()`.

```csharp
state.DoString (@"
	 obj2 = SomeClass() -- you can suppress default values.
	 client = WebClient()
	");
```

Calling instance methods:
To call instance methods you need to use the `:` notation, you can call methods from objects passed to Lua or to objects created inside the Lua context.

```csharp
	state.DoString (@"
	local res1 = obj:Func1()
	local res2 = obj2:AnotherFunc (10, 'hello')
	local res3 = client:DownloadString('http://nlua.org')
	");
```

Calling static methods:
You can call static methods using only the class name and the `.` notation from Lua.

```csharp
	state.DoString (@"
	local res4 = SomeClass.StaticMethod(4)
	");
```

Calling properties:
You can get (or set) any property using  `.` notation from Lua.

```csharp
	state.DoString (@"
	local res5 = obj.MyProperty
	");
```

All methods, events or property need to be public available, NLua will fail to call non-public members.

##Sandboxing##

There is many ways to sandbox scripts inside your application. I strongly recommend you to use plain Lua to do your sandbox.
You can re-write the `import` function before load the user script and if the user try to import a .NET assembly nothing will happen.

```csharp
	state.DoString (@"
		import = function () end
	");
```

Getting started with NLua:
-------------------------

* Look at src/TestNLua/TestLua to see an example of usage from C# 
(optionally you can run this from inside the NLua solution using the debugger).  
Also provides a good example of how to override .NET methods of Lua and usage of NLua
from within your .NET application.

* Look at samples/testluaform.lua to see examples of how to use 
.NET inside Lua

* Enjoy the seamless beauty of these two languages conjoined.
