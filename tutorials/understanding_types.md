---
label: Understanding types
order: 500
---

# Understanding types

Now that we learned about [primitive types](/tutorials/primitive_types/) in the previous lecture, let's move on to non-primitive types. In this tutorial, we are going to demonstrate how to declare them and build them with primitive types. 

This is a natural process as non-primitive values can be **user-defined**, meaning that you can build objects or structures that combines multiple primitive types together.

## Non-primitive types

Let's quickly go through non-primitive types that are present in Motoko and how to declare them.

### 1. Tuples

Tuples are simple objects that can contain multiple elements. These elements can be different types but you can't change number of elements after you declare it. You could create a simple tuple type to store name and surname for a person:

```
type person = (Text, Text);

let me = ("John", "Doe");
```
Then, you could easily create a tuple that will store not only name but also an age of the person as **you can mix different types in a tuple:**

```
type person = (Text, Text, Nat);

let me = ("John", "Doe", 25);
```
However, for more complex situations you should consider creating your own object with named parameters which would be more readable. Tuples are usually used for simple use cases only. 


### 2. Arrays

Arrays are used for storing multiple values. Unlike tuples, they can contain only one specific type. You can define mutable and immutable arrays. The difference is that mutable arrays can be updated and immutable arrays cannot. 

#### 2.1 Immutable arrays

Here is how you define an immutable array:

```
let days : [Text] =  ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
```
Now as we defined an immutable array with the type of text, Motoko playground will not let you include any other value than text, also it doesn't let you to update the value:

```
days[0] := "Tuesday"; // Not ok - Expected mutable target assigment
days := ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]; // Not ok - The same problem
```
Let's make a first excersise. Try to write a function called getDay, that will take a Nat as an argument and returns a name of the corresponding day.

```
public query func getDay(n : Nat) : async Text {
	return days[n];
};
```
Here we go. That was simple. You can notice 2 things – function is very well defined with types in the input as well as in the output, if you try to put anything else then natural number it is not going to work. 

Also, we are using the **query** keyword in the function declaration which will make the function execution much faster. It is a very good idea to use query function anytime we are not trying to update values in the backend of our app.

#### 2.1 Mutable arrays

Declaration of a mutable array is very similar to immutable array. The key difference is using the var keyword. Let's try to declare an immutable array.
```
var days : [Text] = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
```
Notice that now Motoko Playground allows you to reassign the array with different values. For example:

```
days := ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday" ];
```
You could also change the number of elements inside freely. However, you still can't change an element inside individually as only the whole array is mutable but the content inside is not:
```
days[0] := "Tuesday"; // Not ok - expected mutable assigment target
```
If you woul like to make a totally mutable array, you need to use the **var** keyword also inside the declaration. Here is the full working code of a mutable array:
```
actor Days {
    var days : [var Text] = [var "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];

    days[0] := "Tuesday"; // Ok 

    days := [var "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday" ]; // Ok

    public query func getDay(n : Nat) : async Text {
        return days[n];
    };

};
```
If you run this code in you Motoko Playground and try tu run the function in the Candid UI, you should get "Sunday" for the index of 0.

If you'd like to make immutable array mutable and vice versa, you need to use functions from the [Array section](https://smartcontracts.org/docs/current/references/motoko-ref/array) of the Motoko base library. Specifically it is thaw function to make array mutable and freeze function to make it immutable.

```
import Array "mo:base/Array";

actor Days {
    
    let days : [Text] =  ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];

    var days_mutable = Array.thaw<Text>(days);

    days_mutable := [var "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

    days_mutable[0] := "Hello";
};
```
Notice that after using the thaw function, we can assign another array to the variable or even assign a value to a specific index. This wouldn't be possible with immutable array.

### 3. Records
In Motoko, similarly to arrays, records objects can be mutable and immutable. Here is how you define an immutable object Person with attributes name, surname and age:

