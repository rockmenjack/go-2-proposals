This proposal is a mixture of my experience of using github.com/pkg/errors and the syntax sugar ? in Rust.
Sometimes it is very helpful to attch a context(callee info/stack trace) to an error for debugging or logging purpose, while occationally just returning the error is sufficient. There are also cases where you might want to recover from some errors and return the others. 

# Proposal

Add "handle" as new keyword which accepts "func(error) error"  
Add "?" as new identifer which acts like the blank identifier

The idea can be easily demonstrated with below code snippet:

```go
func global_error_handler(e error) error {
    return errors.Wrap(e, "some context")
}

func do_something() error {
    handle func(e error) error {               //  <--------+
        return errors.Wrap(e, "some context")  //           |
    }()  // or just use global_error_handler()              |
                                               //           |
    data, err := read_from_local_cache()       //           |
    if err != nil { // recover from error                   |
        data, ? = read_from_remote()  // let the general handler do it 
    }                                
                                     
    ? = process(data)                
    
    return nil
}
```
"handle" can just be a syntax suger which adds a defer.  

The same is true for "?" where the compiler can just inserts the boilerplate code
```go
if err != nil {
    return err
}
```

This proposal address two concerns to the draft design:  
1. the weirdness of adding additional keyword before a function
2. function "loses" a return value when we use "check"(readability is compromised somehow)
