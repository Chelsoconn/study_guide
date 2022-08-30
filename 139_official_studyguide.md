***BLOCKS AND PROCS***



**What are closures**

Closures are a programming concept that allows you to save a chunk of code and execute it later.  A closure binds to surrounding artifacts (variables, methods, objects) and encloses around everything so that these artifacts can be referenced when the closure is executed. Ruby implements closures through `Proc` objects (instantiated from the `Proc` class), a lambda, or a block. We can pass these chunks of code into existing methods, which can be really useful in maintaing DRY code. 

It's important to note that these bindings allow programmers to use and update local variables that are in scope when the closure is created, even if the call for execution comes from a place where these variables are not in scope. 

1) Closure code example 

   ```ruby 
   def new_array(arr)
     arr.each {|element| yield element}
   end 
   
   arr = [1,2,3,4,5]
   results = []
   
   new_array(arr) do |num|
     results << num.next #blocks have access to outer scope local variables 
   end 
   
   p results #We're able to access local variable `results` from within a method without passing it in as an argument due to the closures' binding. 
   ```

   The block here begins with the `do` keyword and ends at the `end` keyword. We are able to reference the local variable `results` at time of execution due to the closure's binding at time of implementation.  The code within the block gets executed within the `new_array` method and mutates the `results` array, which can be seen when we ouput the `results` array after execution. 

   

2) Interesting Closure example 

   ```ruby 
   def alphabet 
     counter = -1 
     letters ('a'..'z').to_a 
     #instantiate and return a new Proc object
     Proc.new do 
       counter += 1 
       letters[counter]
     end 
   end 
   #assigns return value of alphabet (a Proc) to alpha1
   alph1 = alphabet 
   p alph1
   <Proc object>
   p alph1.call #execute Proc with Proc.call 'a'
   p alph1.call #'b'
   p alph1.call #'c'
   #returns and assigns a second Proc object to alpha2
   alph2 = alphabet 
   p alph2
   <a different Proc object>
   p alpha2.call #'a'
   p alpha1.call #'d'
   ```

   Above, we define the method `alphabet` such that it returns a `Proc` object. This `Proc` forms a closure with local variables `counter` and `alphabet`. We invoke this method on line 14 and assign the returned `Proc` to the local variable `alpha1`. Now that the `Proc` is captured by a variable, we can invoke it whenever we want with the `Proc#call` method.

   When we execute the `Proc`, it has its own private copy of `counter` and `alphabet`. The copy of `counter` gets incremented, but only for this particular `Proc` object. This is evident by the fact that we progress through the letters of the alphabet with each time we invoke the `Proc`.

   We also have the capability to invoke `alphabet` again and return a new `Proc` object. When we call `alphabet` and assign the return value to `alpha2` on line 24, it represents a second instance of `Proc` that has its own separate copy of local variables `counter` and `letters`. Therefore, when we invoke `Proc#call` on it, we begin again with the letter `'a'`, because `counter` references `0`. In `alpha1`, on the other hand, we will get `'d'` by invoking the `Proc`, because in that case `counter` points to `3`.

**What is binding?**

Binding is when a "chunk of code" retains references to its' surrounding artifacts (variables, methods, objects). A closure saves a chunk of code and executes it later, but has access to the scope of the closure at time of implementation.  In other words, a binding consists of the surrounding environment or context for a closure. 

1) Binding code example 

   ```ruby 
   def age(num) #expecting an object as an argument
     num.call 
   end 
   
   age = 33 
   ans = Proc.new {p "I am #{age} years old"}
   
   age(ans) 
   ```

   The closure formed by the formation of the `Proc`  object binds to the local variables that are in scope during time of implementation. Thus, when we execute the `age`method and pass in the `Proc` object assigned to the local variable `ans`, we can access the `age` local variable which would be out of scope within the method without the closure's binding. 

**How does binding affect the scope of closures?**

Closures keep track of the artifacts that are in scope through their binding. When a closure is created, it keeps a memory to references to any artifacts in it's binding, in other words they track references to any artifacts that were in scope at the time of the closures creation.  This can include local variables, methods, objects, etc.. This can lead to what may seem like a violation of scoping rules at the time of execution.  Local variables initialized prior to the closure are in scope, while ones initialized after are not. Local variables can be reassigned after Proc formation as long as they are initialized prior. Methods defined before or after the closure's formation are in scope, as long as they are defined prior to execution. When we think of scope we’re talking about the artifacts- methods, variables, constants that are accessible in a given place in the program. When a closure is created, it keeps a memory to references to any artifacts in it's binding, in other words they track references to any artifacts that were in scope at the time of the closures creation. When it comes to local variables initialized within closures, they are scoped at the closure level and cannot be accessed from other scopes.

**How do blocks work?**

Blocks are a type of closure that begins with the `do` keyword and ends with the `end` keyword or is enclosed with ` {..}`.  The block is an argument to the method call; it gets passed into a method. The code within the block is not the method implementation. The method's implementation decides what to do with the block that is passed in. Every method in Ruby can take a block, but not every method will do something with this block. We can invoke the passed- in block argument from within the method via the `yield` keyword. It is important to remember that the number of arguments passed into a method needs to match the method definition regardless of whether we are passing in a block.  Yielding to a block allows for developers to come in after the method is implemented and add additional code and functionality.  If we to use the `yield` keyword and no block is given at time of invocation then a `LocalJumpError` will arise. A way to give added flexibility to the method is to add `yield if block_given?`. This allows the block to be executed only if the `Kernal#block_given?` conditional evaluates to true. A block parameter(block local variable) is between the pipeline operators `||` and a block local variable is a variable scoped at the block level. Ensure here that our block parameter has a unique name to prevent variable shadowing. We can pass a block argument during method implementation, and assign block parameters within the block itself. Blocks and procs have leniant arity, which means the amount of parameters and arguments don't have to match. Block arguments that are not used in the block execution are ignored, and block parameters will reference `nil` unless assigned to a block argument. Yielding to a block essentially allows the developer to inject a section of code into a method that has it's own implementation. The method's code prior to `yield` will be executed, will pass execution to the block upon the `yield` keyword, and then will return to the method to finish execution.

