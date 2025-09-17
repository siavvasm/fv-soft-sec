# Formal Verification - Absence of Specific Vulnerability Classes - Buffer Overflow - Toy Example

## Buffer Overflow 

 Below is a code snippet showing a function written in C that contains a buffer overflow vulnerability, along with a specification written in Frama-C for checking whether the function contains a buffer overflow vulnerability. 

```[c]
/*@ 
    requires \valid_read(src + (0 .. n-1));
    requires \valid_write(dst + (0 .. n-1));
    ensures \forall integer i; 0 <= i < n ==> dst[i] == src[i];
 */
void safe_copy(char *dst, const char *src, int n) {
    for (int i = 0; i < n; i++) {
        dst[i] = src[i];
    }
}
```

The first two lines of the annotated specification define the preconditions that must  hold prior to the execution of the function (i.e., both the `src` and the `dst` buffers must have at least `n` elements). 


> **Detailed Description of the annotations:** 
> 
> `\valid_read(src + (0 .. n-1));` -> "All memory locations from `src[0]` to `src[n-1]` must be readable."
> 
> `\valid(dst + (0 .. n-1));` -> "All memory locations from `dst[0]` to `dst[n-1]` must be writable."
> 
> `\forall integer i; 0 <= i < n ==> dst[i] == src[i];` -> "After execution, the `i`th element in `dst` must be equal to `src[i]` for every index `i` from 0 to n-1."

As can be seen, these "commands" are special annotations that are provided by the selected formal verification platform (here Frama-C). Different FV platforms, provide different annotations, but the main logic remains the same. 

These annotations, which as we have already said are used to formally describe the behavior of the program, its preconditions, and its postconditions (i.e., properties that need to be verified), are then turned into mathematical logic formulas, in order to be used by the prover to prove or disprove the described condition. More specifically, they are turned into a **Verification Condition (VC)** that needs to be proven by a **theorem prover.** Usually, these VCs are verified by **Automatic Theorem Provers (ATPs)**, due to their simplicity.  

## No Buffer Overflow

Now let's add an instruction in the previous code snippet, in order to remove the potential buffer overflow vulnerability. More specifically, we have included in the code of the function a potential "quick fix" that may be added by a developer in order to prevent buffer overflows from happening. The updated code snippet is provided below: 

```
void safe_copy(char *dst, size_t dst_len,
               const char *src, size_t src_len)
{
    size_t limit = (dst_len < src_len) ? dst_len : src_len;

    for (size_t i = 0; i < limit; ++i) {
        dst[i] = src[i];
    }
}
```

The main difference from the previous code snippet is the addition of the following instruction: 

```
size_t limit = (dst_len < src_len) ? dst_len : src_len;
```

Instead of the `n` variable that was provided as input to the function, we use the variable `limit` as the upper bound of the `for` loop. The value of the variable `limit` is determined based on the sizes of the two buffers (which are provided as input to the function). Particularly, the value of the variable `limit` is set to the smallest size of the two buffers. As a result. in the subsequent for loop, we will **never** cause an **out-of-bounds read** or an **out-of-bounds write**, as the value of the index `i` will never exceed the upper bounds of the two buffers. 

Hence, this instruction has essentially eliminated the possibility of a buffer overflow vulnerability exploitation. 

However, let's see how we could add a formal specification in the above snippet. The specification is show in the snippet below: 

```
/*@ 
  requires \valid(dst + (0 .. dst_len - 1));
  requires \valid_read(src + (0 .. src_len - 1));

  assigns dst[0 .. (dst_len < src_len ? dst_len - 1 : src_len - 1)];

  ensures \forall integer i;
    0 <= i < (dst_len < src_len ? dst_len : src_len) ==> 
      dst[i] == src[i];
*/
void safe_copy(char *dst, size_t dst_len,
               const char *src, size_t src_len)
{
    size_t limit = (dst_len < src_len) ? dst_len : src_len;

    for (size_t i = 0; i < limit; ++i) {
        dst[i] = src[i];
    }
}
```

So, as you can see, in the specification of the function, we have added an additional line, which describes in a formal way the assignment of the value of the variable named `limit`. 

## Question to be answered

> Is it possible to create a more generic specification that will cover the cases of buffer overflow vulnerabilities, without the need of defining a different specification for each code snippet that may be prone to this type of vulnerabilities?
