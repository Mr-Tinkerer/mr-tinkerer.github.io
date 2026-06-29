---
date: 2026-06-29T00:00:00
title: Improving the Blog Script
description: The original script for automating the blog publication had some flaws appear. So I fix them in here.
slug: improving-the-blog-script
tags:
  - Bash Script
  - Obsidian
  - Automation
  - Fixing
  - Streamlined
  - Website
toc: true
---
## The Problem
So after I created the [blog site](/posts/blogs-streamlined), and publish the [gaming VM blog](/posts/frictionless-gaming-vm), I realised that the blog script that I wrote has some flaws with it. The (current) problems is the following:
- The script doesn't manually update the date
- The script doesn't give me a preview of the blog before publishing.
- Stop when the script encounters an error.
- Stop when there is no description/tags.
## Blog's Templates
Something else that I forgot to mention in the blog is that I used the [Obsidian's Templater](https://community.obsidian.md/plugins/templater-obsidian) plugin to automatically insert the font matter with the correct info. Now I did use Claude for this, since at the time, I didn't felt like learning Templater's syntax. But this is what the template file looks like. Just make sure to put it in the `Templates` folder in order for the plugin to be able to use it.

``` markdown
---
date: 2026-06-29T00:00:00
title: <% tp.file.title %>
slug: <% tp.file.title.toLowerCase().replace(/\s+/g, '-') %>
description:
tags:
toc: true
---
## The 





## Citations
> [1] 
```

## Previewing the Blog.
This one was quite stupid for me to somehow miss when I wrote the blog script. regardless, it shouldn't be that much of an issue for me to implement. The first thing that I had to do is to get the script to auto launch the script. A quick google search told me that I would just need to run `xdg-open <URL>` in the terminal to make it auto launch the correct browser to the URL.<sup>[1]</sup> 

![](Pasted%20image%2020260627223629.png)
*The command opening the URL in the correct browser from the terminal.*

After testing that the command worked, modified the script to include the command after it generates the Hugo site. 

```bash
#start the server and auto open the blog posts (silences the output)
hugo server -D --disableFastRender > /dev/null &
xdg-open "http://localhost:1313/posts/"

#infinite loop for checking if the user input is valid
valid_input=false
while [[ "$valid_input" != true ]]; do
    #check if the user wants to stop
    read -p "Do you want to publish the blog(s)? (Y/n): " choice

    #kill the blog, and continue on
    if [[ "$choice" == "Y" || "$choice" == "y" ]]; then
        pkill hugo
        valid_input=true
    #kill the blog, and stop the script
    elif [[ "$choice" == "N" || "$choice" == "n" ]]; then
        pkill hugo
        exit 0
    #tell them about the wrong input
    else
        echo "Wrong input, try again."
    fi
done
```

The first two lines starts the server in the background. The reason why I added the `> /dev/null` into the Hugo server command is to make the command output shut up.<sup>[2]</sup> Without that, the Hugo server command will be showed with everything else, causing confusion. The `&` at the end of the command make the entire command run in the background.<sup>[3]</sup> Without it, the script will be waiting for the server to be killed before continuing. While this is normally fine, I would rather have the script get the user's input on what to do with the server.

Then the next section of the code is just all about getting the user's input. The `while` loop is there to prevent the user mistyping their input crashing the script. It will go in a infinite loop of constantly getting the user's input, until they enter in a valid input.   If they say yes, then the loop is broken, and the script continues on. If they say no, then the script will just stop running here. Either option will kill the Hugo server running in the background. 

## Auto Update the Publishing Date
I quickly realised that this one should be done manually, as there is no way I can trust my self to do it properly. This also shouldn't be that hard to do. I just need to get the current date, in the correct format, and insert it into the correct spot. This is what most of the script is all about, so it didn't take me that long to do. To get the current date in the correct format, I would just have to do `date +%Y-%m-%dT00:00:00`. Then, I would just have to use `sed` to update the date via the following method `sed -i "s/^date:.*/date: $publish_date/" index.md`. 

## Auto exit on Error
When I was working on the other two features, I realised that the script will continue to run when there is an error. To prevent that, I add in `set-eo pipefail` at the very top of the script.<sup>[4]</sup> The `set` command allows you to change the shell's options.<sup>[5]</sup> The `-e` makes it exit upon error, `-u` makes the script throw and error on undefined variables.  and `-o pipefail` makes the script exit on command pipe failures.<sup>[4]</sup> 

Now I am not sure why Bash doesn't automatically stops whenever it encounters an error. My guess is that the default shell in Unix (SH), was like that. And since Bash want's 100% compatibility with the SH shell, they have to implement that. 

However, enabling this has caused an error pop up. The thing is, the regex filter to extract the content inside the Obsidian's backlink has also capture the `[[ "$choice" == "Y" || "$choice" == "y" ]]` and `[[ "$choice" == "N" || "$choice" == "n" ]]` from the script earlier. At first, I tried to make Google's Gemini to rewrite that regex statement to not include any leading/trailing white spaces, since bash requires them. But Gemini failed to generate a new one that actually works. So I then decided to make the script checks if there is a `$` inside the `[[]]`. If there is one, then that means that the `[[]]` must have been pulled from a bash conditional instead of a backlink. So it will just ignore that line. This is done via `if [[ $line != *\$* ]];`.
## Auto Exit on Missing Tags/Descriptions
Just like the publication date, I can't trust myself to remember to enter in the tags, description before running the script. To prevent that, I would just have the script stop and yell at me for that. Since the script already extract the tags, I can insert a snippet that will trigger the yelling when the extracted tags is empty.

```bash
#stop the script, and delete the article folder if the tags are missing
if [[ -z "$tags" ]]; then
	echo "Error, the $md_article is missing tags. Add them in and go again. Will be deleting the article folder to make it easier to re-run this script."
	rm -r "$ARTICLE_PATH"
	exit 1
fi
```

For the description, it's the same story as the tags. The only difference is using `grep` and `awk` to extract the description.

```bash
#extract the description out of the file.
desc=$(grep -e "^description:.*" -m 1 index.md | awk -F ":" '{print $2}')

#throws an error when the description is empty, and delete the article.
if [[ -z $desc ]]; then
	echo "Error, the $md_article is missing a description. Add it in and go again. Will be deleting the article folder to make it easier to re-run this script."
	rm -r "$ARTICLE_PATH"
	exit 1
fi
```


## Citations
> [1] Eugene Yarmash. “Shell Script to Open a URL.” _Stack Overflow_, 1 July 2016, stackoverflow.com/questions/38147620/shell-script-to-open-a-url/38147878#38147878. Accessed 27 June 2026.
> [2] GeeksforGeeks. “What Is /Dev/Null in Linux?” _GeeksforGeeks_, 23 June 2022, www.geeksforgeeks.org/linux-unix/what-is-dev-null-in-linux/. Accessed 27 June 2026.
> [3] Panovski, Dejan. “How to Run Linux Commands in Background.” _Linuxize.com_, 1 Nov. 2019, linuxize.com/post/how-to-run-linux-commands-in-background/. Accessed 27 June 2026.
> [4] Rosenfield, Adam, and ADTC. “Automatic Exit from Bash Shell Script on Error.” _Stack Overflow_, 20 May 2010, stackoverflow.com/a/2871034. Accessed 28 June 2026.
> [5] GNU Foundation. “Bash Reference Manual.” _Gnu.org_, 18 May 2025, www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#The-Set-Builtin. Accessed 28 June 2026.
> 