### What does MongoDB import from a JSON file?

The short answer: Depends on form of content.

What MongoDB imports from a JSON file depends on how the
content is found in the JSON file itself. There is no right
or wrong way to put the content, as long as the JSON takes
in valid forms. However, user must be aware that the *form
of content* would affect the imported data.

First, consider a JSON file named "dogs-lines.json" with
the following content.

    { "owner_id": 1002, "dog_name": "Chase" }
    { "owner_id": 1000, "dog_name": "Dash" }
    { "owner_id": 1000, "dog_name": "Lucy" }
    { "owner_id": 1001, "dog_name": "Latte" }
    { "owner_id": 1002, "dog_name": "Zeke" }

There are 5 objects placed in 5 lines. Each of the objects
has two string-value pairs in each lines (the objects are
actually separated by newlines). The content is imported
into MongoDB by running the following command in Terminal.
The result is "imported 5 objects" (correct).

    $ mongoimport -d example -c dogs --file dogs-lines.json 
    connected to: 127.0.0.1
    Sat Aug 19 15:16:27.984 imported 5 objects

Next, consider a JSON file named "dogs-list.json" with the
following content.

    [
        { "owner_id": 1002, "dog_name": "Chase" },
        { "owner_id": 1000, "dog_name": "Dash" },
        { "owner_id": 1000, "dog_name": "Lucy" },
        { "owner_id": 1001, "dog_name": "Latte" },
        { "owner_id": 1002, "dog_name": "Zeke" }
    ]

There are the same 5 objects, however placed between a pair
of square brackets `[ ]` that is called 'array' in JSON (or
'list' in Python). The objects are separated by commas in
respective lines. Run the command again. The result is
"imported 0 objects" (wrong).

    $ mongoimport -d example -c dogs --file dogs-list.json 
    connected to: 127.0.0.1
    Sat Aug 19 15:18:09.254 exception:BSON representation
    of supplied JSON is too large: code FailedToParse: 
    FailedToParse: Expecting '{': offset:0
    ...
    Sat Aug 19 15:18:09.255 check 0 0
    Sat Aug 19 15:18:09.255 imported 0 objects

Why the errors? Because the content has changed its form
from "one object per line" to "an array of objects". The
command works only for content with "one object per line".
To get the correct result, run with `--jsonArray` option.

    $ mongoimport -d example -c dogs --file dogs-list.json --jsonArray
    connected to: 127.0.0.1
    Sat Aug 19 15:19:37.841 imported 5 objects

Finally, consider a JSON file named "dogs-nested-list.json"
with the following content.

    {
        "listing_date": "2017-08-19",
        "listing": [
            { "owner_id": 1002, "dog_name": "Chase" },
            { "owner_id": 1000, "dog_name": "Dash" },
            { "owner_id": 1000, "dog_name": "Lucy" },
            { "owner_id": 1001, "dog_name": "Latte" },
            { "owner_id": 1002, "dog_name": "Zeke" }
        ]
    }

There are the same 5 objects in an array, however placed as
value for a string called "listing". The array is said be
'nested'. Run the command again with `--jsonArray` option.
The result is "imported 1 objects" (huh).

    $ mongoimport -d example -c dogs --file dogs-nested-list.json --jsonArray
    connected to: 127.0.0.1
    Sat Aug 19 15:35:43.120 imported 1 objects

Why does this happen? At top level, two string-value pairs
are found between a pair of curly brackets `{ }`, which is
the syntax for an object. Despite JSON file has nested
array of 5 objects, these are not placed at the top level.
In other words, content at the top level determines how the
objects are counted.

Then, is the result correct? Yes and no.

* Yes, because the form of content makes the JSON file to
  be seen as one object.
* No, because the nested array of objects is expected
  to be imported as separate objects.

In this write-up, I have shown that form of content will
determine what MongoDB imports from a JSON file. Nested
objects should be imported separately, otherwise the
result will return the entire object, which is undesired.

##### Resources

1. [What Is MongoDB?][1]
2. [Introducing JSON][2]
3. [mongoimport - MongoDB Manual][3]
4. [mongoimport - FailedToParse: Expecting '{': offset:0][4]
5. Trials and Errors

##### Disclaimer

This is *not* an introduction to MongoDB. Instead of "how
to import JSON into MongoDB", this write-up is an attempt
to explain "how MongoDB behave against different contents
of JSON file". This write-up is aimed at beginning users
whom I expect to have had some experience using MongoDB
interactive shell, or even using MongoDB via pymongo in
Python programming.

At time of writing, I have had only 10-days experience with
MongoDB, which I have been using alongside Python during my
internship as Research Intern. Under no circumstances that
this write-up was a professional work.

##### Colophon

This write-up "What does MongoDB import from a JSON file?"
was prepared by Mubiin Kimura. Dated 2017-08-19. First
published in 'kmubiin/write-ups' repository on GitHub.

[1]: https://www.mongodb.com/what-is-mongodb
[2]: http://www.json.org/
[3]: https://docs.mongodb.com/manual/reference/program/mongoimport/
[4]: https://stackoverflow.com/q/18620019