```
type Person = {
    name : Text;
    surname : Text;
    age : Nat;
};

let person1 : Person = {
    name = "John"; 
    surname = "Doe"; 
    age = 30;
};
```
Analogically, if you want to have your object type mutable, you should use the **var** keyword. Let's redesign the example above so we are able to update the age of a person, name and surname cannot be changed:
```
type Person = {
    name : Text;
    surname : Text;
    var age : Nat;
};

let person1 : Person = {
    name = "John"; 
    surname = "Doe"; 
    var age = 30;
};

person1.age += 1; // Ok
person1.name := "Adam"; // Not ok, expected mutable assigment target
```
### 4. Variants
When declaring an object above, we have to define all of the parameters so we have a person that has name AND surname AND age. Variants perform more like OR as we are expecting to get one of the values inside. Let's create a variant type Status that will have options single, married and divorced.
```
type Status = {
	#single;
	#married;
	#divorced;
};
```
Now let's update the Person record type that we created before and add status as another parameter. We expect the status parameter to be updated later on in time.
```
type Status = {
    #single;
    #married;
    #divorced;
};

type Person = {
    name : Text;
    surname : Text;
    var age : Nat;
    var status : Status;
};

let person1 : Person = {
    name = "John"; 
    surname = "Doe"; 
    var age = 30;
    var status = #single;
};

// update
person1.age += 1;
person1.status := #married;
```
With this design, we assured that each person can only have one of the available statuses.
### 5. Functions
We've been already messing around simple functions in this and previous tutorials so you probably know how to define them. Let's go through some more examples so you can recognize different types of functions in Motoko.

#### Public and private functions

Most of the functions that we defined so far were public, as there was no specific reason to make them private. Private functions are used for specific situations when you **don't want to expose the private function to the public interface of your canister.**

Example of a public function in Motoko, using the keyword **public** before the function definition:
```
public query func getDay(n : Nat) : async Text {
    return days[n];
};
```
If you would like to make your function private, you use the **private** keyword. You will notice that private functions also have an _ underscore character in their names. This is not required by the language itself but recommended as a naming convention.

```
private query func _getDay(n : Nat) : async Text {
    return days[n];
};
```
#### Functions from Motoko base library

Like we presented in the array section, there is plenty of functions available in the [Motoko base library](https://smartcontracts.org/docs/current/references/motoko-ref/array). There is a section for each type. These functions are called from the imported module like we did with the thaw function:

```
import Array "mo:base/Array";
<some code>
var days_mutable = Array.thaw<Text>(days);
<some code>
```
Notice that some functions require to define the data type in the <> brackets. 

Go through the base library and check what is inside for each type. These functions are going to save your time when writing your programs. However, please respect that Motoko is still quite a fresh language and there is not yet as many functions available as for more mature languages.

#### Anonymous functions

Anonymous functions are also known as lambdas. You define them for generic purpose in your program to complete small tasks. They are not ment to be part of the public interface.
```
func add(x : Int, y : Int) : Int = x + y;
```

### 6. Async values

When canisters or actors communicate with each other on the Internet Computer, it happens asynchronously. Messagess are processed one after another (in sequence) in an isolated way.

Let's have a look at an example from the [documentation page](https://smartcontracts.org/docs/developers-guide/basic-syntax-rules.html#_actors_and_asynchronous_messaging):
```
let result1 = await service1.computeAnswer(params);
let result2 = await service2.computeAnswer(params);
finalStep(result1, result2)
```
**Await** keyword is used to say to our program that we are expecting a value from the other service but the value will be returned in the future, that is why we have to wait for it. After the future value is assigned to our variables result1 and result2 the program knows that it can call the finalStep function.

### 7. Optional values

Optional values are used when we are expecting a specific type but it can be **null** as well. Optional values are defined with question mark and can be mostly found in the record type declaration or in functions. 

Let's extend the Person model from above and add another parameter partner that will point to another person instance. Naturally, single persons will not have a partner so we must keep this parameter optional.
```
public type Status = {
    #single;
    #married;
    #divorced;
};

type Person = {
    name : Text;
    surname : Text;
    var age : Nat;
    var status : Status;
    var partner : ?Person; // optional attribute
};

let person1 : Person = {
    name = "John"; 
    surname = "Doe"; 
    var age = 30;
    var status = #single;
    var partner = null; // we can assign null when there is no partner
};

let person2 : Person = {
    name = "Charlie"; 
    surname = "Adams"; 
    var age = 28;
    var status = #single;
    var partner = ?person1; // in real situation, we should also update the status of person1 and person2 and set partner for person1 if the relationship is symmetrical 
};
```
Optional values play significant roles also in function definitions as function can as well expect or return a null value. There will be more detailed tutorial on that soon.

## Useful links and resources

Motoko Bootcamp video about variables, functions and types by Albert Du  
https://www.youtube.com/watch?v=4YX41Nm7Wx8

Motoko Bootcamp video about variants by Paul Young  
https://www.youtube.com/watch?v=GFenqSGhj7I&