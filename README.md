# Substance Expressions

Substance Designer Plugin for creating function graphs from code. Includes simple code editor and code generating features

## Installation
Just manually add plugin path to search paths in SD Prefernces:
https://docs.substance3d.com/sddoc/plugin-search-paths-172825000.html
## The Language
The plugin uses Python AST so it's syntactically Python with the expected results. However it supports a very limited featureset of Python, basically just plain arithmetic and logical expressions and function calls.

It looks like this:
```python
# This is one line comment

x = 1.0 # assign float value 1.0 to variable [x]
y = 0.5
pos = get_float2("$pos") # get system variable $pos for pixel processor

sample = samplelum(pos + vector2(x, y), 0, 0) # sample pixel 

_OUT_ = sample
```
## Built-in Function Nodes
### Constant
![Constant](https://github.com/igor-elovikov/sd-sex/blob/master/img/constant.png)

All function graph types are supported
```python
# Boolean
a = True
b = False

# Float 
f = 0.0
f2 = float2(1.0, 2.0)
f3 = float3(1.0, 2.0, 3.0)
f4 = float4(1.0, 2.0, 3.0, 4.0)

# Integer
i = 2
i2 = int2(1, 2)
i3 = int3(1, 2, 3)
i4 = int4(1, 2, 3, 4)

# String
s = "foo"
```

For vector types all components have to be explicit. So `x = float3(0.0)` won't work, use `x = float3(0.0, 0.0, 0.0)` instead

### Vector
![Vector](https://github.com/igor-elovikov/sd-sex/blob/master/img/vector.png)

#### Constructors
Float vector constructors
```python
one = 1.0
two = 2.0
three = 3.0
four = 4.0
f2 = vector2(1.0, 2.0) # [f2] is float2(1.0, 2.0)
f3 = vector3(vector2(1.0, 2.0), 3.0) # [f3] is float3(1.0, 2.0, 3.0)
f4 = vector4(vector2(1.0, 2.0), vector2(3.0, 4.0)) # [f4] is float4(1.0, 2.0, 3.0, 4.0)
```
Built-in constructors are little bit cumbersome for float3 and float4. For those variables you can use merge_float from function.sbs (standart funtions included in SD) instead.
```python
f3 = merge_float3(one, two, three) # much better than vector3((vector2(1.0, 2.0), 3.0)
f4 = merge_float4(one, two, three, four)
```

#### Swizzling
Swizzling works differently for integer and float types. Components for float vector are `.xyzw` and for integer `.abcd`
```python
int_vector = int4(1, 2, 3, 4)
i = int_vector.ab # [i] is int2(1,2)
i = int_vector.aaa # [i] is int3(1, 1, 1)
i = int_vector.dbb # [i] is int3(3, 2, 2)

f_vector = float4(1.0, 2.0, 3.0, 4.0)
f = f_vector.zyx # [f] is float3(3.0, 2.0, 1.0)
f = f_vector.ww # [f] is float2(4.0, 4.0)

# etc
```
Currently swizzling is supported only as rvalue. So assigning to attributes aren't possible
```python
f3 = float3(1.0, 2.0, 3.0)

f3.x = 0.0 # this is not supported!

f3 = merge_float3(0.0, f3.y, f3.z) # use this instead
```

### Variables
![Variables](https://github.com/igor-elovikov/sd-sex/blob/master/img/variables.png)

This one is pretty straightforward
```python
b = get_bool("my_boolean_var")

f = get_float("my_float_var")
f2 = get_float2("my_float2_var")
f3 = get_float3("my_float3_var")
f4 = get_float4("my_float4_var")

i = get_float("my_int_var")
i2 = get_float2("my_int2_var")
i3 = get_float3("my_int3_var")
i4 = get_float4("my_int4_var")
```

### Samplers
![Samplers](https://github.com/igor-elovikov/sd-sex/blob/master/img/samplers.png)

For sampling use `samplelum(uv, input, filtering)` for grayscale or `samplecol(uv, input, filtering)` for grayscale. 


`samplelum` returns float and `samplecol` returns float4

* `uv` float2 variable for uv coordinates
* `input` integer constant for input number
* `filtering` integer constant for sample. `0` for Nearest and `1` for Bilinear

Note that for input and filtering you have to use explicit constants.
```python
input_num = 5
filter = 0

# This is not supported due to how function graph works. Input and filtering can't be mutable
s = samplecol(float2(0.5, 0.5), input_num, filter) # won't work

s = samplecol(float2(0.5, 0.5), 5, 0) # OK
```


### Cast
![Cast](https://github.com/igor-elovikov/sd-sex/blob/master/img/cast.png)

Casting for the same size vectors. SD supports casting from int to float and vice versa. 
```python
# To float casts functions: tofloat(), tofloat2(), tofloat3(), tofloat4()
# To int casts functions: toint(), toint2(), toint3(), toint4()

i = 2
f = tofloat(i) # [f] is 2.0

f3 = float3(1.0, 2.0, 3.0)
i3 = toint3(f3) # [i3] id int3(1, 2, 3)
```

### Operator
![Operator](https://github.com/igor-elovikov/sd-sex/blob/master/img/operator.png)
Like in SD operators work with the same size and the same type variables.

```python
v1 = float3(1.0, 2.0, 3.0)
v2 = float3(1.0, 2.0, 3.0)

# Add
v = v1 + v2

# Subtraction
v = v1 + v2

# Negation
v = -v1

# Division (per component)
v = v1 / v2

# Multiplication (per component)
v = v1 * v2

# Modulo (per component)
v = v1 % v2

# Scalar multiplication (only float)
# The notation is [vector] @ [scalar] with the exact order so scalar always on the right side
v = v1 @ 2.0 # [v] is float3(2.0, 4.0, 6.0)

# Dot product
v = v1 ^ v2
# or
v = dot(v1, v2)
```

Arithmetical expressions follows Python grammar rules so you're not limited to just one operator
```python
v = (v1 + v2 @ 2.0) * float3(5.0, 5.0, 5.0) - (v2 - v1) @ 5.0 
```

### Logical
![Logical](https://github.com/igor-elovikov/sd-sex/blob/master/img/logical.png)
Logical operators are similar to Python
```python
yes = True
no = False

b = yes and no # False
b = yes or no # True
b = not yes # False
```

### Comparison
![Comparison](https://github.com/igor-elovikov/sd-sex/blob/master/img/comparison.png)

Comparison operators are similar to Python
```python
one = 1.0
two = 2.0
four = 4

is_even = four % 2 == 0 # True
b = one > two # False
b = four <= 
```


### Conditional
![Control](https://github.com/igor-elovikov/sd-sex/blob/master/img/control.png)


## Importing External Functions
## Metaprogramming Features
