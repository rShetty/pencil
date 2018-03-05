## Programming Concepts

----------------------------------------------------------------------------------------------------
#### Law of Demeter
1. Law of Loose Coupling.

----------------------------------------------------------------------------------------------------
#### HaProxy : High Availabilty proxy 
1. Is a HTTP/TCP load balancer used to improve the performance of the websites by spreading requests across multiple servers.

----------------------------------------------------------------------------------------------------
#### Varnish Server: 
1. HTTP accelerator.
2. Varnish supports load balancing using both a round robin.

----------------------------------------------------------------------------------------------------
#### Singleton Methods: 
1. Methods belonging to a single instance of an object.
2. Methods defined on the object itself rather than the class.

```ruby
a = Student.new
```

##### Method on Object "a"
```ruby
def a.score
  25
end
```

----------------------------------------------------------------------------------------------------
#### SOLID - OO Principles
1. Single Responsibility Principle
2. Open/Closed Principle
3. Liskov Substitution Principle
4. Interface Segregation Principle
5. Dependency inversion Principle 

----------------------------------------------------------------------------------------------------
#### Single Responsibility Principle:
1. Every class shuld have a single responsibility.
2. Responsibility should be encapsultated by the class. 
3. It would be a bad design to couple two things that change for different reasons at different times.

----------------------------------------------------------------------------------------------------
#### Open/Closed Principle
1. Software entities should be open for extension but closed for modification.

----------------------------------------------------------------------------------------------------
#### Liskov Substitution Principle:
1. Behavioral Subtyping.
2. Let q(x) be a property provable about objects x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T. 
3. Liskov's notion of a behavioral subtype defines a notion of substitutability for mutable objects; that is, if S is a subtype of T, then objects of type T in a program may be replaced with objects of type S without altering any of the desirable properties of that program (e.g., correctness). 

----------------------------------------------------------------------------------------------------
#### Interface Segregation Principle:
1. No client should be forced to depend on methods it does not use.
2. Role Interfaces.

----------------------------------------------------------------------------------------------------
#### Dependency inversion Principle:
1. High-level modules should not depend on low-level modules. Both should depend on abstractions. 
2. Abstractions should not depend upon details. Details should depend upon abstractions.

----------------------------------------------------------------------------------------------------
#### Decorators:
1. A decorator is a design pattern.
2. Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.
3. Used instead of Inheritance.
4. A decorator is an object that encloses a component object.
5. conforms to interface of component so its presence is transparent to clients.
6. forwards (delegates) requests to the component.
7. performs additional actions before or after forwarding.

---------------------------------------------------------------------------------------------------
#### GRASP Principles:
1. Low Coupling/High Cohesion.
2. Creator.
3. Controller.
4. Expert.
5. Pure Fabrication.
6. Indirection.
7. Polymorphism.
8. Protected Variations.

----------------------------------------------------------------------------------------------------
#### Software Design Patterns:
1. Singleton Pattern.
2. Factory method Pattern.

----------------------------------------------------------------------------------------------------

#### Note : "clone" copies all of the singleton classes of a Object whereas "dup" does not.

----------------------------------------------------------------------------------------------------
#### Prototype based Development.

1. No Distinction between classes and Objects.
2. Inherit both state and the behaviour.
3. Experiment

```ruby
animal = Object.new

def animal.no_of_feet=(feet)
  @number_of_feet = feet
end

def animal.no_of_feet
  @number_of_feet
end

## Cat is of type animal
cat = animal.clone
```
----------------------------------------------------------------------------------------------------
#### Class based development.

Inheritance  is_a relationship

1. to_s method to beautify the object output
2. class << self
3. (num & 1) == 0
4. extend makes your module methods available as class methods

```ruby
module Logger
  def log
  end
end

class Animal
  extend Logger ## Truck.log
  include Logger ## Truck.new.log
end
```

----------------------------------------------------------------------------------------------------
#### Hook Methods (included?) Module

----------------------------------------------------------------------------------------------------
#### Blocks 

1. a = lambda { |a| a+1 }
2. b = Proc.new { |a| a+1 }
3. c = proc { |a| a+1 } #Deprecated

```ruby
def convert(&block)
  block
end
```

----------------------------------------------------------------------------------------------------
#### Diff between Procs and Lamdas
1. Lambda check arguments whereas proc does not. (Argument Error).
2. Lambda is like a method call whereas Proc is similar to Parallel Assignment.
3. Return returns from enlcosing method Proc -- Lambda returns from the call (Exits from Proc).
4. Proc is like a incline code whereas Lambda is like a method.

----------------------------------------------------------------------------------------------------
#### Bindings : 
 1. Context in which the code is executing.
 2. "binding" is a kernel method.

"eval" second parameter is a binding.

```ruby
def simple(param)
  lvar = "lvar"
  binding
end

b = simple(99)

eval "puts param", b
```
----------------------------------------------------------------------------------------------------
#### Binding Encapsulates : 
1. self
2. Local Variables
3. any associated block
4. return stack

----------------------------------------------------------------------------------------------------
##### Binding example

Ruby creates a binding to save value of n

```ruby
def n_times
  lambda { |val| n * val }
end

two_times = n_times(2)
puts two_times.call(3)
```
----------------------------------------------------------------------------------------------------
#### Metaprogramming

###### define_methods to create dynamic methods on fly.

1. instance_eval
2. class_eval (alias : module_eval) can only be called on classes and modules

##### instance_eval declares a class method
##### and class_eval declares an instance method

----------------------------------------------------------------------------------------------------
#### Things you can do : 
1. ignore privacy Acess private methods and instance varaibles.
2. create methods without using closures .
3. define stuff in classes given a class object.
4. create DSL's in a block.

----------------------------------------------------------------------------------------------------
#### Closures :

‘A function or a reference to a function together with a referencing environment. Unlike a plain function, closures allow a function to access non-local variables even when invoked outside of its immediate lexical scope.’ – Wikipedia

```ruby
def counter
  n = 0
  return Proc.new { n+= 1 }
end

a = counter
a.call            # returns 1
a.call            # returns 2

b = counter
b.call            # returns 1
a.call            # returns 3
```

----------------------------------------------------------------------------------------------------