(Reword..)

We can see a distinction between *method implementation* and *method invocation*. Method implementation is where we define the method. This is where  execution jumps to when we call a method to begin with. Method  invocation is where we call the method. If we are working with a method  that takes a block, passing the block happens during method invocation,  not implementation. The block in question allows us to **refine the method implementation in a flexible way** without actually changing the other implementation steps of the original method we're calling. 

 1. Code example plus run-down of steps 

    ```ruby 
    # method implementation
    def say(words)
      yield if block_given?
      puts "> " + words
    end
    
    # method invocation
    say("hi there") do
      system 'clear'
    end                                                # clears screen first, then outputs "> hi there"
    ```

    

    - Execution starts at method invocation, on line 8. The `say` method is invoked with two arguments: a string and a block (the block is an implicit parameter and not part of the method definition).

    - Execution goes to line 2, where the method local variable `words` is assigned the string `"hi there"`. The block is passed in implicitly, without being assigned to a variable.

    - Execution continues into the first line of the method implementation, line 3, which immediately yields to the block.

    - The block, line 9, is now executed, which clears the screen.

    - After the block is done executing, execution continues in the method implementation on line 4. Executing line 4 results in output being displayed.

    - The method ends, which means the last expression's value is returned by this method. The last expression in the method invokes the `puts` method, so the return value for the method is `nil`.

 2. Code that shows blocks 

    ```ruby 
    # each executes a given block, but ignores its return value
    ['a', 'b', 'c'].each { |letter| puts letter.upcase }
    # outputs: A B C each on a new line
    # => ['a', 'b', 'c']
    
    # times executes a given block, but ignores its return value
    3.times { |i| i += 5 }
    # => 3
    
    # upcase can take a block, but it will not execute it
    letter = 'a'
    letter.upcase { puts letter }
    # no output, the block is not executed
    # => 'A'
    
    # map executes a given block, uses the return value for transformation
    ['a', 'b', 'c'].map { |letter| letter.upcase }
    # => ['A', 'B', 'C']
    
    # select executes a given block, uses the return value for selection
    ['a', 'B', 'c'].select { |letter| letter =~ /[a-z]/ }
    # => ['a', 'c']
    ```



**When do we use blocks?**

We use blocks for two reasons... passing implementation and sandwich code. 

1) Blocks are useful when we want to allow the user to take an action/ action on a value, but we aren't sure what that action  may be.  They allow for flexibility to define extra implementation at the time of invocation. In other words, we can defer implementation details to the time of invocation without affecting the other implementation steps in the method. 

  - code that shows this: 

    ```ruby
    def map_it_out(array)
      counter = 0 
      new_array = []
      while counter < array.size 
        new_array << yield(array[counter])
        counter += 1
      end 
      p new_array
    end 
    
    map_it_out([1,2,3,4]){|x| x+1 }
    map_it_out(['a','b','c']){|x| x.next}
    ```

    Above, we have defined a method `map_it_out`  which takes an array argument. It iterates through the elements of the  array, yielding them one at a time to the block. It then outputs the new array assigned to the `new_array` local variable, which contains the return values from yielding each element to the block.

    We first call `map_it_out` and pass it the array `[1, 2, 3, 4]` as argument. The block represents the action we want to take on each  element in the array. In this case, we define the block with block  parameter `x`, to which the current element will be assigned for each iteration of the block. The block adds 1 to each element in the array, execution jumps back to the method and the method implementation mutates the `new_array` array object in the method by adding each of these return values to the array.

    Code execution proceeds like this:

    - First we invoke `mep_it_out` and pass it the array `[1, 2, 3, 4]`.
    - Execution jumps to method implementation. `[1, 2, 3, 4]` gets assigned to the parameter `array`.
    - Local variable `counter` is initialized to `0`.
    - Local variable `new_array` is initialized to `[]`
    - We enter a `while` loop that executes until `counter < array.size` evaluates to `false`
    - The current element of the array `array[counter]` (which references `1`) is yielded to the given block.
    - Execution jumps back to method invocation. `1` is assigned to the block parameter `x`.
    - The block is executed and 1 is added to `x` . The block returns this new value.
    - Execution jumps back to method implementation. `map_it_out`uses this return value to mutate `new_array`
    - `counter` is incremented by `1`
    - The process repeats until we iterate through the entire array
    - `map_it_out` outputs the value of `new_array` and returns `nil`

2. Sandwich Code 

These are methods that need to perform before and after actions, which is referred to as "sandwich code".  There are two great examples of this: Time logging, file manipultion.  

```ruby 
def time_it
	start = Time.now
	yield
	stop = Time.now
	stop - start
end

p time_it { 45**33 }
#The method itself does not care about what the action is at all. Its purpose is to time the action, and output the time it took to execute the action. We use a block to allow any action the method caller wants to be specified at the time the method is invoked. The implementation of our method times the action, and does not change at all regardless of what the action is.
```

A built-in example of sandwich code is `File::open`. We can call `File::open` in two ways:

1. With an argument specifying the name of the file to open,  and no block. In this case, we must open the file in one step, take any  action we wish to take on the file in another step, and then explicitly  close the file once we are done with it

   ```
   file = File.open('a_file.txt', 'w+')
   # do something with the file
   file.close
   ```

2. With an argument specifying the name of the file to open,  and with a block containing the action we want to take on the file in  question. In this case, `File::open` will open the file,  perform the specified action, and close the file automatically. The  method returns the return value of the block.

   ```
   File.open('a_file.text', 'w+') do |file|
     # do something with the file
   end
   ```

The necessary cleanup of opening a file (closing it) is *automated* by the implementation of `File::open`. This means that when using `File::open` all we have to worry about as the method caller is passing in the relevant file manipulation code, and let `File::open` worry about set up and take down.

**When can you pass a block to a method? Why?**

You can pass a block to any method. The block will only be executed upon method invocation if the method implementation uses the keyword `yield` or if there is an explicit block parameter in the method implementation and the `Proc.call`method is called within the method. 



**How do we make a block argument manditory?**

