Title: Introduction to Haskell
Date: 2015-01-15
Category: Haskell
Tags: haskell, introduction
Slug: introduction-to-haskell
Authors: Tushar Tyagi
Summary: Introduction To Haskell

### Helper methods:
		ghci> methodname
where methodname can be:
:info  -- prints information about the name passed
:type  -- gives type information about the type
:set   -- sets a variable for the environment
:unset -- unsets a variable


### Lists
* Notation is: [elem1, elem2, elem2] where each elem belongs to the same type. Lists cannot have different data types.
* Numeric lists can use _enumeration notation_ .. so that \[1..5] expands to \[1,2,3,4,5]. This can be useful for creating infinite lists (which are evaluated lazily).
* Different from python lists because those are mutable, these aren't.
* ++ operator concatenates two lists. 
		ghci> [1] ++ [2] 
		[1,2]
* : is cons operator which adds an element to the front of a list.
		ghci> 1 : [2]
		[1, 2]
* Strings are list of characters. We can use the ++ and : operator on them too. The following cons-es a char to a list which is made by concatenating two strings (each is a list of Char)
		ghci> 'a' : "b" ++ "c"
		"abc"
* head and tail functions return the first and all-except-first elements of a list respectively.
* take and drop functions take in number, list as parameters and return either the list of n numbers, or list of all numbers except first n resp. 
	
