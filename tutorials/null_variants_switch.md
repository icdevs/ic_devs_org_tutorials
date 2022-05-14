---
label: Null, variants and the switch statement
order: 400
---

# Null, variants and the switch statement

We have touched null, the optional type and variants softly in the [Understanding types](/tutorials/understanding_types/) tutorial. Now it's time to dive deeper and write some functions that utilize them together with the switch statement.

## Optional type

In the Understanding types tutorial, we have described optional type as a type that we don't know whether we expect a real value of specific type, let's say Text or a null type. 

To examine this, let's write a function called day_of_week that will take Nat as an argument and return the name of the week. We can use the days array from the previous tutorial.

```
let days : [Text] =  ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];

public query func day_of_week(n : Nat) async Text {
	return days[n];
};
```

That was easy! Now what happens if you put a number higher than 6 as an argument? Even if Motoko Playground deploys your code, it doesn't always mean it is safe. You need to take care of this kind of error yourself to prevent your program from failing. 

To fix that, we can use the optional type ?Text, so our function knows that it can also return a null type.

```
public query func day_of_week(n : Nat) : async ?Text {
    if (n <= 6){
        return ?days[n];
    } else {
        return null;
    };
};
```
Notice that ?Text and Text type are different. If your function is supposed to return a ?Text type, you can't just return a Text. In such a case, you would see this error:

```
expression of type
  Text
cannot produce expected type
  ?Text
```
That is why we put a question mark in front of the return type ?days[n]. Notice that in Candid UI you now can see the (opt "Tuesday") value returned instead of just text.

## Switch statement

Switch statement is a very popular concept in many programming languages. We could write this simple switch statement in a function as a warm up just that you get familiar with the Motoko syntax:

```
public query func day_of_the_week(n : Nat) : async Text {
    switch(n){
        case(1) {
            return "Monday";
        };
        case(2) {
            return "Tuesday";
        };
        case(3) {
            return "Wednesday";
        };
        case(4) {
            return "Thursday";
        };
        case(5) {
            return "Friday";
        };
        case(6) {
            return "Saturday";
        };
        case(7) {
            return "Sunday";
        };
        case(_) {
            return "Not a valid index";
        };
    };
};
```
Things get more interesting when we apply switch statement on the optional type. We can use the switch statement to let our program decide what should happen if the optional value is null and what should happen if it's not. 

Let's make a new function called Hello, that will take Text as an optional argument and return a message "Hello, unknown person" if the argument is null and "Hello, /name/" if it is a text.
```
public query func hello(name : ?Text) : async Text {
    switch(name){
        case(null){
            return "Hello, unknown person";
        };
        case(?value){
            return "Hello, " # value;
        };
    };
};
```
To explain this, we have a switch statement that takes the function argument called name, in the case that name is null, it will return unknown person text, if there is something other than null, we are expecting the Text type and we return in concatenation with the greeting.

## Variants

Things will get more interesting, when we put variants into play. Variants give us the opportunity to define a specific set of choices or situations. We already touched variants in the previous tutorial, when we defined a variant type Status that says whether a person is single, married or divorced.

```
public type Status = {
    #single;
    #married;
    #divorced;
};
```
This is a very readable and technically effective way to give users of our program a limited choice. Also your program is much safer as it doesn't need to deal with array indexes, strings, typos and so on.

Let's now bring back the code from the previous [Understanding types](/tutorials/understanding_types/) and write a function that will take a Person type as an argument and uses a switch statement on the status parameter of that person to return a text describing whether the person is single, married or divorce. 

```
actor Person {

	public type Status = {
	    #single;
	    #married;
	    #divorced;
	};

	type Person = {
	    name : Text;
	    surname : Text;
	    age : Nat;
	    status : Status;
	    partner : ?Person; // optional attribute
	};


	public query func get_status(p : Person) : async Text {
	    let name : Text = p.name;
	    if (name == ""){
	        let name = "Person";
	    };
	    switch(p.status){
	        case(#single){
	            return name # " is single.";
	        };
	        case(#married){
	            return name # " is married."
	        };
	        case(#divorced){
	            return name # " is divorced."
	        };
	    };
	};
}
```
Deploy this code in your Motoko Playground project and try to create a person called Alice, 30 years old that is single and call the get_status function. **Notice that Candid UI will let you choose only one of 3 options for the status attribute that we defined in our code.** You should see "Alice is single" message in the output. We have also added a check that if a person decides not to fill the name, we will substitute the name with "Person" string.

To practice variants and the switch statement some more, let's bring up the days from the beginning of this text and try to create a variant representation of the object.
```
// Before – days as Array
let days : [Text] =  ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];

// After – days as Variants
type Day = {
    #Monday;
    #Tuesday;
    #Wednesday;
    #Thursday;
    #Friday;
    #Saturday;
    #Sunday;
}
```
Not to confuse you, these two objects are very different. The variant way might be a better choice when writing your programs as it doesn't give the user options to insert bad inputs. Also your code might be more readable as #Monday is much easier to understand that day[0] where we might not be sure whether we start days from Monday or Sunday or whether the array indexes start from 1 or 0.

Now let's make a function called is_weekend that will take the Day variant and returns a Text "weekend" when the input is Saturday or Sunday or "weekday" when it is one of the other days.
```
actor Day {

	type Day = {
	    #Monday;
	    #Tuesday;
	    #Wednesday;
	    #Thursday;
	    #Friday;
	    #Saturday;
	    #Sunday;
	};

	public query func is_weekend(d : Day) : async Text {
	    switch d {
	        case (#Saturday or #Sunday) "weekend";  
	        case _ "weekday"; 
	    };
	};
}
```
Try to run this code in your Motoko Playground workspace and try to experiment with it. You should immediately notice how this code works better:
1. In the Candid UI you are allowed to input only predefined choices that are straightforward and easy to understand
2. Your Motoko code is also much simpler, you don't need to take care of specific situations such as the user input number that exceeds the index of the array.

## Useful links and resources

Motoko Bootcamp video about variants by Paul Young  
https://www.youtube.com/watch?v=GFenqSGhj7I&

Motoko Documentation page talking about options and errors  
https://smartcontracts.org/docs/current/developer-docs/build/languages/motoko/errors#working-with-optionresult

Motoko Documentation page talking about variants  
https://smartcontracts.org/docs/current/developer-docs/build/languages/motoko/overview#variants

#### The author
> This tutorial was written by Lukas Vozda. If you find some resources out of date, have some suggestions or just want to get in touch. You can find me on Twitter [@lukas_icp](https://mobile.twitter.com/lukas_icp) or Discord LukeVoz#0574.