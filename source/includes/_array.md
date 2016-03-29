# Array

## array_new

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

## ArrayConf
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

## array_new_conf
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

## array_destroy

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
1 | Array* | in | Pointer to an array that is to be destroyed

### Return

`void`

## array_destroy_free

```c
Array *array;
enum cc_stat s = array_new(array);

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


## array_add

```c
Array *ar;
enum cc_stat stat = array_new(&ar);

...

// Add "foo" string to the array and check the return code
if (array_add(ar, "foo") != CC_OK)
    // something bad happened
    ...

```

Adding elements to an array can be done via `array_add` which appends the new element to the end of the Array, making it the element with the highest index. This function returns a status code to indicate success or failure.


### Function

`enum cc_stat array_add(Array*, void*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to an array to which the element is being added
2 | void* | in | Pointer to an element that is being added


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfully added to the array
CC_ERR_ALLOC | if memory allocation for the new element failed


## array_add_at

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

The element must be inserted at an already existing or an index that is `highest-index + 1`. So that for example if the array's highest index is 5 you could insert at index 6 because that would append an element to the end of the array even though the index isn't occupied.


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
CC_OK | if the element was successfully added to the array
CC_ERR_ALLOC | if memory allocation for the new element failed
CC_ERR_OUT_OF_RANGE | If the index is out of range


## array_replace_at

```c
Array *ar;
enum cc_stat stat = array_new(&ar);

...

// Replace an element at index 3 with "foo" and store the old value at 'replaced'
void *replaced;
array_replace_at(ar, "foo", 3, &replaced);

// Replace an element at index 10 but ignore the old value
array_replace_at(ar, "bar", 10, NULL);

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
CC_OK | if the element was successfully replaced
CC_ERR_OUT_OF_RANGE | If the index is out of range

## array_remove

```c
//remove an element foo and ignore the output parameter
array_remove(array, foo, NULL);

...

// remove the element bar and save the removed value to 'rm'
void *rm;
array_remove(array, bar, &rm);
```

Removing an element by reference can be done via `array_remove` which removes the element and optionally returns the removed element through an output parameter. 

### Function

`enum cc_stat array_remove(Array*, void *, void **)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | void* | in | Element that is being removed
3 | void** | out | Pointer to an output variable where the removed element should be saved. This can be set to NULL if you wish to ignore the removed element.


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfully removed
CC_ERR_VALUE_NOT_FOUND | If the element was not found


## array_remove_at

```c
//remove an element at index 3 and ignore the output parameter
array_remove_at(array, 3, NULL);

...

// remove the element at index 10 and save the removed value to 'rm'
void *rm;
array_remove_at(array, 10, &rm);
```

Removing array elements at a specific index can be done via `array_remove_at` which removes the element at the specified index and returns it through the output parameter.

### Function

`enum cc_stat array_remove(Array*, size_t, void **)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | size_t | in | Element that is being removed
3 | void** | out | Pointer to an output variable where the removed element should be saved. This can be set to NULL if you wish to ignore the removed element.


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfully removed
CC_ERR_OUT_OF_RANGE | If the index was out of range


## array_remove_last

```c
array_remove_last(array, NULL);
```

Elements can be removed from the last position, or the one with the highest index, via `array_remove_last`. This function has the same effect as calling `array_remove_at` with the highest index.

### Function

`enum cc_stat array_remove_last(Array*, void **)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | void** | out | Element that is being removed


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfully removed
CC_ERR_OUT_OF_RANGE | If the index was out of range


## array_remove_all

```c
array_remove_all(&array);
```

The array can be cleared with `array_remove_all`. This function does not shrink the underlying buffer.

### Function

`void array_remove_all(Array*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array

### Return

`void`

## array_remove_all_free

```c
array_remove_all_free(&array);
```

Removes and frees all elements from the specified array. This function does not shrink the array capacity.

### Function

`void array_remove_all_free()`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array

### Return

`void`

## array_get_at

```c
Array *ar;
enum cc_stat stat = array_new(&ar);

...

// Retrieve the element from index 5 and store it in 'e'
void *e;
if (array_get_at(ar, 5, &e) != CC_OK)
    // something bad happened
    ...
```

Returns an array element from the specified index. The specified index must be within the bounds of the array.

### Function

`enum cc_stat array_get_at(Array*, size_t, void**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | size_t | in | Index into the array
3 | void** | out | Pointer to a void* to which the element is saved


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfully retrieved
CC_ERR_OUT_OF_RANGE | If the index was out of range


## array_get_last

```c
Array *ar;
enum cc_stat stat = array_new(&ar);

...

// Retrieve the last element of the array
void *e;
if (array_get_last(ar, &e) != CC_OK)
    // something bad happened
    ...
```

Returns the last element of the array, or the element with the highest index,
if the array is not empty.

### Function

`enum cc_stat array_get_last(Array*, void**)`


param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | void** | out | Pointer to a void* to which the element is saved


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the element was successfully retrieved
CC_ERR_OUT_OF_RANGE | If the array is empty

## array_index_of

```c
size_t index = 0;
if (array_index_of(array, &foo, &index) != CC_OK)
    // something bad happened
    ...
```

Returns the index of the first occurrence of the specified array element, or `CC_ERR_OUT_OF_RANGE` if the element could not be found.

### Function

`enum cc_stat array_index_of(Array*, void*, size_t*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | void* | in | Element whose index is being looked up
3 | size_t* | out | Pointer to the index variable

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the index of the element was found
CC_ERR_OUT_OF_RANGE | if the element was not found


## array_subarray

```c
Array *array;
enum cc_stat status = array_new(&array);

if (status != CC_OK)
    ...

...

Array *sub;
status = array_subarray(array, 3, 10, &sub);

if (status != CC_OK)
    ...

```

Creates a sub-array of the specified array, ranging from `b` inclusive to `e` inclusive. The range indices must be within the bounds of the array, while the e index must be greater or equal to the `b` index, otherwise this operation will fail.


### Function

`enum cc_stat array_subarray(Array*, size_t, size_t, Array**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | size_t | in | The beginning index (inclusive) of the sub-array. Must be within the bounds of the array and must not exceed the end index.
3 | size_t | in | The end index (inclusive) of the sub-array. Must be within the bounds of the array and must be greater or equal to the beginning index.
4 | Array** | out | The pointer to where the sub array pointer is stored.

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the sub-array was successfully created
CC_ERR_INVALID_RANGE | If the specified range is invalid
CC_ERR_ALLOC | if the allocation for the new sub array failed.

<aside class="notice">The new sub-array is allocated using the original array's allocators. It also inherits the configuration of the original array.</aside>

## array_copy_shallow

```c
Array *ar;
enum cc_stat status = array_new(&ar);

...

// Make a shallow
Array *copy;
status = array_copy_shallow(ar, &copy);

```

Creates a shallow copy of the specified array. A shallow copy is a copy of the array structure, but not the elements it holds.

### Function

`enum cc_stat array_copy_shallow(Array*, Array**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | Array** | out | The pointer to where the copy array pointer is stored.

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the copy was successfully created
CC_ERR_ALLOC | if the allocation for the new copy array failed.

<aside class="notice">The new array is allocated using the original array's allocators. It also inherits the configuration of the original array.</aside>

## array_copy_deep

```c

Array *ar;
enum cc_stat status = array_new(&ar);

...

// Copy function
void *cp(void *val)
{
    Foo v = *((Foo*) val);
    Foo *new = malloc(sizeof(Foo));
    *new = v;
    return (void*) new;
}

...

// Make a shallow
Array *copy;
status = array_copy_deep(ar, cp, &copy);

```

Creates a deep copy of the specified array. A deep copy is a copy of the array structure and the data it holds.

### Function

`enum cc_stat array_copy_deep(Array*, void *(*) (void*), Array**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Pointer to the array
2 | `void *(*) (void*)` | in | Pointer to the copy function
3 | Array** | out | The pointer to where the copy array pointer is stored.


### Copy function

Takes a pointer to a value and returns a pointer to a copy of that value.

`void *cp(void*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | void* | in | Pointer to the element that is to be copied


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the copy was successfully created
CC_ERR_ALLOC | if the allocation for the new copy array failed.

<aside class="notice">The new array is allocated using the original array's allocators. It also inherits the configuration of the original array.</aside>

## array_reverse
```c
array_reverse(array);
```

Reverses the order of elements in the specified array.

### Function

`void array_reverse(Array*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Array to be reversed

### Return

`void`


## array_trim_capacity
```c
enum cc_stat status = array_trim_capacity(array);

if (status != CC_OK)
    ...
```

Trims the array's capacity to match the number of elements. The capacity can never shrink below 1.

### Function

`enum cc_stat array_trim_capacity(Array*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Array whose capacity is being trimmed

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the capacity was trimmed successfully
CC_ERR_ALLOC | if the allocation for the new capacity failed.

## array_contains
```c
size_t e = array_contains(array, &foo);
```

Returns the number of occurrences of the element within the array.

### Function

`size_t array_contains(Array *ar, void *element)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Array that is being searched


### Return

`size_t`

The number of matches.


## array_size
```c
size_t size = array_size(array);
```

Returns the size of the array. The size is the number of elements that the array holds.

### Function

`size_t array_size(Array*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Array whose size is being returned

### Return 

`size_t`

The number of elements.

## array_capacity

```c
size_t c = array_capacity(array);
```

Returns the capacity of the array. The capacity of the array is the current size of it's internal buffer.

### Function

`size_t array_capacity(Array*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Array whose capacity is being returned

### Return

`size_t`

The capacity of the array.

## array_sort

```c
// compare function
int mycmp(const void *e1, const void *e2)
{
    MyType el1 = *(*((MyType**) e1));
    MyType el2 = *(*((MyType**) e2));

    if (el1 < el2) return -1;
    if (el1 > el2) return  1;
    return 0;
}
    
...

// sort the array with the compare function
array_sort(array, mycmp);

```

Sorts the array.

### Function

`void array_sort(Array*, int (*) (const void*, const void*))`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Array that is being sorted
2 | `int (*) (const void*, const void*)` | in | Compare function. Returns < 0 if the first element goes before the second, 0 if the elements are equal and > 0 if the second elements goes before the first.

### Return

`void`

<aside class="notice"> Pointers passed to the compare function will be pointers to pointers since array elements are already pointers. This means that an extra step of dereferencing will be required before the elements can be compared</aside>

## array_map
```c

void fn(void *e)
{
    *((int*) e) += 10;
}

...

// Adds 10 to each element
array_map(array, fn);

```


Maps a function over each element of the array.

### Function

`void array_map(Array*, void (*) (void*))`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | Array* | in | Array over which the function is mapped
2 | `void (*)(void*)` | in | Function that is to be called on each element

### Return

`void`


## ArrayIter

```c
// Array iterator
ArrayIter ai;

// Initialize the iterator
array_iter_init(&ai, array);
```

Array iterator struct. Used to iterate over elements of the array in an ascending order. The iterator is initialized with `array_iter_init` which initializes the iterator with a specific array.


## array_iter_next
```c
ArrayIter ai;
array_iter_init(&ai, array);

void *next;
while (array_iter_next(&ai, &next) != CC_ITER_END) {
    ...
}

```

Advances the iterator and returns the next element in the sequence if more elements exist.

### Function

`enum cc_stat array_iter_next(ArrayIter*, void**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | ArrayIter* | in | The array iterator
2 | void** | out | The next element in the sequence

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if an element was returned
CC_ITER_END | if there are no more elements in the sequence


## array_iter_remove
```c
ArrayIter ai;
array_iter_init(&ai, array);

void *next;
while (array_iter_next(&ai, &next) != CC_ITER_END) {
    if (next == foo) {
        // safely remove an element from the array and ignore the removed value
        array_iter_remove(&ai, NULL);
    }
}
```

Removes the last returned element by `array_iter_next` without invalidating the iterator.


### Function

`enum cc_stat array_iter_remove(ArrayIter*, void**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | ArrayIter* | in | The array iterator
2 | void** | out | Pointer to where the removed element should be saved. This can be set to NULL if you wish to ignore the removed element.

### Return

code | description
---- | -----------
CC_OK | if an element was removed successfully
CC_ERR_OUT_OF_RANGE | if the element was out of range. This should never be returned if the iterator is used properly.

## array_iter_add
```c
ArrayIter ai;
array_iter_init(&ai, array);

void *next;
while (array_iter_next(&ai, &next) != CC_ITER_END) {
    if (next == foo) {
        // safely add an element
        array_iter_add(&ai, bar);
    }
}
```

Adds a new element to the array  after the last returned element by `array_iter_next`, without invalidating the iterator.

### Function

`enum cc_stat array_iter_add(ArrayIter*, void*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | ArrayIter* | in | The array iterator
2 | void* | in | The element that is being added.


### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if an element was added successfully
CC_ERR_ALLOC | if the memory allocation for the new element failed.
CC_ERR_OUT_OF_RANGE | if the element was out of range. This should never be returned if the iterator is used properly.

## array_iter_replace
```c
ArrayIter ai;
array_iter_init(&ai, array);

void *next;
while (array_iter_next(&ai, &next) != CC_ITER_END) {
    if (next == foo) {
        array_iter_replace(&ai, bar, NULL);
    }
}
```

Replaces the last returned element by `array_iter_next` with the specified element.

### Function

`enum cc_stat array_iter_replace(ArrayIter*, void*, void**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | ArrayIter* | in | The array iterator
2 | void* | in | The replacement element.
3 | void** | out | Pointer to where the replaced element should be stored. This can be set to NULL if you wish to ignore the replaced element.


### Return

code | description
---- | -----------
CC_OK | if an element was replaced successfully
CC_ERR_OUT_OF_RANGE | if the element was out of range. This should never be returned if the iterator is used properly.


## array_iter_index

```c
ArrayIter ai;
array_iter_init(&ai, array);

void *next;
while (array_iter_next(&ai, &next) != CC_ITER_END) {
    size_t i = array_iter_index(&ai);
}
```

Returns the index of the last returned element by `array_iter_next`.


### Function

`size_t array_iter_index(ArrayIter*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | ArrayIter* | in | The array iterator

### Return

`size_t`

The iterator index.
