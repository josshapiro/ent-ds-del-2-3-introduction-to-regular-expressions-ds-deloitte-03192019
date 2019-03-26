
# An Introduction to Regular Expressions


## Introduction
We now have most of the pieces in place to clean up data. We can import a csv file into a Pandas DataFrame, we can "look" at the data using methods like `head()`, `info()`, `describe()` (for numerical columns) and `value_counts()` (for categorical columns). We also know how to do basic clean up operations like fixing capitalization, and how to save the results to another file for additional analysis or processing.

However, sometimes we need to do more complicated clean up of data. For example, we might need to strip dollar signs from a list of revenue figures to make them easier to work with (e.g. to calculate the mean), take a set of phone numbers and format them in a consistent manner (both to make them easier to read and to make it easier to check for overlapping numbers across records), or find all of the occurances of a particular word or phrase in a larger piece of text to (say) find all of the email addresses contained within a larger document.

[Regular Expresssions](https://en.wikipedia.org/wiki/Regular_expression) are the swiss army knife for cleaning up strings. They can be used to do almost anything, but they can be complicated and take a long time to get good at using. In this lesson, we'll introduce the basic syntax for applying Regular Expressions to data stored in DataFrames and will provide you with some "recipes" for common transformations. You certainly won't be a Regular Expression guru by the end of the lesson, but at least you'll know what they are and will have a few trusted Regular Expressions for solving some common problems.

## Objectives
You will be able to:
* Explain common types of problems that Regular Expressions can be used to solve
* Apply Regular Expressions to data stored within a DataFrame
* Reformat US phone numbers to a consistent format using a Regular Expression
* Find all of the occurances of one or more phrases in a larger piece of text using a Regular Expresssion

## Accessing Regular Expressions in Jupyter Notebook

Lets start from the "top down" by introducing some of the common methods in Jupyter Notebooks that allow you to apply Regular Expressions (often called "RegEx's" - *pronounced "rej-exes"*) to your data.

### The `str` Attribute
The first thing you need to know about working with text within a DataFrame (or a Series) is the `str` attribute (short for "string"). It provides you access to common string methods that you can apply to a column of data within your DateFrame (not only will this help with Regular Expressions - it'll even simplify the code we learned in the last couple of lessons).

Let's start by importing some sample company data and looking at it:


```python
import pandas as pd
df1 = pd.read_csv("sample1.csv")
df1.head()
```

You should recognize this from the last lab (just with some additional columns). Let's have a look at the `value_counts()` for the EntityType column:


```python
df1["EntityType"].value_counts()
```

And this time, let's fix this to all have the same capitalization using the `str` attribute and the `upper()` method to make all of the data upper case using a single, simple line of code, and then let's look at the value counts to confirm that they've been fixed:


```python
df1["EntityType"] = df1["EntityType"].str.upper()
df1["EntityType"].value_counts()
```

Perfect! Note that calling the `upper())` method on the `str` attribute didn't in itself change the DataFrame, so we had to tell it to take the transformed EntityType values and to save those to `df1["EntityType"]`. 

Some methods change (or some people say "mutate") a variable, others create a new variable instead. These string methods create a new variable, but don't change the original DataFrame. It just means you have to remember to write the code to assign the new variable to the old column like the code above. 

*If that all sounds confusing, just remember to copy the code above when solving these kinds of problems! A lot of good programming is just cutting and pasting code, seeing what it does, and tweaking it a bit until it does what you want it to do!*

Great! Take a minute in the cell below and play around with some of the string methods. Try `lower()` on the EntityType column to see if you can get the same results, but with all of the entity names lower cased . . .

### `replace()`

There are a whole set of methods you can access within the `str` attribute - here is some [documentation](https://pandas.pydata.org/pandas-docs/stable/user_guide/text.html). One that is worth playing with is the `replace()` method. It returns a copy of the object with all matching occurrences of the RegEx pattern provided replaced by whatever replacement text. Here is the [documentation](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.str.replace.html#pandas.Series.str.replace), but let's look at some practical examples.

Lets say we wanted to take a bunch of numbers with dollar signs and commas and remove all of the non-numeric characters so we could more easily turn them into integers for doing math. If you look at the data we imported into df1, you'll notice that there is an AnnualRevenues column *(use the cell below to run the `head()` method on `df1`).*

But if you look at the data type of the column (try calling `df1.info()` in the cell below) you'll see it's just an "object", not some kind of numeric type like and int64. So that means, doing math on it is going to be problematic (e.g. what is the mean or median revenues for this collection of companies).

Here is some code to remove all of the non-numeric characters from the AnnualRevenues field. Run the cell below to see what it does.


```python
df1["AnnualRevenues"] = df1["AnnualRevenues"].str.replace("[^0-9.]", "")
print(df1["AnnualRevenues"])
```

Perfect! You'll see that we got rid of all of the commas and dollar signs, leaving just numbers. Now, what did we just do? Well, let's break it down...

`df1["AnnualRevenues"] = df1["AnnualRevenues"].str.replace("[^0-9.]", "")`

Remember, we start by evaluating the right hand side of the expression:

`df1["AnnualRevenues"].str.replace("[^0-9.]", "")`

On the right hand side we're taking the AnnualRevenues column of the DataFrame (`df1["AnnualRevenues"]`), and we're calling the replace method on the str (string) attribute (`str.replace()`).

And we're passing in two attributes to the `replace()` method: `"[^0-9.]"` and `""`

The first attribute (`"[^0-9.]"`) is a RegEx that returns any character that is not (the not is the `^` character) a number between 0 and 9 (`0-9`) or a period (`.`). The square braces (`[]`) are used to make a "list" of items to match. So for example, the RegEx `[ab]` would match any a's or b's. The RegEx `[14t]`would match any 1's, any 4's and any lower case t's. Can you guess what the RegEx `[pq]` would do? Yup - it'd match every p and q in a string (talk about minding your . . .), and what about `[^8h]`? Well, that would match every single character in a string that was not either an 8 or an h. 

The second attribute (`""`) is what we are asking the replace method to replace any of those characters that are not a 0-9 or a period with - which is just empty quotes. So we're basically saying go through each of these strings. For any character that isn't numeric (0-9 or a period) replace it with nothing - i.e. get rid of it. And that leaves us with just the numbers. If we'd make the second attribute a `"hello"` it would have replaced all of the non-numeric characters with the word "hello" instead. Fun, but not quite as useful!

And then finally we say on the left `df1["AnnualRevenues"] = `, so that's saying "take this new set of AnnualRevenues which only contains numbers, and save that to the AnnualRevenues column in the DataFrame - overwriting what we had there in the past. Congratulations! You just used your first RegEx!

### Digging into RegEx's
Let's get a little more practice with using RegEx's. To run RegEx's on strings in Python, you need to import the `re` library, and instead of the `replace()` method we used in the DataFrame, we'll use a method called `sub()` that does the same thing - but for strings:


```python
import re
phrase = "hello world"
re.sub("h", "", phrase)
```

Try playing with the cell above for a while. Can you get it to:
* Remove the "l"'s instead of the h?
* Remove all of the vowels *(hint, you'll need the square braces)*
* Remove all of the characters that are **not** vowels *(remember to use the `^` within your square braces)*
* Remove all of the characters that are **not** vowels, but still leave in the space

We could spend weeks diving into all of the details of RegEx's, but hopefully you are at least starting to get a sense of how they work... Here is the documentation for [Regular Expression operations in Python](https://docs.python.org/3/library/re.html) if you want to learn a little more on your own!

### Finishing Up The Last Problem

Before we took some time to experiment with RegEx's, we were in the process of turning a collection of Annual Revenues from strings to numbers. Let's finish up this process of taking the column that is a string (in this case annual revenues containing dollar signs and commas) and turning it into an integer which will be easier to work with mathematically.

If you try running the cell below, you'll see that the DataFrame is still storing the revenues as a data type of "object" rather than an integer, so we have the right kind of information, but we haven't told the DataFrame to store it as the right data type yet:


```python
df1.info()
```

So the final little bit of clean up we need to do is to change the type of the column in the DataFrame, and for various arcane reasons, trying to turn it straight into an integer doesn't work, so you actually need to use the code below to first turn it into a floating point number and *then* covert it to an integer. But when you do, it works and you'll see the `info()` now shows this as a int64 (a type of integer), not an object. Give it a try!


```python
df1["AnnualRevenues"] = df1["AnnualRevenues"].astype('float').astype('int64')
df1.info()
```

And there we go - it's now saved as an integer (int64 to be specific) which means we can now (for example) see information about the data when we call `describe()`


```python
df1.describe()
```

It's using [scientific notation](https://en.wikipedia.org/wiki/Scientific_notation) because some of the numbers are quite big, but at least we can see the things like the minimum value, the maximum value and the mean revenues (9.698182e+07 - or just under $100M).

*Cheat sheet for scientific notation - 9.6e+07 is the way most programming languages write what we'd call 9.6 x 10 to the power of 7. Just move the decimal place seven points to the right and you'll get the way we'd normally write down the number. So 9.6e+01 is 96. 9.6e+03 is 9600, and 9.6e+07 is 96000000 - which is 96,000,000.* 

### Keeping on Keeping On

One of the things it's worth noting is that sometimes with coding you have to be persistent. First we needed to figure out what method to call (`.str.replace()`), then we needed to figure out what RegEx would capture all of the non-numeric characters (`[^0-9.]`), then we found we'd cleaned up the data but not changed the data type in the DataFrame, and then we found out that we actually had to call some slightly strange code to first turn the column into a floating point number, and only then could we convert it to integers:

`df1["AnnualRevenues"] = df1["AnnualRevenues"].astype('float').astype('int64')`

Don't worry if that feels like a lot - it is. But with programming, as long as you keep on taking small steps in the right direction you'll (usually) get where you need to go. So take a breath, and lets look at some other good uses for Regular Expressions!

### Reformatting Phone Numbers

If you look again at the list of companies stored in df1, you'll see we have a contact phone number for each company, but they are not all in the same format. There are different ways we could solve this. The simplest would be to remove all non-numeric characters. Just to make the output easier to read I've added `[:10]` which retrieves just the first 10 entries. For a real project, you'd obviously apply it to all of the cells:


```python
df1["ContactPhone"][:10].str.replace("[^0-9]", "")
```

That actually works pretty well. So if we did want to apply that to the whole DataFrame and save the changes, we'd just run


```python
df1["ContactPhone"] = df1["ContactPhone"].str.replace("[^0-9]", "")
print(df1["ContactPhone"])
```

### Finding Text in a Larger String

Lets say we wanted to search a really long document and retrieve a liat of all of the email addresses within in. Let's start by opening and reading the file, and then printing the first 500 characters just to have a sense of whats contained within the variable:


```python
content  = open("web-page-with-emails.txt", "r").read()
print(content[:500])
```

    Email address
    From Wikipedia, the free encyclopedia
    Jump to navigationJump to search
    
    Example of an email address
    An email address identifies an email box to which email messages are delivered. A wide variety of formats were used in early email systems, but only a single format is used today, following the standards developed for Internet mail systems since the 1980s. This article uses the term email address to refer to the addr-spec defined in RFC 5322, not to the address that is commonly used;


There's one of two ways of doing the next step. If you have a lot of time on your hands, are super smart and really want to learn Regular Expresssions, you could go get a [book](https://www.amazon.com/Mastering-Regular-Expressions-Jeffrey-Friedl/dp/0596528124), spend a couple of weeks practicing and then try to write the perfect RegEx for returning email addresses. 

Alternatively, you could search on Google for something like "python regex find email addresses in string" and get to a page like [this](https://stackoverflow.com/questions/17681670/extract-email-sub-strings-from-large-document) on StackOverflow and then copy, paste and tweak the code to get:


```python
emails = re.findall(r'[\w\.-]+@[\w\.-]+', content)
print(emails)
```

    ['John.Smith@example.com', 'local-part@domain', 'jsmith@example.com.', 'john.smith@example.org', 'local-part@domain', 'John..Doe@example.com', 'bah@domain', 'foo@domain', 'fred@domain.', 'john.smith@example.com', 'john.smith@example.com.', 'jsmith@example.com', 'JSmith@example.com', 'john.smith@example.com', 'john.smith@example.com.', 'simple@example.com', 'very.common@example.com', 'symbol@example.com', 'other.email-with-hyphen@example.com', 'fully-qualified-domain@example.com', 'sorting@example.com', 'user.name@example.com', 'x@example.com', 'example-indeed@strange-example.com', 'admin@mailserver1', 'example@s.example', 'A@b', 'c@example.com', 'l@example.com', 'right@example.com', 'allowed@example.com', 'allowed@example.com', 'x@example.com', 'tag@example.com', 'joeuser@example.com.', 'bar@example.com', 'foobar@example.com.', 'Pelé@example.com', 'δοκιμή@παράδειγμα.δοκιμή', '我買@屋企.香港', '二ノ宮@黒川.日本', 'медведь@с-балалайкой.рф', 'क@ड']


It's worth noting that capturing email addresses from a document is actually a best efforts activity. The RegEx above is great, but there are probably some unusual but valid email addresses it won't catch, and if someone is trying to obfuscate (hide) their email address (say on a web pag e) there are things they can do to hide it so that it won't be captured using any Regular Expression. However, even an 90% automated solution is often better than trying to solve these problems manually.

## Summary

Regular Expressions are a huge topic, and writing good RegEx's is hard - even for professional software developers. But at least now you know what a RegEx is, the kinds of problems it can be used to solve, and you have some recipes you can save and use for cleaning up numbers (whether revenue numbers or phone numbers) and finding a string in a larger document. Next up, lets take some more sample data and get some practice cleaning up data using some of the methods we learned today!

