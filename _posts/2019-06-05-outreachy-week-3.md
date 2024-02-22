---
layout: post
title:  "Let's hunt those Memory Leaks!"
date:   2019-06-05
excerpt: "I learn more when I make mistakes"
tag:
- open source
- outreachy
- technology
- gsoc
- research
- python
- jekyll
- fedora
- modularity
thumbnail: https://s3.amazonaws.com/com.twilio.prod.twilio-docs/images/UvUUs1WXEBgwWcMbhbQ_JB5tScafJWbz95oNsfYnIyQQWW.width-808.jpg
categories: open-source
---

We're into week 4 of Outreachy! And just when I thought I was *finally* getting the hang of Modularity's codebase, I ran into something I've dreaded my entire life: Segmentation faults, runtime errors, AND MEMORY LEAKS!

I do know that when we allocate memory for variables, we should free them after we're done to avoid any memory leaks. You all might know already about the **Automatic garbage collection in Java**

> Java Memory Management, with its built-in garbage collection, is one of the language's finest achievements. It allows developers to create new objects without worrying explicitly about memory allocation and deallocation, because the garbage collector automatically reclaims memory for reuse.

But this is not the case with Object C in which I was working on for a certain PR. So as a methodical approach, I initialized objects, and made sure they got dereferenced once they were out of scope. And then I ran the tests.

### **FAIL**

I ran into a segmentation error I JUST COULDN'T TRACE! I ran certain tests individually. I ran a `GDB` debugger and traced, backtraced, checked my stack, added breakpoints, and possibly tried other `GDB` options, but all in vain! After possibly being stuck at this for about half a day, I turned to my mentor(Stephan) on IRC. 

Something that I must point out before I venture into what resources Stephan pointed me towards, that ultimately helped me solve this issue.

1. Asking doubts on IRC **IS** definitely intimidating. They're usually public channels and the thought that someone might be silently judging me for asking stupid questions has crossed my mind a million times.

2. But over the years, I've become a little less ashamed of what I don't know, and ask because I'll b receiving knowledge in return.

3. But just sometimes, I ask questions in haste only to realize I knew the answer already. *hehe*

Basically kids, ask your questions after you know you've thought through possible answers yourselves. No one in an open source community will ever deny you facts and resources. But if you ask your doubts with context to what you have tried, and eventually failed, others will know you're genuinely trying to solve an issue.

Anyway, I still feel my mentor is mini-Google and so I'm in luck (Thanks a lot, sir) :D

## The victim PR

