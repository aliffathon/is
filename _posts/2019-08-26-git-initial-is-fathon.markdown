---
layout: post
title: jekyll dan github di sub domain fathon
date: 2019-08-26 14:23:00 +0700
categories: jekyll github
tags: jekyll
author: toni
permalink: /jekyll-github-subdomain.html
---

* TOC
{:.toc}

# Setup CURL tanpa password

~~~
nano /home/alpha/.netrc

cat /home/alpha/.netrc
~~~

~~~
machine github.com
login aliffathon
password katasandi

machine api.github.com
login aliffathon
password katasandi

~~~


[gistsfile1](https://gist.github.com/technoweenie/1072829)


## /home/alpha/gem/bin/jekyll new is-fathon/

~~~
Could not load Bundler. Bundle install skipped. 
New jekyll site installed in /home/alpha/Desktop/is-fathon.
~~~

## ls -la

~~~
total 36
drwxrwxr-x 3 alpha alpha 4096 Aug 26 14:05 .
drwxr-xr-x 4 alpha alpha 4096 Aug 26 14:04 ..
-rw-r--r-- 1 alpha alpha  398 Aug 26 14:05 404.html
-rw-r--r-- 1 alpha alpha  539 Aug 26 14:05 about.md
-rw-r--r-- 1 alpha alpha 1652 Aug 26 14:05 _config.yml
-rw-rw-r-- 1 alpha alpha 1119 Aug 26 14:05 Gemfile
-rw-r--r-- 1 alpha alpha   35 Aug 26 14:05 .gitignore
-rw-r--r-- 1 alpha alpha  175 Aug 26 14:05 index.md
drwxrwxr-x 2 alpha alpha 4096 Aug 26 14:05 _posts
~~~

~~~
subl _config.yml

cat .gitignore 

_site
.sass-cache
.jekyll-metadata
~~~

~~~
git status

fatal: Not a git repository (or any of the parent directories): .git
~~~

~~~
git init

Initialized empty Git repository in /home/alpha/Desktop/is-fathon/.git/
~~~

~~~
git status

On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitignore
	404.html
	Gemfile
	_config.yml
	_posts/
	about.md
	index.md

nothing added to commit but untracked files present (use "git add" to track)
~~~

~~~
git add *

git commit -m "blank jekyll"

[master (root-commit) 98c6b15] blank jekyll
 6 files changed, 148 insertions(+)
 create mode 100644 404.html
 create mode 100644 Gemfile
 create mode 100644 _config.yml
 create mode 100644 _posts/2019-08-26-welcome-to-jekyll.markdown
 create mode 100644 about.md
 create mode 100644 index.md
~~~

~~~
git remote add origin https://github.com/aliffathon/is.git

git push -u origin gh-pages

error: src refspec gh-pages does not match any.  
error: failed to push some refs to 'https://github.com/aliffathon/is.git'  

git push -u origin :gh-pages  

error: unable to delete 'gh-pages': remote ref does not exist  
error: failed to push some refs to 'https://github.com/aliffathon/is.git'  
~~~

~~~
git push -u origin master

Counting objects: 9, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (9/9), done.
Writing objects: 100% (9/9), 3.17 KiB | 0 bytes/s, done.
Total 9 (delta 0), reused 0 (delta 0)
To https://github.com/aliffathon/is.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
~~~

~~~
git push -u origin :gh-pages

error: unable to delete 'gh-pages': remote ref does not exist
error: failed to push some refs to 'https://github.com/aliffathon/is.git'
~~~

~~~
git push -u origin HEAD:gh-pages

Total 0 (delta 0), reused 0 (delta 0)
remote: 
remote: Create a pull request for 'gh-pages' on GitHub by visiting:
remote:      https://github.com/aliffathon/is/pull/new/gh-pages
remote: 
To https://github.com/aliffathon/is.git
 * [new branch]      HEAD -> gh-pages
Branch master set up to track remote branch gh-pages from origin.
~~~

~~~
git push -u origin gh-pages

error: src refspec gh-pages does not match any.
error: failed to push some refs to 'https://github.com/aliffathon/is.git'

git remote -v

origin	https://github.com/aliffathon/is.git (fetch)
origin	https://github.com/aliffathon/is.git (push)

git pull

Already up-to-date.
~~~

~~~
git push

fatal: The upstream branch of your current branch does not match
the name of your current branch.  To push to the upstream branch
on the remote, use

    git push origin HEAD:gh-pages

To push to the branch of the same name on the remote, use

    git push origin master
~~~

~~~
git branch -a

* master
  remotes/origin/gh-pages
  remotes/origin/master

git checkout gh-pages

Branch gh-pages set up to track remote branch gh-pages from origin.
Switched to a new branch 'gh-pages'

git branch -a

* gh-pages
  master
  remotes/origin/gh-pages
  remotes/origin/master
~~~

~~~
git branch -D master

Deleted branch master (was 98c6b15).

git branch -a

* gh-pages
  remotes/origin/gh-pages
  remotes/origin/master

git branch -D orign/master

error: branch 'orign/master' not found.

git branch -D -r origin/master

Deleted remote-tracking branch origin/master (was 98c6b15).

git branch -a

* gh-pages
  remotes/origin/gh-pages

git pull

From https://github.com/aliffathon/is
 * [new branch]      master     -> origin/master
Already up-to-date.

git branch -a

* gh-pages
  remotes/origin/gh-pages
  remotes/origin/master

git push origin :gh-pages

To https://github.com/aliffathon/is.git
 ! [remote rejected] gh-pages (refusing to delete the current branch: refs/heads/gh-pages)
error: failed to push some refs to 'https://github.com/aliffathon/is.git'

git push origin :master

To https://github.com/aliffathon/is.git
 - [deleted]         master

git branch -a
* gh-pages
  remotes/origin/gh-pages

git push -u origin gh-pages
~~~

[cloudflare login](https://dash.cloudflare.com/login)

buat records dns ber-tipe CNAME ke username github:

Type 	| Name 			| Content
--------|---------------|------------------------------------------
CNAME	| is 			| aliffathon.github.io

~~~
nano CNAME

cat CNAME

is.fathon.my.id
~~~

~~~
git add CNAME

git commit -m "set CNAME to is.fathon.my.id"

[gh-pages 3070786] set CNAME to is.fathon.my.id
 1 file changed, 1 insertion(+)
 create mode 100644 CNAME
~~~

~~~
git push

Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 287 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/aliffathon/is.git
   98c6b15..3070786  gh-pages -> gh-pages
~~~

~~~
dig is.fathon.my.id

; <<>> DiG 9.10.3-P4-Ubuntu <<>> is.fathon.my.id
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15600
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 8, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1280
;; QUESTION SECTION:
;is.fathon.my.id.		IN	A

;; ANSWER SECTION:
is.fathon.my.id.	300	IN	CNAME	aliffathon.github.io.
aliffathon.github.io.	3600	IN	A	185.199.109.153
aliffathon.github.io.	3600	IN	A	185.199.108.153
aliffathon.github.io.	3600	IN	A	185.199.110.153
aliffathon.github.io.	3600	IN	A	185.199.111.153

;; AUTHORITY SECTION:
github.io.		758	IN	NS	ns-1339.awsdns-39.org.
github.io.		758	IN	NS	ns-1622.awsdns-10.co.uk.
github.io.		758	IN	NS	ns-393.awsdns-49.com.
github.io.		758	IN	NS	ns-692.awsdns-22.net.
github.io.		758	IN	NS	ns1.p16.dynect.net.
github.io.		758	IN	NS	ns2.p16.dynect.net.
github.io.		758	IN	NS	ns3.p16.dynect.net.
github.io.		758	IN	NS	ns4.p16.dynect.net.

;; Query time: 48 msec
;; SERVER: 127.0.1.1#53(127.0.1.1)
;; WHEN: Mon Aug 26 14:41:46 WIB 2019
;; MSG SIZE  rcvd: 365
~~~

~~~
git status

On branch gh-pages
Your branch is up-to-date with 'origin/gh-pages'.
Untracked files:
  (use "git add <file>..." to include in what will be committed)

  .gitignore
  _posts/2019-08-26-git-initial-is-fathon.txt

nothing added to commit but untracked files present (use "git add" to track)
~~~

~~~
git add _posts/2019-08-26-git-initial-is-fathon.txt 

git add .gitignore 

git commit -m "1-1-cara setup github pages untuk domain is-fathon"

[gh-pages 2fe9f3f] 1-1-cara setup github pages untuk domain is-fathon
 2 files changed, 279 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 _posts/2019-08-26-git-initial-is-fathon.txt

git push

Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 2.64 KiB | 0 bytes/s, done.
Total 5 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/aliffathon/is.git
   3070786..2fe9f3f  gh-pages -> gh-pages
~~~
