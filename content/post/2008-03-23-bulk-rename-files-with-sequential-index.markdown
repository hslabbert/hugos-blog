---
date: 2008-03-23 06:31:17+00:00
slug: bulk-rename-files-with-sequential-index
title: Bulk Rename Files with Sequential Index
tags:
- .net
- file management
- powershell
- scripting
- windows
---

I am pretty sure I'm not the only one who wants something more descriptive than DSC1900298.JPG to name my digital photos. And yes, I know that Windows Explorer allows you to rename pictures en masse, but I don't like the convention they have chosen in that the first file is named _[common name].JPG_, then the subsequent files are named _[common name] (2).JPG_ and so on and so forth.

I had a few requirements for how I wanted to go about this:



	
  1. Get rid of the parentheses. If I will be posting those pics online anywhere, I wanted to keep the names as free of special characters as I can.

	
  2. Number the first file. The Windows Explorer route does not number the first file when doing bulk renames. This is easy enough to do manually, but I just don't want to bother.

	
  3. Keep a constant number of digits in the index number. I want the renaming process to take into account how many pictures there are and adjust the number of index digits accordingly. If there are fewer than 10 files/images, then only 1 digit is required (e.g. 1, 2, 3, 4...9). If there are between 10 and 99 files (inclusive), then two digits are required (01, 02, 03...10, 11, 12...99). I think you get the idea. Windows definitely doesn't do that.


<!-- more -->I thought that Powershell would be the best tool for the job, but I still struggled a bit. I eventually found the following would do the trick:

```
$prefix = "[SomePrefix]"
$files = Get-ChildItem
$id = 1
$files | foreach { Rename-Item -Path $_.fullname -NewName ( $prefix + ((($id++).tostring()).padleft(($files.count.tostring()).length) -replace ' ','0' ) + $_.extension) }
```

Note that everything after the `$files | `is all on one line of code, but it might appear in the post as being spread across multiple lines.

The Explanation:

The first line saves the contents of the current directory into an array. `$id = 1` just resets our index counter so that the first file will be numbered 1. Now the fun stuff begins!

We pipe the `$files` array into a foreach loop to process through each of the files in the directory. The foreach loop just contains the Rename-Item cmdlet. We select the full path of the item passed through the pipeline, `$_.fullname`, as the original item to rename. The -NewName item gets a bit complicated.

As we will be including a whole bunch of items, we first open with parentheses, "(". The new name will start with a string we have to define earlier, `$prefix`. If, for instance, your collection of pictures in the working directory is from the slick cabling job you just did on one of your racks, you can set `$prefix = "SlickCablingJob"`. This will result in files named something like `SlickCablingJob01.jpg`, etc.

The next portion has a few opening parentheses, and this is because we will be performing a bunch of operations on our `$id` variable, which is our index number. We use `$id++` because we want to `$id` variable to increment each time we pass through the foreach loop and use the variable.

To set the number of digits, I used the PadLeft() method. The PadLeft() method basically adds spaces to the left side of whatever string you are processing so that the resultant string is of a length that you specify. If, for instance, I have a string of "four" saved to the variable `$string` and used the command $string.PadLeft(10), the resultant string would be "four" with 6 spaces to its left, for a total of 10 characters in our string.

Unfortunately, PadLeft() is not a method of the Int32 class, of which our `$id` variable is an instance, but it is a method of the String class. So, we first have to convert the `$id` variable to a string using `$id.ToString()`. I like to keep things clean, so I used `($id.ToString()).PadLeft()` to first wrap up the result of `$id.ToString()` and then apply the PadLeft() method.

Now, PadLeft() requires an Int32 parameter to tell it how many places to pad to the left of the string. This is where some of the magic comes in. Remember how we saved the list of files in the directory to the array `$files`? Well, that array has a property `Count`, which is a count of the number of items in the array, in our case the number of files in the array.

