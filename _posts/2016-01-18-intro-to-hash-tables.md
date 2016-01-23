---
layout: post
title: Intro To Hash Tables
---

A hash table is an incredibly useful data structure we use throughout our daily lives, whether we know it or not.  We use them to store relational data and quickly manipulate and interact with that data.  Before we delve into what they actually are, though, lets think about a potential use case.

Imagine we have a directory of users.  In this directory, we might store a user's name and phone number.  A primitive method of storage for this information would be an array of arrays, and might look something like this:

{% highlight js %}
//Our directory storage array
var users = [
  ["John Smith", "8756452234"],
  ["Jane Johnson", "3412009876"],
  ["Beck Wethers", "7032228795"],
  ["Tim Stevenson", "9872346363"]
];
{% endhighlight %}

Each inner array pertains to a particular user.  The first item is the name, and the second is that user's phone number.  We could in theory store much more information inside these arrays (email, password, payment info, location), but for now lets keep it simple.  Imagine we want to find the phone number for a particular user, we might write a function like this:

{% highlight js %}
//Name and the user storage are the parameters of our function
var findPhoneNumber = function (name, userDirectory) {
  //We would first iterate over all the items in the directory
  for (var i = 0; i < userDirectory.length; i++) {
    //Get the current users information
    var user = userDirectory[i];
    //Check if the users name is equal to the name passed in
    if (user[0] === name) {
      //If so, return the phone number
      return user[1];
    }
  }

  //If the name was never found, return a message 
  return "Sorry! " + name + "is not in the directory";
};

//EX: findPhoneNumber("Beck Wethers", users) --> returns "7032228795"
{% endhighlight %}

It took our function 3 iterations to find and return Beck's phone number.  Not too terrible, but what if our user directory had thousands, or even millions of items?  In this implementation we must examine each user's data and check if the name matches up with our search parameter.  This means potentially we might have to check every single user in the directory before we find the right one.  With this logic, deleting and modifying a users information would also take the same amount of time.  These operations have O(n) linear time complexity, where n is the total number of users in the directory.

What if there was a way we could perform all those operations instantly?  If we could know where the user lives in storage before we even perform the lookup? 

## Enter hash tables. 

A basic hash table can perform these main operations (insertion, retrieval, deletion) in constant time, O(1).  It achieves this speed through the help of a hashing function.

There are a number of different hashing functions, but a basic one takes an arbitrary string as its input, and outputs a number.  It uses an algorithm to determine a numerical value for your input string based on each of its characters, then performs an operation (like modulus) on that number and outputs the result.  

If you pass the same string into the hashing function, it will always return the same number.  Further, every distinct input string you pass to the hashing function will produce a unique output.  (There are a couple exceptions to this, but I'll go over that topic in another post, just know that most hash tables handle these exceptions quite gracefully).

![Silvrback blog image](https://silvrback.s3.amazonaws.com/uploads/5308aa32-c704-4c82-96e5-2238ed2ef39f/Screen%20Shot%202015-09-28%20at%206.56.07%20PM_large.png)

The storage for our hash table will again take the form of an array of arrays.  However this time we will place items in storage at precise locations, as determined by our hashing function.  Lets walk through an example of placing one of our user's phone numbers into storage.  Lets take  ["John Smith", "8756452234"].  First we would pass the string "John Smith" into our hashing function, which would output an integer, lets say 4 for example.  We would then place the array containing John's name and number at the 4th index in our storage array.  See below for a basic implementation of a hash table with insertion and retrieval: 

{% highlight js %}
//Example hashing function
var hashingFunction = function (input, limit) {
  var hash = 0;
  for (var i = 0; i < input.length; i++) {
    hash = (hash<<5) + hash + input.charCodeAt(i);
    hash = hash & hash; 
    hash = Math.abs(hash);
  }
  return hash % limit;
}

//Hash table with insertion and retrieval. NO collision 
//resolution or storage buckets. 
var HashTable = function () {
  //Define our hash table as an object
  var obj = {};
  //Initialize our storage object as an empty array
  obj.storage = [];
  //Set an initial max size
  obj.limit = 10;

  //A method for inserting items into storage
  obj.insert = function (input, value) {
    //Find hash key for given input
    var hash = hashingFunction(input, obj.limit);
    //Insert our input/value into the storage array
    obj.storage[hash] = [input, value]
  }

  //A method for retrieving items from storage
  obj.retrieve = function (input) {
    var hash = hashingFunction(input, obj.limit);
    return obj.storage[hash][1];
  }
 //return the instance of the hash table.  It will have access
 //to the methods and properties defined above.
  return obj;
}

var myNewHashTable = HashTable();
myNewHashTable.insert("John Smith", "8756452234");
myNewHashTable.retrieve("John Smith") -> returns "8756452234"
{% endhighlight %}

When we want to lookup John's phone number in the future, we can simply pass his name into our hashing function, then grab the item from storage with the given index.  Deletion would work the same way.  We assume the hashing function takes a negligible amount of time, and we know that retrieval of an item from an array is a constant time O(1) operation, hence we get our constant time hash table operations!



Out in the world, you will see hash tables implemented in different types of computer memory and database storage.  They are very useful for sets of data that you need to modify and interact with frequently.

Like all data structures, hash tables have drawbacks which must be accounted for.  Some examples might be when your hashing function produces the same key for two different input strings, so two pieces of data will be stored at the same location.  Technically, this will slow down your insertion/retrieval operations, but fortunately most hash table implementations have dynamic resizing, which determines when the table has become overpopulated, expands the size of the storage array, and redistributes its contents, maintaining an even distribution.  In another post, I'll talk all about hash table resizing and collision resolution.


 