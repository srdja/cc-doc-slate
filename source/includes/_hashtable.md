# HashTable

## hashtable_new

```c
HashTable *table;

// Create a new string key table
enum cc_stat status = hashtable_new(&table);

// Check if the HashTable was initialized correctly
if (status == CC_ERR_ALLOC)
    ...
```

New hash tables can be created with `hashtable_new`. This will create a new empty hash table with default parameters. Hash tables created this way will work with **string keys**.

### Function
`enum cc_stat hashtable_new(HashTable**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable** | out | Pointer to a HashTable that is being initialized

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the HashTable was successfully initialized
CC_ERR_ALLOC | if memory allocation failed

## HashTableConf

> Configuring a new HashTable to use pointer values as keys

```c
HashTableConf htc;

// Initialize all fields to default values
hashtable_conf_init(&htc);

// Configure the HashTable to work with pointer keys
htc.hash        = POINTER_HASH;
htc.key_length  = KEY_LENGTH_POINTER;
htc.key_compare = CMP_POINTER;

// Create a new HashTable with pointer keys
HashTable *table;
if (hashtable_new_conf(&htc, &table) != CC_OK) {
    ...
}

// use the pointer value itself as a key
hashtable_add(table, (void*) 42, "answertolifeuniverseandeverything")
```

> Configuring a new HashTable to use values at pointers as keys

```c

struct foo {
    int x;
    float y;
};

bool foo_compare(void *f1, void *f2) {...}

...

HashTableConf htc;

// Initialize all fields to default values
hashtable_conf_init(&htc);

// Configure the HashTable to work with values
htc.hash        = GENERAL_HASH;
htc.key_length  = sizeof(struct foo);
htc.key_compare = foo_compare;

// Create a new HashTable with `struct foo` keys
HashTable *table;
if (hashtable_new_conf(&htc, &table) != CC_OK) {
    ...
}

struct foo f1 = ...;
struct foo f2 = ...;

// use `struct foo` values as keys
hashtable_add(table, &f1, "somevalue");
hashtable_add(table, &f2, "someothervalue");

```

> Configuring a new HashTable to use string keys (this has the same effect as creating the table with 'hasthable_new()')

```c
HashTableConf htc;

// Initialize all fields to default values
hashtable_conf_init(&htc);

// Configure the HashTable to work with string keys
htc.hash        = STRING_HASH;
htc.key_length  = KEY_LENGTH_VARIABLE;
htc.key_compare = CMP_STRING;

// Create a new HashTable with string keys
HashTable *table;
if (hashtable_new_conf(&htc, &table) != CC_OK) {
    ...
}

hashtale_add(table, "foo", "bar");

```

HashTableConf struct lets you pass additional parameters when creating a new HashTable.

### Struct

`HashTableConf`

field | type | description
----- | ---- | -----------
load_factor | float | Load factor determines when the underlying table buffer grows. For example if the load factor is 0.5 and the internal buffer capacity is 100, the resize will be triggered once the 50Th entry is added.
initial_capacity | size_t | The initial capacity of the table (rounded to power of two)
key_length | int | The length of the key. This should be set to -1 if the keys are of variable length.
hash_seed | uint32_t | Hash function seed
hash | `size_t (*) (const void *, int, uint32_t)` | The hash function which takes an `const void*` key,`int` key length and a `uint32_t` seed and returns a hash of `size_t` size. 
key_compare | `bool (*) (void*, void*)` | key comparator function which returns true if the keys are identical
mem_alloc | void *(*) (size_t) | Pointer to a user specified "malloc"
mem_calloc | void*(*) (size_t, size_t) | Pointer to a user specified "calloc"
mem_free | void (*) (void*) | Pointer to a user specified "free"

### Definitions in hashtable.h

Hash functions:

definition | description
---------- | -----------
STRING_HASH | String hash function
GENERAL_HASH | General hash function that hashes the data at the pointer.
POINTER_HASH | Hash function which hashes the pointer itself.
KEY_LENGTH_VARIABLE | Variable key length (used with string keys)

<aside class="warning"> Make sure that all fields are initialized before passing it to <b>hashtable_new_conf()</b> either by manually setting each field or by initializing the struct with <b>hashtable_conf_init()</b>. </aside>

## hashtable_new_conf
```c
HashTableConf htc;

// Initialize all fields to default values
hashtable_conf_init(&htc);

// Configure the HashTable to work with pointer keys
htc.hash        = POINTER_HASH;
htc.key_length  = KEY_LENGTH_POINTER;
htc.key_compare = CMP_POINTER;