A block argument is mandatory if the keyword `yield` is used without a conditional statement and the `Kernel#block_given?` method. A `LocalJumpError` will arise if no block is given.  Another way is to implement the method with an explicit block parameter and use `Proc.call` within the method. If no block is given, a NoMethodError will arise as `Proc.call` is not available to `nil`. 

**How do methods access both implicit and explicit blocks passed in?**

Implicit blocks are accessed through the `yield` keyword. Explicit blocks are accessed through the `Proc.call` method.  Explicit blocks are named objects: they are assigned to method parameters and can be managed like any other object. It can be reassigned, passed to other methods, and invoked many times. To define an explicit block you add a parameter that begins with unary `&` to the method definition.

**What is `yield` in Ruby and how does it work?**

A way that we can invoke the passed- in block argument from within the method is through the `yield` keyword. If method implentation contains a `yield`, a developer using your method can come in after this method is fully implemented and inject additional code in the middle of this method withoutt modifying the method implementation, by passing in a block of code. We can use a conditional with the method `Kernel#block_given?` to ensure a `LocalJumpError` isn't raised if no block is given at time of invocation. We can implement blocks with block parameters which can be assigned to block arguments upon invocation.

**Why is it important to know that methods and blocks can return closures?**

Methods and blocks have a return value which is the last line of code executed in the method or block body. If that is a Proc object or a lambda, or if they are returned using the `return` keyword, that closure is the return value of the block. This can be assigned to a variable and manipulated in other parts of the code. 

- If a Proc is a created and returned in a method or block, any surrounding artifacts in that method or block are part of that Procs binding, so can be accessed from the Proc. If we return several new Proc objects, each one is going to track those artifacts independently of each other - meaning if when we execute a returned Proc object, where we reassign a methods local variable from a = 0 to a = 8, the manipulation will not be seen or happen to the artifacts tracked in other Proc objects returned from the same method.

Consider the following code..

```ruby 
def sequence
  counter = 0
  Proc.new { counter += 1 }
end

s1 = sequence
p s1.call           # => 1
p s1.call           # => 2
p s1.call           # => 3
puts

s2 = sequence
p s2.call           # => 1
p s1.call           # => 4 (note: this is s1)
p s2.call           # => 2
```

Here the #sequence method returns a `Proc` that forms a closure with the local variable `counter`.  We can call the returned `Proc` repeatedly. Each time it increments its own private copy of the `counter` variable. We can create mutiple `Proc`'s from the `sequence` each with its own copy of `counter`.  

**What are the benefits of explicit blocks?**

An explicit block is a block that gets treated as a named object. It gets assigned to a method parameter so that it can be managed like any other object (it can be reassigned, passed to other methods, and invoked many times).  To define an explicit block, you add a parameter to the method definition where the name begins with an `&` character.  This essentially converts the block into a `Proc` object.

1) Code example: 

   ```ruby 
   def test(&block)
     puts "What's &block? #{block}"
   end
   
   test{'anything can go here'}
   # What's &block? #<Proc:0x007f98e32b83c8@(irb):59>
   # => nil
   ```

   The local variable `block` is a `Proc` object.  This provides extra flexibility.  Unlike with an implicit block, which we could only yield to, an explicit block provides us a variable that represents the block. We can now pass this block to another method: 

2) Code example of passing an explicit block to another method

   ```ruby 
   def test2(block)
     puts "hello"
     block.call          # calls the block that was originally passed to test()
     puts "good-bye"
   end
   
   def test(&block)
     puts "1"
     test2(block)
     puts "2"
   end
   
   test { |prefix| puts "xyz" }
   # => 1
   # => hello
   # => xyz
   # => good-bye
   # => 2
   ```

   To invoke the `Proc` from either `test` or `test2` we can call `Proc#call`.

Conversion of a block to a proc via `&method_param` is also useful if we want to pass in a block and return a `Proc` object that we can assign to a local variable and call later 

3. Code example showing conversion of block to proc and subsequent local variable assignment 

   ```ruby 
   def some_method(&block)
     block
   end 
   
   var = some_method{puts 'hi'}
   var.call
   ```

**Describe the arity differences of blocks, procs, methods, and lambdas**

The rule regarding the number of arguments that you must pass to a block, proc, or lambda is called the arity.  In Ruby, blocks and procs have leniant arity, which is why Ruby doesn't throw and error when you pass in too many or too few arguments to a block.  Methods and lambdas have strict arity, which means that you must pass the exact number of arguments that the method or lambda expects.  

1) Code example of leniant arity (procs/blocks)

   ```ruby 
   def some_method
     yield('hi')
   end 
   
   some_method{|x| puts x}
   #'hi'
   #=>nil 
   
   some_method{puts 'unused argument is ok'}
   #'unused argument is ok'
   #=>nil
   
   some_method{|x,y| puts x,y} #extra block param is ok and will return nil
   #'hi'
   #
   #=>nil
   ```

2) Code example showing strict arity (methods/lambdas)

```ruby 
def some_method(arg)
  puts 'arg'
end

some_method('This works')
#'This works'
#=>nil 

some_method
#ArgumentError
```

**When does `&` do when in a method parameter?**



1. from launch: 
   1. It is being used to describe an explicit block *parameter*
   2. It expects to take a block
   3. It will return a converted `Proc` from the block

```ruby 
def method(&var);end
```

The unary & preceding a parameter in a method definition, transforms the given block into a simple Proc object that can be referenced in the method by the local variable of the same name as the parameter, minus the ampersand. This allows us to pass around the chunk of code and even have the calling method return it, which we cannot do with blocks. The Proc can be invoked with the Proc#call method, to which any arguments can be passed in, note that the amount of arguments does not matter, just like with blocks, Procs have lenient arity. This is known as an explicit block which implies that you must pass in a block, technically you do not, but if in the method implementation we invoke the callmethod on that variable, we will get a NoMethodError , as that variable would be assigned to Nil.

1. Code example

   ```ruby 
   # proc passed as an argument to a 
   # another method
   
   def name(exclaim)
   	exclaim.call("John")
   end
   
   def greeting(&exclaim)
   	puts "Welcome!"
   	name(exclaim)
   	puts "Have a good time."
   end
   
   greeting { |title| puts "#{title}, hi!" }
   ```