### Tuple
* Notation is (elem1, elem2, elem3) where each elem may belong to a different type. 
* The order of elements matter a lot.
* (123, "hello") is (Num, [Char]) whereas ("hello, 123) is ([Char], Num) where these aren't equal
* () is a zero element tuple. There is no one element tuple in the language. Two and three element tuples are common.
* As per the book, we shouldn't use them when number of elements are too many; limit the number to 3-4.
* fst and snd functions work on two element lists and return the first and second element of the list.

### Type system
* Statically and strongly typed language. 
	* The first means that the types are checked at compile time.
	* The second means that the types are strong, i.e. one type cannot be converted into by auto-coercian (without casting).
* Char, Bool, Int, Integer, Double are some common types.
* In Haskell, variable are bound to expressions which cannot change their values during program's execution. In imperative languages, variable refer to a memory location which can contain different data during different time.
This snippet will work fine in so many other languages, but will cause an error in Haskell
		x = 10
		x = 11

### Language Constructs
#### if statement
	if predicate
	then some_expression
	else some\_other_expression
* The two expressions in if and then branches have to evaluate to the same type. We cannot have if's expression evaluating to Bool and then's expression evaluating to Int.
* Expressions are evaluated lazily. What the parent expression gets is a promise, called _thunk_. As long as it's value is not needed, thunk isn't evaluated. e.g (1 + 2) will keep passing around the code unless it reaches some point where the value is needed.

#### case construct  ####
	
		case value in
			value1 -> returnValue1
			value2 -> returnValue2
			_      -> returnValueN

#### Polymorphism
* Haskell support parametric polymorphism where the function body can work with any type of parameter.
		ghci> :type fst
		fst :: (a, b) -> a
	Here the function fst doesn't really care about what the type of its parameters is. In actuality, it has no way of finding it out.
* Subtype polymorphism -- where the child class can override the methods of parent class -- isn't supported.
* Coercian polymorphism -- which allows a variable of one type to be automatically converted to other type -- isn't supported.

### Functions
* function signature: The arrow in the signatures are right-associative.
		take :: Int -> [a] -> [a]
	actually means
		take :: Int -> ([a] -> [a])
	which means take is a function which takes a value of type int, and returns another function which takes in a list and returns another list. This function returning another function helps in currying.

### Data Type

#### Creating new data types
The creation of data types takes the following form:
		-- Inside the hs file
		data TypeConstructorName = ValueConstructorName args
									body of the constructor
the TypeConstructorName and the ValueConstructor name may or may not be different. Type constructor is used only in type declaration or type signature; during actual code, the value constructor is used.

		-- hs file
		data BookInfo = Book Int String [String]
						deriving (Show)

		myBook = Book 123 "hello" ["author"]

		ghci> :info myBook
		ghci> :info BookInfo

What we are creating is [Algebraic Data Types](https://www.haskell.org/haskellwiki/Algebraic_data_type), called so because the definition of the datatype takes in either the sum or the product of the parameters:
		
		data Type1 = Int Double -- This is product, both are required
		data Type2 = Int | Double -- This is sum, any one is required
		
The benefit of Algebraic Data Types is that it helps in maintaining type safety. Two Algebraic Data Types are always different, thanks to their type names, even if their signatures are same. So

		-- hs file
		data Type1 = Type1 Int Double
					deriving (Eq)
		data Type2 = Type2 Int Double
					deriving (Eq)
		
		ghci> let type1 = Type1 1 1.5
		ghci> let type2 = Type2 1 1.5
		ghci> type1 == type2 -- Error since the types are different
		

will not be equal. 

This is in contrast with two tuples of the same shape, which are equal.

		ghci> (1, 1.5) == (1, 1.5)
		True

The sum of Algebraic Data Types are used when we want to give the option of picking up one value. Bool is one such example.

		data Bool = True | False
		

#### Pattern Matching
A function can be broken down into set of expressions which are executed based on a some specific pattern. This is called _pattern matching_.

The negate function:
		-- hs file
		negate True = False
		negate False = True

		ghci> negate False
		True

The parameter passed is matched against those which are defined for that function, and the value on the right side of = is returned. This way  kind of short-circuits because once a match is found, the next statements are not evaluated.

Pattern matching can be used to create one-liner methods as well. 

		-- hs file
		myPower (a, b) = a ** b

		ghci> myPow 2 3
		8.0

Or to create accessor methods for value constructors
		
		bookID (Book id title authors) = id
		bookTitle (Book id title authors) = title
		bookAuthors (Book id title authors) = authors

		ghci> bookID (Book 3 "Some name" ["Author"])
		3

The compiler can get the types of using the signatures of our accessopr functions.

		ghci> :type bookID
		bookID :: BookInfo -> Int

We can do away with writing so much, and simply providing what is absolutely neccessary in accessor functions, using wild card pattern. The WC pattern takes in anyvalue, and is represented by _

		-- hs file
		bookID (Book id _ ) = id
		bookTitle (Book _ title _ ) = title
		bookAuthors (Book _ _ authors) = authors

		ghci> bookID (Book 3 "Some name" ["Author"])
		3

In addition to this, we can use what is called as the _record syntax_ which creates the accessors for us, and one advantage is that we can pass the parameters in whatever order we prefer. The syntax for this is:

		-- hs file
		data Customer = Customer {
			customerID :: CustomerID,
			customerName :: String,
			customerAddress :: Address
		} deriving (Show)

		ghci> customer = Customer {
			customerID = 123456,
			customerAddress = ["Hyd", "India"],
			customerName = "Tushar"
		}


		ghci> customerID customer
		123456

		ghci> customerName customer
		"Tushar"


__Maybe__ is a type constructor which is used in places where we are expecting what are otherwise called "null" values in imperative languages.

The RWH explaination was a bit confusing, but after banging enough head, I came to know what it's used for.
The definition of Maybe is like:
		
		Maybe a = Just a
				| Nothing
				
The usage is pretty simple, once it is understood. 


## [Num](https://www.haskell.org/tutorial/numbers.html) ##
I stumbled upon the different types of / division operators that behave differently for different numbers. Num type in haskell can be of tyo types:
* Integral 
* Fractional
and while other arithmatic operators (+, -, *, negate, abs) are shared across both these types, there are two divisions:
function div for whole number division
/ operator for fractional division

while the other operators can be mixed and matched (as long as they aren't hardwired from Num to something else), 
for division we have to use either div or / based on the need.

        -- a fractional number and integral number can be added, subtracted, multiplied
        ghci> 1 + 1.5
        2.5
        ghci> 1 - 1.5
        0.5
        ghci> 1 * 1.5
        1.5

        -- hs file
        divBySame :: Int -> Int
        divBySame a = a / a -- Won't work since these are hardcoded to be int
        divBySame a = (div a a) -- This will work
        


### List of Common Functions ###

* break (Prelude)
break :: (a -> Bool) -> [a] -> ([a], [a])
Takes in a predicate function and a list and returns two lists, broken apart at that first element where the function returns True.

* isUpper, isLower (Data.Char)
is* :: Char -> Bool

* unlines (Prelude)
unlines :: [String] -> String
Takes in a list of strings and returns a concatenated string.

* isPrefixOf, isInfixOf, isSuffixOf (Data.List)
is*Of :: String -> String -> Bool
Takes in string1 and string2 and finds if the string1 starts, is part of, or ends string2.

* head
head :: [a] -> a
Returns the first element

* tail
tail :: [a] -> [a]
Returns everything except the first element

* length
length :: [a] -> Int
Returns the length of the list

* null
null :: [a] -> Bool
Returns if the list is empty

* last
last :: [a] -> a
Returns the list element of the list. This is spiritually connected to head.

* init
init :: [a] -> [a]
Returns the list of elements excluding the last. This is spiritually connected to tail.

A lot of methods working with lists throw error in case an empty list is passed to them.

	* Using lenght is inefficient since it walks through the whole lists, and also because H supports infinite lists, which can screw up things.

* The best approach is to use null check in if statements
		fun1 xs = if null xs then ... else ...


		fun2 (x:_) = do something with x
		fun2 [] = do something else

* concat
concat :: [[a]] -> [a]
Takes in a list of lists, and concatenates them; also removes one level of nesting.

* reverse
reverse :: [a] -> [a]
Reverses the list

* all, any
f :: (a -> Bool) -> [a] -> Bool
Takes in a predicate and list. Return true if all or any one of the elements matches the predicate.

* take, drop
f :: Int -> [a] -> [a]
Returns a sublist by taking or dropping first n elements from the list.

* splitAt
f :: Int -> [a] -> ([a], [a])
Returns a two-tuple by splitting the input list across the given index.

* takeWhile
f :: (a -> Bool) -> [a] -> [a]
Takes in a predicate function and returns a list. This list is generated by taking elements as long as the predicate returns True.

* dropWhile
f :: (a -> Bool) -> [a] -> [a]
Takes in a predicate function and returns a list. This list is generated by removing elements as long as the predicate returns True. When predicate returns false, the remaining elements are returned.

* span, break
f :: (a -> Bool) -> [a] -> ([a], [a])
break splits the list when predicate returns true.
split splits the list when predicate returns false.
The element around which list is split goes to the second sublist.

* elem
elem :: (Eq a) => a -> [a] -> Bool
Returns true if the passed element is part of the passed list.

* filter
filter :: (a -> Bool) -> [a] -> [a]
Takes in a predicate and creates a list of all the elements which passed the predicate test.

* zip
zip :: [a] -> [b] -> [(a, b)]
Takes in two lists and creates a new list containing pairs of elements picked from both the lists. Length is equal to the length of the shorter list.

* zipWith
zipWith :: (a -> b -> c) -> [a] -> [b] -> [c]
Zips the two list into a third list, the function is then applied to the elements of the resulting list (after zip) and the results are provided in the final list.

* words, unwords
words :: String -> [String]
unwords :: [String] = String

words takes in a string and splits it across whitespace
unwords takes in list of strings and joins then using a single space


* map
map :: (a -> b) -> [a] -> [b]
map returns a new list resulting from applying the function on every element of the list


* foldl, foldr
foldl :: (a -> b -> a) -> a -> [b] -> a
Takes in an accumulator function, an init value for the accumulator and a list. Do something to every element of the list, updating an accumulator as we go, then return the accumulator.