We count the number of items in our directory listing array using `$files.Count`. Since we want to see how many characters are in the highest number in our file list (which is equal to the number of files in the `$files` array), we can use the `Length` property. The `Length` property, when used on Strings, will return the number of characters in the string as an Int32 value. Using our `$string` of "four" from earlier, `$string.Length` will be equal to and Int32 value of 4, since there are four characters in the string "four". If `$string` were "one", `$string.Length` would be 3, since there are three characters in the string "one". This can get confusing, so you might want to try it out with a few different values to get the hang of it.

Let's say that we have 45 files in our directory. `$files.Count `would be an Int32 value of 45. The length property cannot be applied to Int32 values, so we have to convert our value to a string using `$files.Count.ToString()`. In our example of 45 files in our directory, `$files.Count.ToString()` would return a string value of "45". The length property of that value would be the Int32 value 2. This is because the string "45" has two characters in it. If we had between 1 and 9 files (inclusive), the `$files.Count.ToString()` value would be 1 because there is only one character in the numbers 1, 2, 3, 4, 5, 6, 7, 8, and 9 since they are single-digit numbers. If we had between 10 and 99 files in our list, we would have a double-digit number, and hence `$files.Count.ToString()` would have a value of 2. Again, you might want to play around with this if you are unfamiliar with the concept.

So far, then, we have `(($id++).ToString()).PadLeft(($Files.Count.ToString()).Length`, which takes the index number, converts it to a string, and then pads it with spaces to its left so that the total of characters is equal to the number of characters in the count of the number of files in the list. It might be best to demonstrate with another example to bring it all together. Let's say that we have one hundred (100) files that we are renaming, and the foreach loop is working through the first file.

The `$id` variable is set to the Int32 value of 1. This is converted to a string that reads "1". Since we have 100 files in our list, `$files.Count` will have an Int32 value of 100. In order to count the number of characters in that value, we convert it to a string using `$files.Count.ToString()`, then count the number of characters using `$files.Count.ToString().Length`. This gives us an Int32 value of 3, since there are three characters in the number 100. PadLeft() then pads our `$id` of 1 so that it has three characters in total, as a one with 2 spaces to its left. We're almost there!

The last part of the indexing replaces the spaces from PadLeft() to zeroes, so that the files will show up chronologically in a directory listing. This is done through the `-replace ' ','0'` part of the line, which replaces spaces with zeroes. For our example, we would no longer have " 1" but rather "001" (insert 007 joke here). Finally, my third requirement is met!

All that is left to then append the original file extension to the filename. Since we have the original file in the pipeline, we can access its extension using `$_.Extension`. And voila! We have our bulk rename tool!

Now, as the code stands, we have to first define our `$prefix` variable before using it, and it only acts on files in the current working directory. I threw this into a function with a parameter of `$prefix`, so that I can easily define the prefix and call the function conveniently from within the directory that contains the files I want to rename. You could also specify an additional parameter of `$directory` to the function to be able to work on directories other than your current working directory, as long as you change `$files = Get-ChildItem $directory`.

Here is the function as it sits in my current Powershell profile:

```
function Rename-Bulk($prefix)
{
    $files = Get-ChildItem
    $id = 1
    $files | foreach { Rename-Item -Path $_.fullname -NewName ( $prefix + ((($id++).tostring()).padleft(($files.count.tostring()).length) -replace ' ','0' ) + $_.extension) }
}
```

To define the function in your Powershell session, either paste the code above at the Powershell prompt, or paste it into your Powershell profile to have it available each time you start Powershell (be sure to remove any line breaks; remember that everything after `$files |` is one one line)!

To add the extension I had mentioned to specify a directory other than your current working directory, use the following instead:

```
function Rename-Bulk($prefix,$directory=$pwd)
{
    $files = Get-ChildItem $directory
    $id = 1
    $files | foreach { Rename-Item -Path $_.fullname -NewName ( $prefix + ((($id++).tostring()).padleft(($files.count.tostring()).length) -replace ' ','0' ) + $_.extension) }
}
```

That will select the current working directory by default if the second parameter is not specified, while still giving you the option to specify an alternate directory.

As always, if there is something I've missed (it's getting late!) or if you have something to add, please let me know.

Bitcoin tip address for this post: 1M4ydWcEPsUXWrQYN254UrfkAbVGBbpZB8