**What does `&` do when in a method invocation argument?**

1. From launch: 
   - It is being used to help pass a `Proc` (or Symbol) as an argument to a method that expects a block
   - It expects to take a `Proc`
   - Will return a block that was converted from the given `Proc`

```ruby 
method(&var)
```

When the unary `&` prepends an object, that object is transformed into a block so that the method can yield to it, if the object is already a Proc that is easily transformed to a block, if not the methods own ___#to_proc method is called on it, and then that Proc is converted to a block which can be yielded to

**What is happening in the code below?**

```ruby 
arr = [1, 2, 3, 4, 5]

p arr.map(&:to_s) # specifically `&:to_s`
```

```
method(:method_name).to_proc == symbol ⇒ Method object => Proc object
```

- the `object#method` converts the given method referenced by the symbol or string to a Method object, the 

  ```
  Method#to_proc
  ```

   creates a Proc that is the body of the given method, with method arguments turning to block arguments for our Proc object

  - `method(:my_method).to_proc == proc { my_method implemenation }`

- ```
  symbol#to_proc
  ```

   calls the method name on the argument

  - `[1,2,3].map(&:to_s) == [1,2,3].map { |n| n.to_s }`

1. from launch: 

   When pairing `&` with a symbol, we are basically saying, take the method that this symbol represents and create a `Proc` that calls this method on the object that gets passed to it. Subsequently, this `Proc` can be acted on by unary `&` to convert it into a block.

   This allows us to pass symbols with unary `&` into method that expect blocks, when we are passing in a very simple block.

   ```
   proc1 = :upcase.to_proc
   # => #<Proc:0x0000562c4d59b6e8(&:upcase) (lambda)>
   
   proc2 = Proc.new { |str| str.upcase }
   # => #<Proc:0x0000562c4d56ae08 (irb):38>
   
   array = %w(a b c d e)
   
   # pass the Proc with & to a method that expects a block
   array.map(&proc1)
   # => ['A', 'B', 'C', 'D', 'E']
   
   array.map(&proc2)
   # => ['A', 'B', 'C', 'D', 'E']
   
   # we know that & will automaticall call to_proc so:
   array.map(&:upcase)
   # => ['A', 'B', 'C', 'D', 'E']
   # has the same results as the other two methods
   ```

   Note that this shortcut **only works for blocks that consist of a single method call on a single block parameter**. It does not work for methods that require an argument.

   Steps:

   1. We pass an object to `&`. If it is a `Proc`, this is converted to a block.
   2. If it is not a `Proc`, `&` will call `to_proc` on the object. If the object is not then converted to a `Proc`, an error will be raised.
   3. Now we have a `Proc` that can be converted to a block with `&`.

**How do we get the desired output without altering the method or the method invocations?**

```ruby
def call_this
  yield(2)
end

# your code here

p call_this(&to_s) # => returns 2
p call_this(&to_i) # => returns "2"
```



```ruby 
to_s = Proc.new{|x| x.to_i}
to_i = Proc.new{|x| x.to_s}
```

**How do we invoke an explicit block passed into a method using `&`? Provide example.**

We invoke an explicit block passed into a method by calling the `Proc#call` method.  The. unary `&` used in a method parameter will expect a block and will convert it to a `Proc` object.  

```ruby 
def some_method(&block)
  block.call 
end 

some_method{puts 'the block is invoked'}
```

**What concept does the following code demonstrate?**

```ruby 
def time_it
  time_before = Time.now
  yield
  time_after= Time.now
  puts "It took #{time_after - time_before} seconds."
end

time_it{5**7}
```

This is an example of sandwich code, which is one of the main reasons to use blocks. Upon method invocation, execution goes to line 1 where the block is passed implicitly, without being assigned to a variable. The current time is assigned to the local variable `time_before` within the method implementation. The method then yields to the block, which is passing implementation details to the developer.  The block is executed and execution goes back to the method implementation. The return value of the block is not used for the remainder of the method implementation. The local variable `time_after` is assigned to the current time. We then output the sentence passed to the `puts` method (interpolate the difference in local variable values) and return `nil`. This is effectively timing how long it takes to execute the block. 

**What will be outputted from the method invocation `block_method('turtle') below? Why does/doesn't it raise an error?**

```ruby 
def block_method(animal)
  yield(animal)
end

block_method('turtle') do |turtle, seal|
  puts "This is a #{turtle} and a #{seal}."
end
```

```ruby 
#"This is a turtle and a  "
#=>nil 
```

This is an example of a block's leniant arity.  When `block_method` is invoked, execution goes to `line1` and the string 'turtle' is assigned to the method parameter `animal`.  The block is passed implicitly without being assigned to a variable. Execution continues to line 2 and yields to the block.  The string assigned to the local variable `animal` is passed to `yield` as a block argument. The block is now executed with two block parameters `turtle` and `seal`.  The string 'turtle' is assigned to the block parameter `turtle` and nothing is assigned to the block parameter`seal`.  The method `puts` passes in a sentence which interpolates `turtle` and `seal`.  `turtle` wil evaluate to 'turtle' and `seal` will evalutate to 'nil'.  The output is "This is a turtle and a  " and `nil` is returned.  Blocks and procs have leniant arity which means that the amount of parameters and arguments don't need to match (no error message will be raised).  If more parameters are defined than arguments passed in, the extra parameters will reference `nil`.  If more arguments are passed in than parameters defined, the additional arguments will be ignored. 

**What will be output if we add the following code to the code above? Why?**

```ruby 
block_method('turtle') { puts "This is a #{animal}."}
```

 A `NameError` will be raised because we didn't define any block parameters so the local variable`animal` within the block is not defined. 

**What will the method call `call_me` ouput? Why?**

```ruby
def call_me(some_code)
  some_code.call
end

name = "Robert"
chunk_of_code = Proc.new {puts "hi #{name}"}
name = "Griffin"

