![googliser](http://i.imgur.com/yahgjDC.png) googliser.sh
---
This is a BASH script to perform fast image downloads sourced from **[Google Images](https://www.google.com/imghp?hl=en)** based upon a user-specified search-phrase. In short - it's a web-page scraper that feeds a list of image URLs to **[Wget](https://www.gnu.org/software/wget/)** to download images concurrently. The idea is to build a picture of that phrase. 

(This is an expansion upon a solution provided by ShellFish **[here](https://stackoverflow.com/questions/27909521/download-images-from-google-with-command-line)** and has been updated to handle Google page-code that was changed in April 2016.)

---
**Description:**

1. The user supplies a search-phrase and some other optional parameters on the command line. 

2. A sub-directory is created below the current directory with the name of this search-phrase.

3. **[Google Images](https://www.google.com/imghp?hl=en)** is then queried and the results saved.

4. The results are parsed and all image links are extracted and saved to a list file in this sub-directory. Any links for **YouTube** and **Vimeo** are removed.

5. The script then iterates through this list and downloads the first [**n**]umber of available images into this sub-directory. Up to **1000** images can be requested. If an image is unavailable, the script skips it and continues downloading until it has obtained the required amount of images or its failure-limit has been reached. 

6. Lastly, a thumbnail gallery image is built using **[ImageMagick](http://www.imagemagick.org)'s montage**.

---
**Notes:**

I wrote this scraper so that users do not need to obtain an API key from Google to download multiple images. It uses **[Wget](https://www.gnu.org/software/wget/)** as I think it's more widely available than alternatives such as **[cURL](https://github.com/curl/curl)**.

Thumbnail gallery building can be disabled if not required. As a guide, I built from 380 images (totalling 70MB) and created a single gallery image file that is 191MB with dimensions of 8,004 x 7,676 (61.4MP). This took **montage** 10 minutes to render on my old Atom D510 CPU :)

When the gallery is being built, it will only create a thumbnail from the first image of a multi-image file (like an animated .gif).

This script will need to be updated from time-to-time as Google periodically change their search results page-code. The last functional check of this script by me was on 2016-06-10. 

The latest copy can be found **[here](https://github.com/teracow/googliser)**.  

Script icon has been used from **[here](http://www.iconarchive.com/show/social-inside-icons-by-icontexto/social-inside-google-icon.html)**. Please support the artist!

Suggestions / comments / bug reports / advice (are|is) most welcome. :) **[email me](mailto:teracow@gmail.com)**

---
**Usage:**

    $ ./googliser.sh [PARAMETERS] ...

Allowable parameters are indicated with a hyphen then a single character or the alternative form with 2 hypens and the full-text. Parameters can be specified as follows:

`-n` or `--number [INTEGER]`  
Number of images to download. Default is 25. Maximum is 1000.  

`-p` or `--phrase [STRING]`  
**Required!** The search-phrase to look for. Enclose whitespace in quotes e.g. *"small brown cows"*  

`-f` or `--failures [INTEGER]`  
How many download failures before exiting? Default is 40. Enter 0 for unlimited (this may try to download all results - so only use if there are many failures). In rare circumstances, it is possible for the script to show more failures than this. Worst case would be reported as high as `((failures - 1) + concurrency)`. The inevitable consequence of concurrent downloads. :) 

`-c` or `--concurrency [INTEGER]`  
How many concurrent image downloads? Default is 8. Maximum is 40. **Larger is not necessarily faster!**

`-t` or `--timeout [INTEGER]`  
Number of seconds before retrying download. Default is 15. Maximum is 600 (10 minutes).

`-r` or `--retries [INTEGER]`  
Retry download of each image this many times. Default is 3. Maximum is 100.

`-u` or `--upper-size [INTEGER]`  
Only download image files that are reported by the server to be smaller than this many bytes. Some servers do not report file-size, so these will be downloaded anyway and checked afterward. Enter 0 for unlimited size. Default is 0 (unlimited).

`-l` or `--lower-size [INTEGER]`  
Only download image files that are reported by the server to be larger than this many bytes. Some servers do not report file-size, so these will be downloaded anyway and checked afterward. Default is 1000 bytes. I've found this useful for ignoring files sent by servers that send HTML instead of the JPG I requested. :)

`-i` or `--title [STRING]`  
Specify a custom title for the gallery. Default is to use the search-phrase.

`-g` or `--no-gallery`  
Don't create thumbnail gallery. Default always creates a thumbnail gallery after downloading images.

`-h` or `--help`  
Display this help then exit.

`-v` or `--version`  
Show script version then exit.

`-q` or `--quiet`  
Suppress display output. Error messages are still shown.

`-d` or `--debug`  
Append debug info to file. Default is no debug file output. If selected, debugging output is appended to 'googliser-debug.log' in working directory. Great for finding out what external commands and parameters were used!

**Usage Examples:**

    $ ./googliser.sh -p "cows"
This will download the first 25 available images for the search-phrase *"cows"*

    $ ./googliser.sh --number 250 --phrase "kittens" --concurrency 10 --failures 0
This will download the first 250 available images for the search-phrase *"kittens"* and download up to 10 images at once and ignore the failures limit.

    $ ./googliser.sh --number 56 --phrase "fish and chips" --upper-size 50000 --lower-size 2000 --failures 0 --debug
This will download the first 56 available images for the search-phrase *"fish and chips"* but only if the image files are between 2KB and 50KB in size, ignore the failures limit and write a debug file.

---
**Samples:**

    $ ./googliser.sh --phrase "kittens" --upper-size 100000 --lower-size 2000 --failures 0
![kittens](http://i.imgur.com/vm1eisrh.jpg)

    $ ./googliser.sh -n 400 -p "cows" -u 200000 -l 2000 -f 0
![cows](http://i.imgur.com/SMV2BInh.jpg)  
For this sample, only 249 images were available out of the 381 search results returned. 132 images failed as they were unavailable or their file-size was outside the limits I set. The gallery image is built using however many images are downloaded.

---
**Return Values ($?):**  

0 : success!  
1 : required external program unavailable.  
2 : specified parameter incorrect - help / version shown.  
3 : unable to create sub-directory for 'search-phrase'.  
4 : could not get a list of search results from Google.  
5 : image download aborted as failure-limit was reached.  
6 : thumbnail gallery build failed.

---
**Known Issues:**

- (2016-06-10) - None at the moment.

---
**Work-in-Progress:**

- (2016-06-10) - Stuff & junk...
 
---
**To-Do List:**

- separate temp files for list download success/fail?
- need way to cancel background procs when user cancels. Trap user cancel?
- ignore results for .php in list?
- limit download results? 