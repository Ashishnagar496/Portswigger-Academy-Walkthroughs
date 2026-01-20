# Information Disclosure

### Mechanism :

- Occurs when the application fails to properly protect the sensitive information and giving users access to information that should not be available publicly .

### Preventions :

- regularly audit public repos to confirm that no sensitive information is included in them .
- tools (secret bridge ) can help to monitor code for secrets .
- handle exceptions by returning an error page not an technical page that can leak application version and other details .

### Hunting For The Information Disclosure :

- start with finding the versions and other information using recon techniques .

### STEP 1 : Attempt a path traversal

*path traversal attacks are used to access files outside the web application’s root folder .*

- Discussed about two types of URLs :

      relative URLs : contains only a part of the full URL .

      absolute URLs :  entire address and URL .

for example if  a web application fetches images as :

```python
https://example.com/image?url=/images/1.png
```

here url parameter contains the relative url , we can perform path traversal in the url parameter to navigate outside the image folder .

```python
https://example.com/image?url=/images/../../../../etc/shadow
```

/etc/shadow is a file in Linux systems which stores the system’s user accounts and their encrypted passwords .

- this requires some hit and trials .
- if input is validated then check by url encoding (../ = %2e%2e%2f ) or we can also use double url encoding and partial encoding like (..%2f)

## STEP 2 : Search the wayback machine

- it is the online archive that store the websites previous versions at a particular time ,we can use this to find hidden and old endpoints of the target without crawling the  target .
- 

```jsx
for the domain - example.com search 
https://web.archive.org/web/*/eample.com/* this will return the domain example.com
related URLs and then we can search for admin pages , config file with extensions (.conf) 
and js file and php file (.js and .php ) to try any old hardcoded credential still in use .
```

 

# STEP 3 : Search Paste Dump Sites

- pastedump sites like pastebin and github gists
- these let users to share the text documents via a direct link rather than via email or services like google docs .
- pastehunter and pastebin scrapper can be used for automating the task of finding sensitive informations shared on the paste bin because by default pastebin has files set to public .
- use the pastebin scraper by downloading in a local directory and use the command  (./scrape.sh -g keyword ) this will return an id for the searches which can be used to search as ( [pastebin.com/id](http://pastebin.com/id) ).

## STEP 4 : Reconstruct source code from an exposed .git directory

- sometimes the developers uses git to version control a project’s source code and git will projects version control infromation including the commit history of project files .
- attacker can obtain the source code or harcoded apis and other sensitive data via secret scanning tools like trufflehog or gitleaks .

## Checking :

- check (example.com/ .git )
- if 404 error then it means git repo is not public
- if no error then we will list files using wget command with recurssive mode (-r ) to download all files within that specific repo directory . (wget -r [example.com/.git](http://example.com/.git) )
- then use ls .git command to list the downloaded files .
- if the files listing is not allowed and directory files are not shown then we can use (curl [https://example.com](https://example.com)/.git/config)
- if the above link is accessible then we can download the files .
- a git repo contains main files like …
- 

```jsx
COMMIT_EDITMSG
HEAD
branches
config
description 
hooks 
index 
info 
logs
objects
refs
```

- these files are important for us to reconstruct the source code .
- objects directory is used to store git objects . it also contains two subdirectories each has two characters names corresponding to the first two characters of the SHA1 hash of the git objects stored in it .for example : a  git object with a hash of :
- 

```jsx
0a008934564564r3r532453532532fgfhhsd
then it will be stored in the filename  008934564564r3r532453532532fgfhhsd
and this file will be located at /.git/objects/0a.
```

- git stores different types of objects in .git/objects commits ,trees,blobs and annotated tags we can determine an object type using the command :
- 

```jsx
git cat-file -t OBJECT-HASH
```

```jsx
commit - these objects store information such as the
commit's tree object hash parent commit ,author ,
commiter ,date and message of the commit 

tree objects contain the directory listings for commits

blob objects contain copies of files that were committed 
(read actual source code ) 
tag objects contains information about the tagged - objects 
and their associated tag names .
we can use the command git cat-file -p OBJECT-HASH 
to display the file associated with a git object.

config file is the git configuration file for the project.
aand the /HEAD file contains a referece to the current branch .
cat .git/HEAD

```

# Step 5 : find information in public files

- we can also try to find information leaks in HTML and Javascript files ( richer )

## STEPS INVOLVED :

1. browse the web application ,and take note where application displays or uses personal information .
2. then right click those pages and view the page source .
3. read for comments in the HTML texts and navigate through all the links in the HTML texts to find more HTML and javascript files application is using .
4. use keywords like password ,api to find sensitive information leaks .
5. to locate js files on a web application we can also use the tools like linkfinder or jsfinder and other js finders .

## Escalating the attacks :

after finding any information leaks always validate and confirm whether they are valid in the present or not .

also try to escalate the attacks using other techniques like using the finded admin password to leak the private informations .

also find the CVE available for an software version or not .