call_me(chunk_of_code)
```

```ruby
#'hi Griffin'
```

The Proc keeps track of its surrounding context, and drags it around wherever the chunk of code is passed to.  This is called binding, or surrounding environment/context.  A closure must keep track of its binding in order to have all the information it needs to be executed later.  A closure binds to its artifacts (local variables, methods, constants, etc.), which is why we can access the local variable `name` without having to pass it into `call_me` as an argument. Local variables need to be defined before the closure is created unless explicitly passed into the closure.  Once the local variable is defined, it can be reassigned after the formation of the proc.  Bindings and closures are at the core of variable scoping rules in Ruby and why 'inner scopes can access outer scopes.'

**What happens when we change the code as such:**

```ruby 
def call_me(some_code)
  some_code.call
end

chunk_of_code = Proc.new {puts "hi #{name}"}
name = "Griffin"

call_me(chunk_of_code)
```

A `NameError` will be raised because the local variable `name` within the closure was not defined prior to the creation of the closure. 

Note that local variables must be initialized before closure creation, and methods before closure execution.

**What will the method call `call_me` output? Why?**

```ruby 
def call_me(some_code)
  some_code.call
end

name = "Robert"

def name
  "Joe"
end

chunk_of_code = Proc.new {puts "hi #{name}"}

call_me(chunk_of_code) 
```

Ruby resolves variables first and then performs method calls. So when it runs into `name` it does its standard order of operations and checks the binding for a value, and it finds one so it moves on. If it hadn't found a value it would then search the next available methods.



**Why does the following raise an error?**

```ruby 
def a_method(pro)
  pro.call
end

a = 'friend'
a_method(&a)
```

This raises a `TypeError` because on line 6 the unary `&` expects a Proc object upon method invocation. Instead a string is passed in. Also, lets say we passed in a Proc object on line 6... an `ArgumenError` will be raised because the unary `&` will convert the Proc object to a block upon method invocation, and the method definition requires an object. Remember that methods have strict arity so an  will be raised if the wrong number of arguments are passed in.  Finally lets remember that `Proc.call` is only available to Proc objects, so calling it on a string wouldn't work even outside the method (`NoMethodError`). 

**Why does the following code raise an error?**

```ruby 
def some_method(block)
  block_given?
end

bl = { puts "hi" }

p some_method(bl)

```

Here we are attempting to initialize a local variable to a block.  This will not work because a block is not an object and a `SyntaxError` will be raised. To initialize `bl` to a Proc object, we can prepend our block with `Proc.new` or simply `proc`. This will create a new Proc object and assign it to the local variable `bl`. Now when we invoke the method `some_method` on line 8 and pass in the object assigned to `bl`, we will not get an `ArgumentError`.  As you can see on line 1, `some_method` requires an argument as the method is defined with one method parameter and methods have strict arity. Upon invocation, execution begins at line 1 and we can see that the local variable `block` gets assigned to the argument `bl` which is initialized to a Proc object.  Within the method, `Kernel#block_given?` is implemented, which determines if a block was passed into the method. Because we passed in a Proc object and never converted this to a block via the `&` during method invocation and implementation, this will return `false`.  If we instead ran this..

```ruby 
def some_method(&block) #explicit block converts block to Proc to block
  block_given?
end
 
bl = Proc.new{puts 'hi'}

some_method(&bl) #& converts Proc to block 
```

Or

```ruby 
def some_method #implicit block
  block_given?
end

bl = Proc.new{puts 'hi'}

some_method(&bl) #`& converts Proc to block
```

or 

```ruby 
def some_method(block) #expecting object and accepts impicit block
  block_given?
end

bl = Proc.new{puts 'hi'} 

some_method(bl){'this is the block'} #pass in Proc object as argument and block
```

we can return `true`.

**How does `Kernel#block_given?` work?**

Make your `yield` statements more flexible by including them in a conditional that utilizes the `Kernel#block_given?` method. This will return `true` if a block is passed to the method in question and `false` if not.

**Why do we get a `Local JumpError` when executing the below code?  & How do we fix it so the output is `hi`? (2 possible ways)**

```ruby
def some(block)
  yield
end

bloc = proc { p "hi" } # do not alter

some(bloc)
```

The way to execute a Proc object is by the `Proc#call` method.  We can use the keyword `yield` to execute a block if there is one given.  We can see that the method `some` is defined to take an object, some argument that can include a Proc object. We can then see that we assign the local variable `bloc` to a proc object on line 5.  Upon method invocation on line 7, we pass in the Proc object initialized to `bloc`.  We run into an issue within the method implementation. The parameter `block` on line 1 gets assigned to the Proc object `bloc`.  We then attempt to yield to a block, but a block is not given. A `LocalJumpError` will be raised. We can fix this as follows:

```ruby 
def some(&block) #This will convert our block to a proc and back into a block
  yield
end

bloc = proc { p "hi" } # do not alter

some(&bloc) #This converts our Proc to a block
```

Here we can see we defined our method with an explicit block which will convert a Proc to a block.  If we pass in a block as an argument upon invocation, the unary `&` will first convert the block to a Proc and then back to a block.  We have to add the unary `&` to our method invocation as well to convert the Proc object to a block and to avoid an `ArgumentError`.  

Another way is..:

```ruby 
def some(block)
  block.call #This will execute the Proc object
end

bloc = proc { p "hi" } # do not alter

some(bloc)
```

This will allow us to avoid using the unary `&` and Proc to block conversions and utilize the `Proc#call` method.  We can see we passed in a Proc object to the method `some` as an argument, which is great because the method is defined with a parameter.  We then use a method that is available to the Proc object assigned to `block` and 'hi' is ouput to the screen and `nil` is returned. 

**What does the following code tell us about lambdas?**

```ruby 
bloc = lambda { p "hi" }

bloc.class # => Proc
bloc.lambda? # => true

new_lam = Lambda.new { p "hi, lambda!" } # => NameError: uninitialized constant 
```

- A special kind of `Proc` object in Ruby that exemplifies a closure.
- Does not have its own class (lambdas are instances of `Proc`)
- Can be assigned to a variable and passed around
- Initialized with `Kernel#lambda`, which is equivalent to `Proc.new`, except that the resulting `Proc` objects check the number of parameters passed when called
- Has strict arity, must be passed the correct number of expected arguments.

**What does the following code tell us about explicitly returning from proc's and lambda's?**

