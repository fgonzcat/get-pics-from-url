# get-pics-from-url
This program downloads the jpg pictures of a given website URL. First, the program downloads the whole html of the site, and then parses the text to get the jpg. and then applies wget to  the list of the so parsed pictures. Finally, it removes all pictures whose file size is less than 100k.