// Create a new HashTable with pointer keys
HashTable *table;
if (hashtable_new_conf(&htc, &table) != CC_OK) {
    ...
}
```

HashTables can be created with additional parameters by calling `hashtable_new_conf` instead of `hashtable_new`. The `hashtable_new_conf` function takes a pointer to a `HashTableConf` struct and a pointer to a pointer to a HashTable structure. `hashtable_new_conf` returns a status code to indicate success or failure.

The HashTable is allocated using the allocators passed in by `HashTableConf`. These allocators are used for all future operations on this table structure.

### Function

`enum cc_stat hashtable_new_conf(HashTableConf*, HashTable**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | const HashTableConf const* | in | Pointer to a config struct
2 | HashTable** | out | Pointer to an table that is being initialized

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the table was successfully initialized
CC_ERR_ALLOC | if memory allocation failed

## hashtable_destroy

```c
HashTable *table;
enum cc_stat s = hashtable_new(&table);

...

hashtable_destroy(table);
```

Destroys the HashTable structure.

### Function

`void hashtable_destroy(HashTable*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to a table that is to be destroyed

### Return

`void`

## hashtable_add

```c
HashTable *table;
enum cc_stat stat = hashtable_new(&table);

...

if (hashtable_add(table, "foo", "bar") != CC_OK) {
    // something bad happened
    ...
}
```

Adding new key-value mappings to a HashTable can be done via `hashtable_add`. If the key is already mapped to a value in the table, that value is replaced with the new value.

### Function

`enum cc_stat hashtable_add(HashTable*, void*, void*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to a table that is to be destroyed
2 | void* | in | Pointer to a key, or a NULL key
3 | void* | in | Pointer to a value

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the mapping was successfully added to the table
CC_ERR_ALLOC | if memory allocation for the new mapping failed

## hashtable_get

```c
HashTable *table;

...

// Retrieve the value mapped to key "foo"
void *value;
if (hashtable_get(table, "foo", &value) != CC_OK) {
    // something went wrong
    ...
}

```

Gets a value associated with the specified key. If there is not value associated with the key, `CC_ERR_KEY_NOT_FOUND` is returned and the output is not set.

### Function

`enum cc_stat hashtable_get(HashTable*, void*, void**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to the table
2 | void* | in | Table key
3 | void** | out | Pointer to a `void*` to which the value is saved

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if the value was successfully retrieved
CC_ERR_KEY_NOT_FOUND | If the key doesn't exist


## hashtable_remove

```c
HashTable *table;

...

void *removed;
enum cc_stat stat = hashtable_remove(table, "foo", &removed);

```

Removes a key-value mapping from the specified table. In case the key doesn't exist, `CC_ERR_KEY_NOT_FOUND` is returned.


### Function

`enum cc_stat hashtable_remove(HashTable*, void*, void**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to the table
2 | void* | in | Key
3 | void** | out | Pointer to an output variable where the removed value should be saved. This can be set to NULL if you wish to ignore the removed value.


## hashtable_remove_all

```c
HashTable *table;

...

// clears the table
hashtable_remove_all(table);
```

Removes all key-value mappings from the table

### Function

`void hashtable_remove_all(HashTable*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to the table

### Return

`void`


## hashtable_contains_key

```c
HashTable *table;

...

// check if the table contains the key "foo"
if (hashtable_contains_key(table, "foo")) {
    ...
}
```

Checks whether or not the table contains the specified key

### Function

`bool hashtable_contains_key(HashTable*, void*)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to the table
2 | void* | in | key that is being looked up

### Return

`bool`

value | description
---- | -----------
true | if the key exists 
false | if the key doesn't exist


## hashtable_size

```c
HashTable *table;

...

// get the number of key-value mappings in the table
size_t size = hashtable_size(table);

```

Returns the number of key-value mapping in the table.

### Function

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to the table

### Return

`size_t`


## hashtable_capacity

```c
HashTable *table;

...

size_t capacity = hashtable_capacity(table);
```

Returns the current capacity of the table. The capacity represents the size of the internal table buffer.

### Function

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTable* | in | Pointer to the table

### Return

`size_t`

## HashTableIter

```c
HashTableIter hti;

// Initialize the iterator
hashtable_iter_init(&hti);
```

HashTable iterator struct used to iterate over table entries. The iterator is initalized with `array_iter_init`.


## TableEntry

### Struct

field | type | description
----- | ---- | -----------
key | void* | Table key
value | void* | Value mapped to the key
hash | size_t | Hash of the key

## hashtable_iter_next

```c
HashTable *table;

...

HashTableIter hti;
hashtable_iter_init(&hti, table);

TableEntry *entry;
while (array_iter_next(&hti, &entry) != CC_ITER_END) {
    ...
}
```

Advances the iterator and gets the next entry if it exists.


### Function

`enum cc_enum hashtable_iter_next(HashTableIter*, TableEntry**)`

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTableIter* | in | The table iterator
2 | TableEntry** | out | Pointer to where the next entry is set

### Return

`enum cc_stat`

code | description
---- | -----------
CC_OK | if an entry was returned
CC_ITER_END | if there are no more entries


## hashtable_iter_remove

```c
HashTable *table

...

HashTableIter hti;
hashtable_iter_init(&hti, table);

TableEntry *entry;
while (hashtable_iter_next(&hti, &entry) != CC_ITER_END) {
    if (entry->key == foo) {
        // safely remove an entry from the table and ignore the removed value
        hashtable_iter_remove(&hti, NULL);
    }
}
```

Removes the last returned entry by `hashtable_iter_next` without invalidating the iterator.

### Function

param | type | in/out | description
----- | ---- | ------ | -----------
1 | HashTableIter* | in | The table iterator
2 | void** | out | Pointer to where the removed value should be saved. This can be set to NULL if you wish to ignore the removed value.


### Return

code | description
---- | -----------
CC_OK | if an value was removed successfully
CC_ERR_KEY_NOT_FOUND | if the key was not found. This should never be returned if the iterator is used properly.