```ruby 
def lambda_return
  puts "Before lambda call."
  lambda {return}.call
  puts "After lambda call."
end

def proc_return
  puts "Before proc call."
  proc {return}.call
  puts "After proc call."
end

lambda_return #=> "Before lambda call."
              #=> "After lambda call."

proc_return #=> "Before proc call."
```

We can see here that returning a call to a proc will complete the execution of a method, while returning a call to a lambda will continue the method implementation.

**What will #p output below? Why is this the case and what is this code demonstrating?**

```ruby 
def retained_array
  arr = []
  Proc.new do |el|
    arr << el
    arr
  end
end

arr = retained_array
arr.call('one')
arr.call('two')
p arr.call('three')
```

Here we can see that the `retained_array` method will reurn a Proc object.  We can then assign a local variable to this Proc object as we did on line 9.  We can use the method `Proc#call`to execute this Proc, as in lines 10-12.  Line 12 will output ['one','two','three']. 

A block isn't an object, so it isn't a value that we can return. However, a `Proc` *is* an object, it's an instance of the `Proc` class. It's also a closure. That means we can actually assign closures  to variables and pass them around. It also means that we can return them from either a method or a block.

Above, we define the method `retained_array` to return a Proc object.  This proc forms a closure with local variable `arr`.  We invoke this method on line 9 and assign it to the local variable `arr`.  `arr` captures this Proc which can now be invoked through `Proc#call`. 

When this gets executed, it has its own private copy of `arr`, so as the local variable `arr` within the method gets mutated with each call to Proc, it only mutates this specific Proc's copy of `arr`.  

So let's take a closer look:

```ruby 
arr2 = retained_array
arr2.call('one')
p arr2.call('two') #=> ['one,'two]
p arr.call('four') #=> ['one', 'two', 'three',' four']
```

As we can see we have two separate instances of Proc that each have their own copy of the local variable `arr`.



-----------------------------------

***TESTING***



**What is a test suite?**

Refers to *the entire set of tests* that accompany an application or program. This can include *unit tests*, *integration tests*, *regression tests*, and a number of other things. Basically, includes **all** the tests involved with a project.

**What is a test?**

A situation or context in which  tests are run. A test ensures that a single "rule" about the interface  in question is being upheld. For example, a test can ensure you get an  error message about trying to log in with the wrong password, or be as  simple as ensuring that an instantiated object exists.

A test is made up of one or more assertions.

**What is an assertion?**

The part of a test which confirms that the results we are getting from the interface being tested match what is expected. With an assertion, we first give it the  value that we expect the code being tested to return, then we give it  the code we want to test. An assertion will compare the expected value  with the value returned by the code in a number of ways. Ostensibly,  they represent *what you are trying to verify*.

Common Assertions:

```
assert(obj, [error message])
assert_raises(ErrorType) { ... }
assert_equal(expected, actual) # value equivalence #==
assert_includes(collection, object) #include?
assert_instance_of(class, object)
assert_in_delta(3.145, Math::PI, 0.001)
assert_empty(collection) #empty?
assert_nil(obj) #nil?
assert_same(array, array.sort!) # object equivalence
assert_match(regex, obj)
```

**What are the differences of Minitest vs RSpec**

MiniTest: 

