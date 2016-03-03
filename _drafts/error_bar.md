"
If the only possible errors are programmer errors, don't return an error code, use asserts inside the function.
An assertion that validates the inputs clearly communicates what the function expects, while too much error checking can obscure the program logic. Deciding what to do for all the various error cases can really complicate the design. Why figure out how functionX should handle a null pointer if you can instead insist that the programmer never pass one?
"

# Results and Error Propagation

Generally speaking, there are three programming styles for handling errors in C:

1. continual checks with immediate returns
2. pass checks with gotos for cleanup
3. nested ifs

# The Error Bar