This PR made me understand the niti-gritties of memory management techniques: [Add modulemd_get_stream_names()](https://github.com/fedora-modularity/libmodulemd/pull/313)

I was supposed to write a function that took a Modulemd.Module object as an input, and gave as output an ordered list of names of all the streams present in that module.

```sh
Iterate through all the streams in module:
	Store each stream name in a set

Sort set alphabetically
Return set	
```

Streams are stored as a `GPtrArray` in a `module` object. The data structure needed for the `set()` as in the psuedocode was a `GHashtable`. This is so that this set can be fed into an ordering(sorting) function later.

So I initialised a `GHashtable` with the name `stream_names`.

```sh
GHashtable *stream_names = NULL;
```
And there were two mistakes here:

1. I created a pointer to `GHashtable`, and not the hash table itself. We can think of it as initializing an array in C, but not using `malloc()` to allocate memory for it.
2. I never freed the hash table.

### Lesson 1
1.  `(transfer full)` on a container type (GPtrArray, GArray and GHashTable) tells the binding: "You are responsible for freeing the array container and all of its members".

2. `(transfer container)` on a container type tells it: "You only need to free the container itself. The elements are handled elsewhere."

So if I initialise a `GPtrArray`, store array elements in it, and de-allocate memory later, the array lets me set a function to be called on each element when the array itself is freed. Which means I don't need to individually free the elements manually. I just call `g_ptr_array_unref()` and it will also clean up the array contents.

Hence, now my code becomes:
```sh
 g_autoptr (GHashTable) stream_names =
    g_hash_table_new _full(g_str_hash, g_str_equal, g_free, NULL);

for (guint i = 0; i < self->streams->len; i++)
    {
      g_hash_table_add (stream_names,
                        (void *)modulemd_module_stream_get_stream_name (
                          g_ptr_array_index (self->streams, i)));
    }
```

3. `g_hash_table_new_full()` allows to create and allocate memory for a hash table container. `g_str_equal` specifies that the keys of this hash table need to be unique. `g_free` de-allocates memory for the keys. Since we don't have values for keys, `NULL` is provided as no memory de-allocation is required.

4. By using `g_autoptr()`, it will cause the compiler to automatically call `g_hash_table_unref()` on it when it goes out of scope.

One problem solved!

And now we come to a second, BIGGER problem! My next failure point was that I was getting an error message for a function I wasn't even using! I couldn't even trace it back to how this function was being called. That's when Stephan decided to walk me through this hurdle together.

### Reproducing the error

Observations, assumptions, and steps along the way:

1. Error messages may be coming from another test. The logs sometimes show expected failures, such as tests to make sure we gracefully handle bad input.

2. Ran just the new test, so we don't have to debug any of the other tests.
```sh
meson test --print-errorlogs module_v2_debug --test-args "-p /modulemd/v2/module/stream_names"
```
There was no mention of the assertion failure. That's because that error comes from a different test. 

3. Next, we ran the Valgrind checks. These checks detect any kinds of memory leaks and runtime errors.
```sh
meson test --wrap="valgrind --leak-check=full" --print-errorlogs module_v2_debug --test-args "-p /modulemd/v2/module/stream_names"
```
The error logs were clearly divided into 3 distinct sections:
	- The location the error was detected.
	- The location where the memory being accessed was previously freed.
	- The location where it was originally allocated.

The output was:
```sh
free(): double free detected in tcache 2
invalid free() detected
```

This meant that there were no memory *leaks*, but there were memory *errors*. We need to have the same number of free()/unref() calls as we do allocations/ref()s. But somewhere, **an extra free() was being called for memory that was already freed!** 

Now that the exact error was detected, it was time to tackle it!


### Transcending towards Nirvana

Coming back to this piece of code:
```sh
 g_autoptr (GHashTable) stream_names =
    g_hash_table_new _full(g_str_hash, g_str_equal, g_free, NULL);
```

In the function `g_hash_table_new_full()`:

1. There were no values. Hence the `NULL`. 
2. But keys needed a way to get freed. Hence, the `g_free()` on each key.
3. But because we're using `g_autoptr()`, it will free it on its own. Hence, that's a double free.

So, to correct the error, in the function `g_hash_table_new_full()` I assigned `NULL` to both the ways of freeing the keys and values.

**However, the solution was right but my reason was WRONG.**

This approach is correct and makes sense **if the hash table owns the memory of its keys**. However, when I added the keys with `g_hash_table_add()`, I added the return value of `modulemd_module_stream_get_stream_name()` to it directly. Looking at the definition of `modulemd_module_stream_get_stream_name()`, its `transfer` value was `none`. This means that the function doesn't transfer any ownership of elements, but only their values. So the caller must not `free()` it.

There are wo ways of resolving this problem: 

1. You can `g_strdup()` the string into the GHashTable, making a copy of it that you can free. Copying the memory and then allowing the GHashTable to free it is safer if there's a chance that the memory it's pointing to could be changed before you need it for something. But it comes at the performance cost of having to do a memory allocation.

2. You can drop the free function from the hash initialization.

So the next question: **Where do we use the stream names in this function?**

We add them to the `GHashtable` and then consume those keys to create an ordered set. By looking at the implementation of `modulemd_ordered_str_keys_as_strv()`, we notice:

1. It explicitly makes a `g_strdup()` of the strings when adding them to the array. 

2. So the data coming back from `modulemd_ordered_str_keys()` is not pointing at the original GHashTable internal values, but has its own memory. 

3. This means that `modulemd_module_get_stream_names_as_strv()` can safely return it as-is, because it's not coupled to the stream_names GHashTable *or* the underlying ModuleStream object.

4. It's fresh and can be passed back as `(transfer full)` safely.

5. So, since that function is returning memory that is fully self-contained, it's safe to return it. Since nothing will change it underneath us and the API consumer can hold onto it however long they need to.

```sh
GPtrArray *
modulemd_ordered_str_keys (GHashTable *htable, GCompareFunc compare_func)
{
  ...

  while (g_hash_table_iter_next (&iter, &key, NULL))
    {
      g_ptr_array_add (keys, g_strdup ((const gchar *)key));
    }
  ...

  return keys;
}
```

Hence, since we already have a generalized layer that makes a copy and returns a fully self-contained value, we wouldn't need to make a copy in our `get_stream_names()` method. And so making the copy there would be harmless but wasteful. As a general policy, if you're ever *unsure* if the memory is going to change on you, make the copy. As a general policy, if you're ever *unsure* if the memory is going to change on you, make the copy. But in this case, we can prove that it won't, so we don't need to.

With this analysis, we reach to the conclusion that the first solution is good enough (assigning NULL for freeing values). To which Stephan replied:

> <sgallagh> Not just "good enough" but, as we have just explored, it's the best answer!


Hence, my final method looked like this! 
```sh
GStrv
modulemd_module_get_stream_names_as_strv (ModulemdModule *self)
{
  g_return_val_if_fail (MODULEMD_IS_MODULE (self), NULL);

  if (!self->streams)
    return NULL;

  g_autoptr (GHashTable) stream_names =
    g_hash_table_new (g_str_hash, g_str_equal);

  for (guint i = 0; i < self->streams->len; i++)
    {
      g_hash_table_add (stream_names,
                        (void *)modulemd_module_stream_get_stream_name (
                          g_ptr_array_index (self->streams, i)));
    }

  return modulemd_ordered_str_keys_as_strv (stream_names);
}
```


### Fin.