A Ruby gem that helps us with [unit testing](https://github.com/gcpinckert/rb130_139/blob/main/study_guide/testing_terms.md#unit-testing). Minitest is the default testing library that comes with Ruby. It's an  easy and simple way to begin setting up automated tests. Unlike other  testing tools like RSpec, Minitest allows us to write tests in ordinary  Ruby code, and not rely on a [DSL](https://github.com/gcpinckert/rb130_139/blob/main/study_guide/testing_terms.md#dsl).

Minitest has 2 different syntax options:

- *assert-style*: uses regular Ruby code, more intuitive for beginning Ruby developers. Writes a series of methods beginning with `test_` to test certain aspects of code, each method may contain one or more [assertions](https://github.com/gcpinckert/rb130_139/blob/main/study_guide/testing_terms.md#assertion).
- *expectation* or *spec-style*: more in line with RSpec testing style. Groups tests into `describe` blocks and writes individual tests with the `it` method. Uses expectation matchers instead of assertions.

When minitest runs tests it will give you one of 4 outputs:

- `.` = the test passed successfully
- `F` = the test failed
- `S` = the test was skipped
- `E` = the program raised and error and stopped execution

RPSEC:

RSpec bends over backwards to allow developers to write code that reads like natural English, but at the cost of simplicity. RSpec is what we call a **Domain Specific Language**; it's a DSL for writing tests. Minitest can also use a DSL, but it can also be used in a way that reads like ordinary Ruby code without a lot of magical syntax. 

**What is Domain Specific Language?(DSL)**

A higher level language built with a *general purpose language* that helps solve a problem within a specific domain. That is, it has a  specialized and specific application. Some examples of this in Ruby  include RSpec, which has a domain specific to testing, and Rails, which  has a domain specific to setting up web applications.

**What is the difference of assertion vs refutation methods?**

Refutations:

Refutations are the opposite of [assertions](https://github.com/gcpinckert/rb130_139/blob/main/study_guide/testing_terms.md#assertions). That is, instead of confirming that an arbitraty expected value is in  line with some value returned by an execution of code, they confirm that these two results are *not* in line with each other. Where as assertions *assert* (prove an expression to be true), refutations *refute*. They prove an expression to be *false*.

Take, for example, `refute_equal`. `assert_equal` asserts that the expected value and the result of the executed code are equal (using `#==`). `refute_equal` on the other hand, expects the value given and the result of executed code to be *not equal*.

Every assertion has a corresponding refutation. The  corresponding refutation takes the exact same arguments as the  assertion, except looks for a false of falsey result.

For more on assertions.. see assertions.

**How does assert_equal compare its arguments?**

`assert_equal` tests *value equality*. That is, it uses the `#==` method defined for the object in question to ascertain if two objects have equivalent values.

Because `assert_equal` uses the `#==` method to determine value equivalence, if the objects being tests are custom defined objects, we need to ensure that a custom `#==` has been implemented for the objects in their class. This is because the inherited `BasicObject#==` method checks for *object equality*, which is not necessarily the goal here.

Trying to use `assert_equal` without a custom `#==` method defined for the object in question may result in an error  message from Minitest. This message will direct you to the  implementation of `#==` to the class in question.

**What is the SEAT approach and what are its benefits?**

The **SEAT Approach** describes a common algorithm used for writing tests.

1. **S** - **set up the necessary objects(s)**

   Instead of running any repeated "set up" steps for each of our test methods, we can extract these repeated steps into a method  that gets executed before the running of each test. In [Minitest](https://github.com/gcpinckert/rb130_139/blob/main/study_guide/testing_terms.md#minitest) this is acheived by using the `setup` instance method, which will automatically run before each of the `test_...` methods. This is a good place to put something like object instantiation. By assigning objects to an *instance variable*, you ensure that they are available throughout the rest of the `test_` methods in the testing class.

2. **E** - **execute the code against the object to test**

   This is where any code that you need to execute in order  to test it gets run. For example, let us say that we want to ensure that `2 + 2` is equal to `4`. In order to test this, we will have to run the statement `2 + 2`, and then test the results against our expected value `4` in an assertion. Sometimes, code that needs to be executed is simple  enough to run within the assertion itself, such as the above example.  Other times, it may require multiple steps, and it is better to run it  outside of the assertion and save the result in a local variable for  comparison.

3. **A** - **assert the results of the execution**

   This is where our assertions come in. We either *assert* or *refute* the results of our executed code with the expected results, to  determine if the behavior of our program is in line with the particular  "rule" we have defined for this particular test. This might be to see if a calculated value is equal to an expected one, if a certain action  raises a certain error, to ensure that an object is instantiated  properly, or any number of different things.

4. **T** - **tear down and clean up lingering artifacts**

   Similar to the *set up* step, we may also need a *teardown* step. This gets run after the execution of each test in our testing class. In [Minitest](https://github.com/gcpinckert/rb130_139/blob/main/study_guide/testing_terms.md#minitest) we can define our "teardown" steps in a `teardown` method. Actions that we might complete here include cleaning up and  deleting any files, logging results, or closing database connections.

**When does setup and tear down happen when testing?**

The setup instance method gets executed before running each of the `test_...` tests.  This is a good place to put something like object instantiation. By assigning objects to an *instance variable*, you ensure that they are available throughout the rest of the `test_` methods in the testing class.

The teardown steps also gets run after the execution of each step.  

**What is code coverage?**

Code coverage tells you how much of the code that the program consists of is actually tested in the test suite. Bases it's percentage on *all* code, including both public and private interfaces.

100% code coverage is only achieved when *every single method*, including private methods, are tested in the testing suite. 100% code  coverage does not necessarily mean you have perfect code. This only  means that you have tested all the methods in the code base, not that  you've considered all the edge cases or revealed potential bugs. It's a  tool used to gauge code quality but it's not perfect.

Further, while you *can* always get up to 100% code coverage it's not always necessary. The more *fault tolerant* your code should be, the higher the necessary coverage.

The `simplecov` Ruby gem is a [code coverage](https://github.com/gcpinckert/rb130_139/blob/main/study_guide/testing_terms.md#code-coverage) testing tool. Install it with the command `gem install simplecov`. Include the following lines at the top of your test file:

```
require 'simplecov'
SimpleCov.start
```

When run, it will generate a file within a `coverage` directory that details all the metrics of your code coverage.

**What is regression testing?**

A type of bug in software  that occurs after the program in question has been working already.  Typically, it describes a bug that is introduces with some kind of  change to the code base. These changes can include adding new  features/functionality or any kind of refactoring.

There are three types of regression:

- *Local Regression*: refers to a bug that is introduced by and included in any changes or additions to code.
- *Remote Regression*: refers to a change in one part of the  code that causes a different part (such as a separate module or class)  of the code to break.
- *Unmasked Regression*: refers to a bug that is revealed by a  change to code, where the bug existed before the change but didn't  previously affect the program or functionality.

**What is unit testing?**

Automated tests that are  designed and run in order to ensure that the smallest possible "unit" of a program is functioning as intended. In OOP, this "unit" usually  consists of a singe interface, as in that of a class.

The goal of unit testing is to ensure that each isolated piece of a program is functioning correctly.By using individual **assertions**, it provides a written contract that the interface in question must  satisfy. We can build "units" based on this contract. Once all the tests pass, we consider the interface in question to be complete.

Further, we can run these tests each time a change is  made, to ensure that the unit in question continually fulfills its  obligations and no regression occurs.



--------------------------

**What are the purposes of core tools?**

Core and standard library, commands like `gem` and `ruby`, an irb, documentation tools, and some Gems including `Rake` and newer version `Bundler`

- Tools aid programmers to build, write, test, document, analyze, distribute and do many other things with software. In short, tools are just programs that programmers have developed for themselves (and thankfully shared with others) to make tasks related to software easier.
- Ruby's core tools include Gems, Ruby Version Managers, Bundler, and Rake. They help Ruby developers stay efficient when building applications. 

**What are RubyGems and why are they useful?**

- RubyGems are packages of code you can download and install to use in your Ruby programs or command line. They are managed by the `gem` command that comes standard with all modern Ruby versions. Once installed, you can use the Gem by using `require`

  and the name of the Gem at the top of the file to load the file onto the file you wish to use it in. There are many Gems that serve a variety of purposes including but not limited to writing, testing, documenting, analyzing, and distributing software. Gems, in short, are just tools programmers have developed for themselves (and thankfully shared with others) to make tasks related to software easier.

  - files required to release a Gem: `test` `lib` `filename.gemspec`

- Where do they come from?

  RubyGems are available in the RubyGems library, to download one onto your local library simply use the commands gem install followed by the name of the Gem. The gem command locates the correct Gems in the remote library, downloads, and installs them. There are additional remote libraries aside from the main that you can specify and download Gems from.

- Where they go?

  Once installed, RubyGems are automatically saved on your system where they and their commands can be appropriately accessed, in what is called your local library. Where exactly that is will depend on a few factors including but not limited to your access levels, operating system, and the versions of Ruby and RubyGems you have. 

**What are Version Managers and why are they useful?**

- Ruby Version Managers are programs that help with installing, managing, and using different versions of Ruby. As Ruby comes out with new versions, features are added, deleted and modified so a program that worked on one version might not work on another, luckily we are able to continue using older versions of Ruby no matter how many new ones come out. That can get confusing very quickly though because if each project you’re working on has a different Ruby version, as they often do since applications will standardize a version that all their devs have to use, and you have to keep lead to use of unsupported language features. To make switching between these multiple Rubies, Ruby version managers like RVM and rbenv were developed.

- RVM and rbenv do similar functions and both create libraries where your Rubies and associated files are stored and from which they are managed. Key differences include: RVM comes with more functionalities, though rbenv can supply those through plug-ins. And RVM modifies the PATH variable as well as replacing the command line executable `cd`, where rbenv just modifies the PATH variable to give access to a special set of files unique to rbenv called shims.

- ```ruby 
  $ rvm install ruby-2.5.0
  $ rvm use 2.5.0
  $ gem install bundler
  ```

**What is bundler and why is it useful?**

what? / why?

- The RubyGem Bundler is a dependency manager that comes standard with newer Ruby versions, it uses on a Gemfile to determine and what Rubygems your program needs with your specified version of Ruby. Since we can have many versions of RubyGems and many RubyGems are dependent on other RubyGems, they are very difficult to keep track of, especially when you consider having many projects going at once. Bundler takes the information you give it in the Gemfile regarding which versions of which Gems and Ruby you need and creates a Gemfile.lock which gives you program access to those Gems as well as automatically install any other required Gems. This is very useful for keeping your own local projects organized as well as for collaboration - if a Gemfile is included in a shared project, you can just run the command `bundle install` and that project will have the right configurations on your system without you having to hunt down this information and install it manually.

how?

- Bundler requires a Gemfile, written in DSL, which needs to include: the remote library from which Bundler should look for Gems, (typically the main RubyGem library at [https://rubygems.org](https://rubygems.org/) ), which version of which RubyGems, a gemspec statement, and optionally though preferrably, which version of Ruby the project should use. Then, when `bundle install` is executed, a `Gemfile.lock`is created which is where all the right versions and dependent Gems are listed. Be sure to put `require 'bundler/setup'` at the very top of any file you will need Gems for, so Ruby checks the `Gemfile.lock` to get the correct version of Gems.
- ‘bundle exec’ is used preceding shell commands, it ensures that the command given will be the one that is supplied by the version of the Gem specified in the Gemlock file. It's not always necessary but does help especially with rake.

```ruby 

ruby '2.6.3' #optional

# to find versions run `gem env` to 
# confirm where to look,then
# $ find ~/.rvm -type f -iname minitest.rb

gem 'minitest', '~> 5.15'
gem 'minitest-reporters', '~> 1.5'
gem 'stamp', '~> 0.6'
gem 'rake'

gemspec #if there is a gemspec file

#then run $ bundle install so 
#Bundler creates Gemfile.lock

# add 'require 'bundler/setup'' to top
# of main program files
# (lib/todolist_project.rb and test/todolist_project_test.rb):

# to add a new gem just write it in and run `$bundle install` again
```

**What is Rake and why is it useful?**

what?

- Simply put, Rake is a RubyGem used to automate repetitive tasks. In the various cycles of a projects development, we run into a lot of repetition when it comes to building, testing, packaging, and installing programs. Rake comes pre-installed with newer version of Ruby and to use it we simply create a `Rakefile` which is a normal Ruby file, in which we define our actions we wish to automate using the Rake commands (Rakes DSL) `desc` and `task`., which take normal Ruby code. We could manually perform all the tasks but using Rake is more efficient and also reduces possible errors.

  ```ruby 
  # require "bundler/gem_tasks"
  # the above adds several tasks
  # to rakefile (optional)
  
  desc 'Say hello'
  task :hello do
    puts "Hello there. This is the `hello` task."
  end
  
  desc 'Say goodbye'
  task :bye do
    puts 'Bye now!'
  end
  
  desc 'Do everything'
  task :default => [:hello, :bye]
  
  # run using bundle exec rake hello
  # OR rake -T for all at once
  # add to Gemfile `gem 'rake'`
  ```

  - `rake build`: Constructs a `.gem` file in the `pkg` directory. This file contains the actual Rubygem that you will distribute.
  - `rake install`: runs `rake build` then installs the program in your Ruby's Gem directory. This way, you can test the Gem without having to load information from your project directory.
  - `rake release`: Send your `.gem` file to the remote Rubygems library for the world to download.

**What is a .gemspec file?**

The Gemspec file is where information about the project(version, files, name, etc) is stored, this file must be include in a projects library for distribution.

```ruby 
#The name of this file is todolist_project.gemspec
Gem::Specification.new do |s|
  s.name        = 'todolist_project'
  s.version     = '1.0.0'
  s.summary     = 'Todo List Manager!'
  s.description = 'This is a simple todo list manager!'
  s.authors     = ['Pete Williams']
  s.email       = 'pw@example.com'
  s.homepage    = 'http://example.com/todolist_project'
  s.files       = ['lib/todolist_project.rb', 'test/todolist_project_test.rb']
end
```

**What constitutes a Ruby project?**

We'll use the term **project** pretty loosely in this lesson. A project is simply a collection of one or more files used to develop, test, build, and distribute software. The software may be an executable program, a library module, or a combination of programs and library files. The project itself includes the source code (not only Ruby source code, but any language used by the project, such as JavaScript), tests, assets, HTML files, databases, configuration files, and more. While small projects include only a dozen or less files, major projects can include hundreds, even thousands, of files.

To aid developers moving from one project to another, most Ruby-based projects follow standard patterns: certain files and directories must be present, certain types of data are stored in certain locations, and some data must be presented in well-defined formats, etc. The most common standard for Ruby projects is the Rubygems standard. We won't explore the Rubygems standard in depth, but will stick to the standard in our example project. Later, you can read the Rubygems documentation to gain more in-depth knowledge.

https://www.notion.so/139-Study-Guide-c72598ca91284f2786619d4ca6000aa5