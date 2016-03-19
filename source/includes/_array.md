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

The array is allocated using the allocators passed in by `ArrayConf`.

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
CC_ERR_INVALID_CAPACITY | if the capacity in the `ArrayConf` does not meet the following condition: `exp_factor < (CC_MAX_ELEMENTS / capacity)`

<aside class="notice"> The <b>ArrayConf</b> struct is not modified by <b>array_new_conf</b>. All of its values are copied into the Array structure and no reference to it is stored in the Array structure, which means that the conf struct can be reused for creating other arrays.</aside>

## Destroying arrays

```c
Array *array;
enum cc_stat s = array_new(&array);

...


array_destroy(array);

```

Arrays can be destroyed by passing a pointer of the Array structure to `array_destroy`. The Array structure is freed using the `mem_free` function that is specified during Array creation.

### Function

`void array_destroy(Array*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | int | Pointer to an array that is to be destroyed

### Return

`void`

## Destroying arrays and elements

```c
Array *array;
enum cc_stat s = array_new(&array);

...


array_destroy_free(array);

```


Destroying an Array along with all the data it holds can be done by calling  `array_destroy_free` on a Array struct.

The Array structure and the contents are freed using the `mem_free` function that is specified during Array creation.

### Function

`void array_destroy_free(Array*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to an array that is to be destroyed

### Return

`void`

<aside class="warning">When using this function make sure that the array doesn't contain any elements allocated on the stack.</aside>


## Adding an element

```c
Array *ar;
enum cc_stat stat = array_new(&ar);

...

// Add "foo" string to the array and check the return code
if (array_add(ar, "foo") != CC_OK)
    // someting bad happened
    ...

```

Adding elements to an array can be done via `array_add`. Which appends the new element to the end of the Array, making it the element with the highest index. This function returns a status code to indicate success or failure.


### Function

`enum cc_stat array_add(Array*, void*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to an array that is to be destroyed
2 | void* | in | Pointer to an element that is being added


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfuly added to the array
CC_ERR_ALLOC | if memory allocation for the new element failed


## Adding element at specific position

```c
Array *ar;
enum cc_stat stat = array_new(&ar);

...

// Insert an element at index 0 and check for errors
if (array_add_at(ar, "foo", 0) != CC_OK)
    // something bad happened
    ...

```

Elements can be added to specific position within the array via `array_add_at`. Which adds an element to a specific index while shifting all subsequent elements by one. 

The element must be inserted at an already existing or an index that is `hightest-index + 1`. So that for example if the array's highest index is 5 you could insert at index 6 because that would append an element to the end of the array even though the index isn't occupied.


### Function

`enum cc_stat array_add_at(Array*, void*, size_t)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to an array that is to be destroyed
2 | void* | in | Pointer to an element that is being added
3 | size_t | in | Index at which the element is to be added


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfuly added to the array
CC_ERR_ALLOC | if memory allocation for the new element failed
CC_ERR_OUT_OF_RANGE | If the index is out of range


## Replacing elements at specific locations

```c
Array *ar;
enum cc_stat stat = array_new(&ar);

...

// Replace an element at index 3 with "foo" and store the old value at 'replaced'
void *replaced;
array_replace_at(&ar, "foo", 3, &replaced);

// Replace an element at index 10 but ignore the old value
array_replace_at(&ar, "bar", 10, NULL);

```

Existing elements can be replaced with `array_replace_at`.

### Function

`enum cc_stat array_replace_at(Array*, void*, size_t, void **)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to an array that is to be destroyed
2 | void* | in | Pointer to a replacement element
3 | size_t | in | Index of the replaced element
4 | void** | out | Pointer at which the replaced element is stored. Can be NULL if you wish to ignore it.


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfuly replaced
CC_ERR_OUT_OF_RANGE | If the index is out of range
