---
title: 代码中的坏味道
categories: knowledge
---



## Object-Orientation Abusers

All these smells are incomplete or incorrect application of object-oriented programming principles.
### Alternative classes with different interfaces
#### Symptom
Two classes perform identical functions but have different method names.
#### Reason
The programmer who created one of the classes probably didn’t know that a functionally equivalent class already existed.

<!-- more -->

#### Treatment
1. Rename Methods to make them identical in all alternative classes.
2. Move Method, Add Parameter and Parameterize Method to make the signature and implementation of methods the same.
3. If only part of the functionality of the classes is duplicated, try using Extract Superclass. In this case, the existing classes will become subclasses.
4. After you have determined which treatment method to use and implemented it, you may be able to delete one of the classes.
#### PayOff
1. You get rid of unnecessary duplicated code, making the resulting code less bulky.
2. Code becomes more readable and understandable (you no longer have to guess the reason for creation of a second class performing the exact same functions as the first one).
#### Ignore
1. Sometimes merging classes is impossible or so difficult as to be pointless. One example is when the alternative classes are in different libraries that each have their own version of the class.
### Refused bequest
#### Symptom
If a subclass uses only some of the methods and properties inherited from its parents, the hierarchy is off-kilter. The unneeded methods may simply go unused or be redefined and give off exceptions.
#### Reason
Someone was motivated to create inheritance between classes only by the desire to reuse the code in a superclass. But the superclass and subclass are completely different.
#### Treatment
1. If inheritance makes no sense and the subclass really does have nothing in common with the superclass, eliminate inheritance in favor of Replace Inheritance with Delegation.
2. If inheritance is appropriate, get rid of unneeded fields and methods in the subclass. Extract all fields and methods needed by the subclass from the parent class, put them in a new subclass, and set both classes to inherit from it (Extract Superclass).
#### PayOff
1. Improves code clarity and organization. You will no longer have to wonder why the Dog class is inherited from the Chair class (even though they both have 4 legs).
#### Ignore
1. Рефакторинг
2. Паттерны
### Switch statements
#### Symptom
You have a complex switch operator or sequence of if statements.
#### Reason
Relatively rare use of switch and case operators is one of the hallmarks of object-oriented code. Often code for a single switch can be scattered in different places in the program. When a new condition is added, you have to find all the switch code and modify it.
#### Treatment
1. To isolate switch and put it in the right class, you may need Extract Method and then Move Method.
2. If a switch is based on type code, such as when the program’s runtime mode is switched, use Replace Type Code with Subclasses or Replace Type Code with State/Strategy.
3. After specifying the inheritance structure, use Replace Conditional with Polymorphism.
4. If there aren’t too many conditions in the operator and they all call same method with different parameters, polymorphism will be superfluous. If this case, you can break that method into multiple smaller methods with Replace Parameter with Explicit Methods and change the switch accordingly.
5. If one of the conditional options is null, use Introduce Null Object.
#### PayOff
1. Improved code organization.
#### Ignore
1. When a switch operator performs simple actions, there’s no reason to make code changes.
2. Often switch operators are used by factory design patterns (Factory Method or Abstract Factory) to select a created class.
### Temporary field
#### Symptom
Temporary fields get their values (and thus are needed by objects) only under certain circumstances. Outside of these circumstances, they’re empty.
#### Reason
Oftentimes, temporary fields are created for use in an algorithm that requires a large amount of inputs. So instead of creating a large number of parameters in the method, the programmer decides to create fields for this data in the class. These fields are used only in the algorithm and go unused the rest of the time.
#### Treatment
1. Temporary fields and all code operating on them can be put in a separate class via Extract Class. In other words, you’re creating a method object, achieving the same result as if you would perform Replace Method with Method Object.
2. Introduce Null Object and integrate it in place of the conditional code which was used to check the temporary field values for existence.
#### PayOff
1. Better code clarity and organization.
#### Ignore
1. Рефакторинг
2. Паттерны
## Bloaters
Bloaters are code, methods and classes that have increased to such gargantuan proportions that they are hard to work with. Usually these smells do not crop up right away, rather they accumulate over time as the program evolves (and especially when nobody makes an effort to eradicate them).
### Long method
#### Symptom
A method contains too many lines of code. Generally, any method longer than ten lines should make you start asking questions.
#### Reason
Like the Hotel California, something is always being added to a method but nothing is ever taken out. Since it’s easier to write code than to read it, this “smell” remains unnoticed until the method turns into an ugly, oversized beast.
#### Treatment
1. To reduce the length of a method body, use Extract Method.
2. If local variables and parameters interfere with extracting a method, use Replace Temp with Query, Introduce Parameter Object or Preserve Whole Object.
3. If none of the previous recipes help, try moving the entire method to a separate object via Replace Method with Method Object.
4. Conditional operators and loops are a good clue that code can be moved to a separate method. For conditionals, use Decompose Conditional. If loops are in the way, try Extract Method.
#### PayOff
1. Among all types of object-oriented code, classes with short methods live longest. The longer a method or function is, the harder it becomes to understand and maintain it.
2. In addition, long methods offer the perfect hiding place for unwanted duplicate code.
#### Ignore
1. Рефакторинг
2. Паттерны
### Large class
#### Symptom
A class contains many fields/methods/lines of code.
#### Reason
Classes usually start small. But over time, they get bloated as the program grows.
#### Treatment
1. Extract Class helps if part of the behavior of the large class can be spun off into a separate component.
2. Extract Subclass helps if part of the behavior of the large class can be implemented in different ways or is used in rare cases.
3. Extract Interface helps if it’s necessary to have a list of the operations and behaviors that the client can use.
4. If a large class is responsible for the graphical interface, you may try to move some of its data and behavior to a separate domain object. In doing so, it may be necessary to store copies of some data in two places and keep the data consistent. Duplicate Observed Data offers a way to do this.
#### PayOff
1. Refactoring of these classes spares developers from needing to remember a large number of attributes for a class.
2. In many cases, splitting large classes into parts avoids duplication of code and functionality.
#### Ignore
1. Рефакторинг
2. Паттерны
### Primitive obsession
#### Symptom
Use of primitives instead of small objects for simple tasks (such as currency, ranges, special strings for phone numbers, etc.)
#### Reason
Use of constants for coding information (such as a constant USER_ADMIN_ROLE = 1 for referring to users with administrator rights.)
#### Treatment
1. Use of primitives instead of small objects for simple tasks (such as currency, ranges, special strings for phone numbers, etc.)
2. Use of constants for coding information (such as a constant USER_ADMIN_ROLE = 1 for referring to users with administrator rights.)
3. Use of string constants as field names for use in data arrays.
#### PayOff
1. If you have a large variety of primitive fields, it may be possible to logically group some of them into their own class. Even better, move the behavior associated with this data into the class too. For this task, try Replace Data Value with Object.
2. If the values of primitive fields are used in method parameters, go with Introduce Parameter Object or Preserve Whole Object.
3. When complicated data is coded in variables, use Replace Type Code with Class, Replace Type Code with Subclasses or Replace Type Code with State/Strategy.
4. If there are arrays among the variables, use Replace Array with Object.
#### Ignore
1. Code becomes more flexible thanks to use of objects instead of primitives.
2. Better understandability and organization of code. Operations on particular data are in the same place, instead of being scattered. No more guessing about the reason for all these strange constants and why they’re in an array.
3. Easier finding of duplicate code.
### Long parameter list
#### Symptom
More than three or four parameters for a method.
#### Reason
A long list of parameters might happen after several types of algorithms are merged in a single method. A long list may have been created to control which algorithm will be run and how.
#### Treatment
1. Check what values are passed to parameters. If some of the arguments are just results of method calls of another object, use Replace Parameter with Method Call. This object can be placed in the field of its own class or passed as a method parameter.
2. Instead of passing a group of data received from another object as parameters, pass the object itself to the method, by using Preserve Whole Object.
3. If there are several unrelated data elements, sometimes you can merge them into a single parameter object via Introduce Parameter Object.
#### PayOff
1. More readable, shorter code.
2. Refactoring may reveal previously unnoticed duplicate code.
#### Ignore
1. Don’t get rid of parameters if doing so would cause unwanted dependency between classes.
### Data clumps
#### Symptom
Sometimes different parts of the code contain identical groups of variables (such as parameters for connecting to a database). These clumps should be turned into their own classes.
#### Reason
Often these data groups are due to poor program structure or "copypasta programming”.
#### Treatment
1. If repeating data comprises the fields of a class, use Extract Class to move the fields to their own class.
2. If the same data clumps are passed in the parameters of methods, use Introduce Parameter Object to set them off as a class.
3. If some of the data is passed to other methods, think about passing the entire data object to the method instead of just individual fields. Preserve Whole Object will help with this.
4. Look at the code used by these fields. It may be a good idea to move this code to a data class.
#### PayOff
1. Improves understanding and organization of code. Operations on particular data are now gathered in a single place, instead of haphazardly throughout the code.
2. Reduces code size.
#### Ignore
1. Passing an entire object in the parameters of a method, instead of passing just its values (primitive types), may create an undesirable dependency between the two classes.
## Couplers
All the smells in this group contribute to excessive coupling between classes or show what happens if coupling is replaced by excessive delegation.
### Feature envy
#### Symptom
A method accesses the data of another object more than its own data.
#### Reason
This smell may occur after fields are moved to a data class. If this is the case, you may want to move the operations on data to this class as well.
#### Treatment
1. If a method clearly should be moved to another place, use Move Method.
2. If only part of a method accesses the data of another object, use Extract Method to move the part in question.
3. If a method uses functions from several other classes, first determine which class contains most of the data used. Then place the method in this class along with the other data. Alternatively, use Extract Method to split the method into several parts that can be placed in different places in different classes.
#### PayOff
1. Less code duplication (if the data handling code is put in a central place).
2. Better code organization (methods for handling data are next to the actual data).
#### Ignore
1. Sometimes behavior is purposefully kept separate from the class that holds the data. The usual advantage of this is the ability to dynamically change the behavior (see Strategy, Visitor and other patterns).
### Inappropriate intimacy
#### Symptom
One class uses the internal fields and methods of another class.
#### Reason
Keep a close eye on classes that spend too much time together. Good classes should know as little about each other as possible. Such classes are easier to maintain and reuse.
#### Treatment
1. The simplest solution is to use Move Method and Move Field to move parts of one class to the class in which those parts are used. But this works only if the first class truly doesn’t need these parts.
2. Another solution is to use Extract Class and Hide Delegate on the class to make the code relations “official”.
3. If the classes are mutually interdependent, you should use Change Bidirectional Association to Unidirectional.
4. If this “intimacy” is between a subclass and the superclass, consider Replace Delegation with Inheritance.
#### PayOff
1. Improves code organization.
2. Simplifies support and code reuse.
#### Ignore
1. Рефакторинг
2. Паттерны
### Incomplete library class
#### Symptom
Sooner or later, libraries stop meeting user needs. The only solution to the problem – changing the library – is often impossible since the library is read-only.
#### Reason
The author of the library hasn’t provided the features you need or has refused to implement them.
#### Treatment
1. To introduce a few methods to a library class, use Introduce Foreign Method.
2. For big changes in a class library, use Introduce Local Extension.
#### PayOff
1. Reduces code duplication (instead of creating your own library from scratch, you can still piggy-back off an existing one).
#### Ignore
1. Extending a library can generate additional work if the changes to the library involve changes in code.
### Message chains
#### Symptom
In code you see a series of calls resembling $a->b()->c()->d()
#### Reason
A message chain occurs when a client requests another object, that object requests yet another one, and so on. These chains mean that the client is dependent on navigation along the class structure. Any changes in these relationships require modifying the client.
#### Treatment
1. To delete a message chain, use Hide Delegate.
2. Sometimes it’s better to think of why the end object is being used. Perhaps it would make sense to use Extract Method for this functionality and move it to the beginning of the chain, by using Move Method.
#### PayOff
1. Reduces dependencies between classes of a chain.
2. Reduces the amount of bloated code.
#### Ignore
1. Overly aggressive delegate hiding can cause code in which it’s hard to see where the functionality is actually occurring. Which is another way of saying, avoid the Middle Man smell as well.
### Middle man
#### Symptom
If a class performs only one action, delegating work to another class, why does it exist at all?
#### Reason
This smell can be the result of overzealous elimination of Message Chains.
#### Treatment
1. If most of a method’s classes delegate to another class, Remove Middle Man is in order.
#### PayOff
1. Less bulky code.
#### Ignore
1. A middle man may have been added to avoid interclass dependencies.
2. Some design patterns create middle man on purpose (such as Proxy or Decorator).
## Dispensables
A dispensable is something pointless and unneeded whose absence would make the code cleaner, more efficient and easier to understand.
### Comments
#### Symptom
A method is filled with explanatory comments.
#### Reason
Comments are usually created with the best of intentions, when the author realizes that his or her code isn’t intuitive or obvious. In such cases, comments are like a deodorant masking the smell of fishy code that could be improved.
#### Treatment
1. If a comment is intended to explain a complex expression, the expression should be split into understandable subexpressions using Extract Variable.
2. If a comment explains a section of code, this section can be turned into a separate method via Extract Method. The name of the new method can be taken from the comment text itself, most likely.
3. If a method has already been extracted, but comments are still necessary to explain what the method does, give the method a self-explanatory name. Use Rename Method for this.
4. If you need to assert rules about a state that’s necessary for the system to work, use Introduce Assertion.
#### PayOff
1. Code becomes more intuitive and obvious.
#### Ignore
1. When explaining why something is being implemented in a particular way.
2. When explaining complex algorithms (when all other methods for simplifying the algorithm have been tried and come up short).
### Duplicate code
#### Symptom
Two code fragments look almost identical.
#### Reason
Duplication usually occurs when multiple programmers are working on different parts of the same program at the same time. Since they’re working on different tasks, they may be unaware their colleague has already written similar code that could be repurposed for their own needs.
#### Treatment
1. If the same code is found in two or more methods in the same class: use Extract Method and place calls for the new method in both places.
2. If the same code is found in two subclasses of the same level: Use Extract Method for both classes, followed by Pull Up Field for the fields used in the method that you’re pulling up. If the duplicate code is inside a constructor, use Pull Up Constructor Body. If the duplicate code is similar but not completely identical, use Form Template Method. If two methods do the same thing but use different algorithms, select the best algorithm and apply Substitute Algorithm.
3. Use Extract Method for both classes, followed by Pull Up Field for the fields used in the method that you’re pulling up.
4. If the duplicate code is inside a constructor, use Pull Up Constructor Body.
5. If the duplicate code is similar but not completely identical, use Form Template Method.
6. If two methods do the same thing but use different algorithms, select the best algorithm and apply Substitute Algorithm.
7. If duplicate code is found in two different classes: If the classes aren’t part of a hierarchy, use Extract Superclass in order to create a single superclass for these classes that maintains all the previous functionality. If it’s difficult or impossible to create a superclass, use Extract Class in one class and use the new component in the other.
8. If the classes aren’t part of a hierarchy, use Extract Superclass in order to create a single superclass for these classes that maintains all the previous functionality.
9. If it’s difficult or impossible to create a superclass, use Extract Class in one class and use the new component in the other.
10. If a large number of conditional expressions are present and perform the same code (differing only in their conditions), merge these operators into a single condition using Consolidate Conditional Expression and use Extract Method to place the condition in a separate method with an easy-to-understand name.
11. If the same code is performed in all branches of a conditional expression: place the identical code outside of the condition tree by using Consolidate Duplicate Conditional Fragments.
#### PayOff
1. Use Extract Method for both classes, followed by Pull Up Field for the fields used in the method that you’re pulling up.
2. If the duplicate code is inside a constructor, use Pull Up Constructor Body.
3. If the duplicate code is similar but not completely identical, use Form Template Method.
4. If two methods do the same thing but use different algorithms, select the best algorithm and apply Substitute Algorithm.
#### Ignore
1. If the classes aren’t part of a hierarchy, use Extract Superclass in order to create a single superclass for these classes that maintains all the previous functionality.
2. If it’s difficult or impossible to create a superclass, use Extract Class in one class and use the new component in the other.
### Data class
#### Symptom
A data class refers to a class that contains only fields and crude methods for accessing them (getters and setters). These are simply containers for data used by other classes. These classes don’t contain any additional functionality and can’t independently operate on the data that they own.
#### Reason
It’s a normal thing when a newly created class contains only a few public fields (and maybe even a handful of getters/setters). But the true power of objects is that they can contain behavior types or operations on their data.
#### Treatment
1. If a class contains public fields, use Encapsulate Field to hide them from direct access and require that access be performed via getters and setters only.
2. Use Encapsulate Collection for data stored in collections (such as arrays).
3. Review the client code that uses the class. In it, you may find functionality that would be better located in the data class itself. If this is the case, use Move Method and Extract Method to migrate this functionality to the data class.
4. After the class has been filled with well thought-out methods, you may want to get rid of old methods for data access that give overly broad access to the class data. For this, Remove Setting Method and Hide Method may be helpful.
#### PayOff
1. Improves understanding and organization of code. Operations on particular data are now gathered in a single place, instead of haphazardly throughout the code.
2. Helps you to spot duplication of client code.
#### Ignore
1. Рефакторинг
2. Паттерны
### Dead code
#### Symptom
A variable, parameter, field, method or class is no longer used (usually because it’s obsolete).
#### Reason
When requirements for the software have changed or corrections have been made, nobody had time to clean up the old code.
#### Treatment
1. Delete unused code and unneeded files.
2. In the case of an unnecessary class, Inline Class or Collapse Hierarchy can be applied if a subclass or superclass is used.
3. To remove unneeded parameters, use Remove Parameter.
#### PayOff
1. Reduced code size.
2. Simpler support.
#### Ignore
1. Рефакторинг
2. Паттерны
### Lazy class
#### Symptom
Understanding and maintaining classes always costs time and money. So if a class doesn’t do enough to earn your attention, it should be deleted.
#### Reason
Perhaps a class was designed to be fully functional but after some of the refactoring it has become ridiculously small.
#### Treatment
1. Components that are near-useless should be given the Inline Class treatment.
2. For subclasses with few functions, try Collapse Hierarchy.
#### PayOff
1. Reduced code size.
2. Easier maintenance.
#### Ignore
1. Sometimes a Lazy Class is created in order to delineate intentions for future development, In this case, try to maintain a balance between clarity and simplicity in your code.
### Speculative generality
#### Symptom
There’s an unused class, method, field or parameter.
#### Reason
Sometimes code is created “just in case” to support anticipated future features that never get implemented. As a result, code becomes hard to understand and support.
#### Treatment
1. For removing unused abstract classes, try Collapse Hierarchy.
2. Unnecessary delegation of functionality to another class can be eliminated via Inline Class.
3. Unused methods? Use Inline Method to get rid of them.
4. Methods with unused parameters should be given a look with the help of Remove Parameter.
5. Unused fields can be simply deleted.
#### PayOff
1. Slimmer code.
2. Easier support.
#### Ignore
1. If you’re working on a framework, it’s eminently reasonable to create functionality not used in the framework itself, as long as the functionality is needed by the frameworks’s users.
2. Before deleting elements, make sure that they aren’t used in unit tests. This happens if tests need a way to get certain internal information from a class or perform special testing-related actions.
## Change Preventers
These smells mean that if you need to change something in one place in your code, you have to make many changes in other places too. Program development becomes much more complicated and expensive as a result.
### Divergent change
#### Symptom
Divergent Change resembles Shotgun Surgery but is actually the opposite smell. Divergent Change is when many changes are made to a single class. Shotgun Surgery refers to when a single change is made to multiple classes simultaneously.
#### Reason
You find yourself having to change many unrelated methods when you make changes to a class. For example, when adding a new product type you have to change the methods for finding, displaying, and ordering products.
#### Treatment
1. Split up the behavior of the class via Extract Class.
2. If different classes have the same behavior, you may want to combine the classes through inheritance (Extract Superclass and Extract Subclass).
#### PayOff
1. Improves code organization.
2. Reduces code duplication.
3. Simplifies support.
#### Ignore
1. Рефакторинг
2. Паттерны
### Parallel inheritance hierarchies
#### Symptom
Whenever you create a subclass for a class, you find yourself needing to create a subclass for another class.
#### Reason
All was well as long as the hierarchy stayed small. But with new classes being added, making changes has become harder and harder.
#### Treatment
1. You may de-duplicate parallel class hierarchies in two steps. First, make instances of one hierarchy refer to instances of another hierarchy. Then, remove the hierarchy in the referred class, by using Move Method and Move Field.
#### PayOff
1. Reduces code duplication.
2. Can improve organization of code.
#### Ignore
1. Sometimes having parallel class hierarchies is just a way to avoid even bigger mess with program architecture. If you find that your attempts to de-duplicate hierarchies produce even uglier code, just step out, revert all of your changes and get used to that code.
### Shotgun surgery
#### Symptom
Shotgun Surgery resembles Divergent Change but is actually the opposite smell. Divergent Change is when many changes are made to a single class. Shotgun Surgery refers to when a single change is made to multiple classes simultaneously.
#### Reason
Making any modifications requires that you make many small changes to many different classes.
#### Treatment
1. Use Move Method and Move Field to move existing class behaviors into a single class. If there’s no class appropriate for this, create a new one.
2. If moving code to the same class leaves the original classes almost empty, try to get rid of these now-redundant classes via Inline Class.
#### PayOff
1. Better organization.
2. Less code duplication.
3. Easier maintenance.
#### Ignore
1. Рефакторинг
2. Паттерны
