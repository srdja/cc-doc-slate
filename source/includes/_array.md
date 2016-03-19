# Array

## Creating a new Array

```c
Array *ar;
enum cc_stat status = array_new(&ar);


// Check if the array was initialized correctly
if (status == CC_ERR_ALLOC)
    ...
```

New arrays can be created with `array_new`. This will create a new empty array with default parameters. `array_new` takes a pointer to a pointer to an Array structure and initializes it with the new Array and then returns a status code to indicate success or failure. 

### Function
`enum cc_stat array_new(Array**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array** | out | Pointer to an array that is being initialized

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the array was successfully initialized
CC_ERR_ALLOC | if memory allocation failed

## Array configuration
```c
ArrayConf ac;

// Initialize all fields to default values
array_conf_init(&c);


// change the default initial capacity
ac.capacity = 100;


// Now we can create a new array with an initial capacity of 100
Array *ar;
enum cc_stat s = array_new_conf(&ac, &ar);

...
```

ArrayConf struct lets you pass additional parameters when creating a new Array.

Perhaps you would like to initialize the array with a different initial capacity, or perhaps you might want to change the rate at which the array buffer grows. To do this you can pass a `ArrayConf` struct to `array_new_conf()` to get new customized Array.

To use the `ArrayConf` struct we first initialize all of its fields to default values with `array_conf_init(ArrayConf*)`, and then we overwrite individual fields that we're interested in.

If we simply passed `ArrayConf` to `array_conf_new()` after initializing it with `array_conf_init` without changing any values, it would have the same effect as calling `array_new()` without using the configuration struct.


### Struct

`ArrayConf`

field | type | description
----- | ---- | -----------
capacity | size_t | The initial capacity of the newly created array
exp_factor | float | The rate at which the internal buffer expands (capacity * exp_factor). For example if the exp_factor is set to 0.5, then the internal buffer would grow by 50% on each resize.
mem_alloc | void *(*) (size_t) | Pointer to a user specified "malloc"
mem_calloc | void*(*) (size_t, size_t) | Pointer to a user specified "calloc"
mem_free | void (*) (void*) | Pointer to a user specified "free"

<aside class="warning"> Make sure that all fields are initialized before passing it to <b>array_new_conf()</b> either by manually setting each field or by initializing the struct with <b>array_conf_init()</b>. </aside>

## Creating an array with parameters
```c
ArrayConf conf;
array_conf_init(&conf);

conf.capacity = 50;
conf.exp_factor = 0.7f;

// Create a new array with with additional parameters
Array *array;
enum cc_stat status = array_new_conf(&array, &conf);

// Handle errors
if (status == CC_ERR_ALLOC)
    ...

```

Arrays can be created with additional parameters by calling `array_new_conf` instead of `array_new`. The `array_new_conf` function takes a pointer to a pointer to an Array structure and also a pointer to an `ArrayConf` structure through which additional parameters are passed. `array_new_conf` returns a status code to indicate success or failure.

### Function

`enum cc_stat array_new_conf(const ArrayConf const*, Array**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | const ArrayConf const* | in | Pointer to a config struct
2 | Array** | out | Pointer to an array that is being initialized

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the array was successfully initialized
CC_ERR_ALLOC | if memory allocation failed

<aside class="notice"> The <b>ArrayConf</b> struct is not modified by <b>array_new_conf</b>. All of its values are copied into the Array structure and no reference to it is stored in the Array structure, which means that the conf struct can be reused for creating other arrays.</aside>
