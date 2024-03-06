# get-pics-from-url
This program downloads the jpg pictures of a given website URL. First, the program downloads the whole html of the site, and then parses the text to get the jpg. and then applies wget to  the list of the so parsed pictures. Finally, it removes all pictures whose file size is less than 100k.


```sh
GetPicsFromURL is a software to download JPG pictures from a website, using its URL through wget.
by F. Gonzalez, CA 27/02/2015

How to use it: ./get-pics-from-url.sh URL
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


## THE CODE
```sh
#!/bin/bash
###################################################################
#  This program downloads the jpg pictures of a given website URL #
#  First, the program downloads the whole html of the site, and   #
#  then parses the text to get the jpg. and then applies wget to  #
#  the list of the so parsed pictures. Finally, it removes all    #
#  pictures whose file size is less than 100k.                    #
###################################################################


size=50    # Eliminate all files less than $size kb of weight
file=$1
format="jpg"
pattern="src"   # Usually src or href
pattern2=""      # Additional search pattern 
prefix=""
debug=false
remove=false
replace=false
rem_pat=()
rep_pat=()

if [ $# == 0 ] || [ "$1" == "-h" ] || [ "$1" == "--help" ] ; then
   echo -e "\nGetPicsFromURLis a software to download JPG pictures from a website, using its URL."
   echo -e "Depends on wget."
   echo -e "by F. Gonzalez, CA 27/02/2015\n"
   echo "How to use it: $PWD/$0 URL"
   echo "EXAMPLES"
   echo "Usage: $0 www.site.com"
   echo "Usage: $0 www.site.com/subfolder"
   echo "Usage: $0 www.site.com --size 100"
   echo "Usage: $0 www.site.com --remove \"folder1/thumbs/\""
   echo "OPTIONS"
   echo "     	-s,--size N     Eliminates all pics less than N kilobytes.                  "
   echo "     	-d,--debug      Look the list of pics, to check which are the importat URLs."
   echo "     	-p,--pattern    Usually 'src' or 'href', pattern is the first letters of URLS with pictures."
   echo "     	-p2,--pattern2  Additional filter after applied after -p."
   echo "     	-f,--format     jpg, png or any other pic format."
   echo "     	-r,--remove     Remove a string from the URL."
   echo "     	-R,--replace    Replace a string in the URL."
   echo "     	--prefix        Explicitely insert the http://www.website... prefix if not included in the url."
   exit
fi

while [ $# -gt 0 ]; do
 case $1 in
  -s,--size)
   size=$2
   shift
   ;;
  -p|--pattern)
   pattern=$2
   echo "Looking for pattern= $pattern"
   shift
   ;;
  -p2|--pattern2)
   pattern2=$2
   echo "Looking for pattern= $pattern"
   echo "Looking also for pattern= $pattern2"
   shift
   ;;
  -r|--remove)
   remove=true
   rem_pat+=($2)                         # append to array
   echo "Removing $2 from the URL"
   shift
   ;;
  -R|--replace)
   replace=true
   rep_pat+=($2 $3)                         # Replace string $1 by string $2
   echo "Replacing $2 by $3 in the URL"
   shift
   shift
   ;;
  -f|--format)
   format=$2
   shift
   ;;
  --prefix)
   prefix=$2
   shift
   ;;
  -d|--debug)
   debug=true
   shift
   ;;
  **)
   ;;
 esac
 shift # This makes $4=$5, $5=$6...
done


list=$(wget "$file" -q -O - | tr ' ' '\n' | grep -i -o "\<.*$format\>")     # List of all jpg
if [ "$(echo $list |grep "$pattern")" == "" ]; then pattern="href"; fi      # Change src=... by href=... or other
n=$(echo "$list" | grep "$pattern"|wc -l)                            # Number of jpg found in $pattern

# Remove string
if [ "$remove" == true ]; then
 n=${#rem_pat[@]}
 for i in `seq 1 $n` 
 do 
  var="${rem_pat[`echo $i-1|bc`]}"
  list=$(echo "$list" | sed -e "s/$var//g")
 done
fi
# Replace string
if [ "$replace" == true ]; then
 n=${#rep_pat[@]}
 for i in `seq 1 2 $n`  # each 2 arguments
 do 
  var="${rep_pat[`echo $i-1|bc`]}"
  var2="${rep_pat[`echo $i|bc`]}"
  list=$(echo "$list" | sed -e "s/$var/$var2/g")
 done
fi


# Print the list to debug and then exit, if problems are present:
if [ "$debug" == true ]; then echo "$list" | grep "$pattern" | grep "$pattern2" ; exit; fi


#--------------------#
# DOWNLOAD WITH WGET #
#--------------------#
#echo "$list" | grep "$pattern" | sed 's/\/\//\/\/ /'  | awk -v format=$format '{print "wget -c --no-use-server-timestamps -t 2 \""$2"\"","-O",NR"."format}'  | sh 
#echo "$list" | grep "$pattern" | grep "$pattern2" | sed 's/\"/ /g'  | awk -v format=$format -v pfx=$prefix '{print "wget -c --no-use-server-timestamps -t 2 \""pfx$2"\"","-O",NR"."format}'  | sh 
echo "$list" | grep "$pattern" | grep "$pattern2" | sed 's/\/\// /g'  | awk -v format=$format -v pfx=$prefix '{print "wget -c --no-use-server-timestamps -t 2 \""pfx$2"\"","-O",NR"."format}'  | sh 

echo -e "\n\n$n PICS DOWNLOADED"

# Eliminamos la basura que pese menos de 100 kb
find . -type f -size "-"$size"k" -exec rm {} +

# Cambiamos los nombres por 1.jpg, 2.jpg, etc, en el orden en q se bajaron
#n=1; for i in $(ls -tr); do echo mv "$i" "$n.jpg"; n=$(( $n+1 )); done

# Eliminar duplicados (en este punto, los nombres son 1.jpg, 2.jpg, ..., 10.jpg, etc)
echo "ELIMINATING DUPLICATES..."
md5sum * | sort -k1 | uniq -w32 -d | awk '{$1="";print}' | awk '{$1=$1;print "rm","\""$0"\""}'| sh

# Convertir las 1.jpg a 01.jpg, etc
ls -tr ?.$format | awk '{print "mv",$1,00$1 }'|sh 
ls -tr ??.$format | awk '{print "mv",$1,0$1 }'|sh 

# Anteponer el nombre de la carpeta
ls *.$format | awk -v pwd="$(basename "$PWD")" '{print "mv",$1,"\""pwd,"-",$1"\""}' | sh

ls *.$format| wc -l| awk '{print $1,"PICS KEPT"}'
```
