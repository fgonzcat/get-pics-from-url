# get-pics-from-url
This program downloads the jpg pictures of a given website URL. First, the program downloads the whole html of the site, and then parses the text to get the jpg. and then applies wget to  the list of the so parsed pictures. Finally, it removes all pictures whose file size is less than 100k.


```sh
GetPicsFromURL is a software to download JPG pictures from a website, using its URL through wget.
by F. Gonzalez, CA 27/02/2015

How to use it: /Users/fgonzalez/University/Programas/2021-Get-Pics-From-URL/./get-pics-from-url.sh URL
EXAMPLES
Usage: ./get-pics-from-url.sh www.site.com
Usage: ./get-pics-from-url.sh www.site.com/subfolder
Usage: ./get-pics-from-url.sh www.site.com --size 100
Usage: ./get-pics-from-url.sh www.site.com --remove "folder1/thumbs/"
OPTIONS
     	-s,--size N     Eliminates all pics less than N kilobytes.
     	-d,--debug      Look the list of pics, to check which are the importat URLs.
     	-p,--pattern    Usually 'src' or 'href', pattern is the first letters of URLS with pictures.
     	-p2,--pattern2  Additional filter after applied after -p.
     	-f,--format     jpg, png or any other pic format.
     	-r,--remove     Remove a string from the URL.
     	-R,--replace    Replace a string in the URL.
     	--prefix        Explicitely insert the http://www.website... prefix if not included in the url.
```
