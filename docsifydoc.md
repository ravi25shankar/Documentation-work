# Untitled

## SETUP DOCSIFY

STEP -  
1- create directory 
2- create Dockerfile

```bash
ravi@client3:~$ mkdir Mydocsify1
ravi@client3:~$ cd Mydocsify1/
ravi@client3:~/Mydocsify1$ touch Dockerfile
ravi@client3:~/Mydocsify1$ vim Dockerfile 
```

DOCKERFILE :  In Dockerfile add details

```bash
FROM node:latest
LABEL description="A demo Dockerfile for build Docsify."
WORKDIR /docs
RUN npm install -g docsify-cli@latest
RUN npm install docsify-print
EXPOSE 3000/tcp
ENTRYPOINT docsify serve . 
```

BUILD IMAGE : with that dockerfile create a image  with a name .  

```bash
ravi@client3:~/Mydocsify1$ podman build -f Dockerfile -t mydoc .     # . represent current directory

# output : 
ravi@client3:~/Mydocsify1$ podman build -f Dockerfile -t mydoc .
STEP 1/7: FROM node:latest
Resolved "node" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/node:latest...
Getting image source signatures
Copying blob 4d81a4d3e863 done  
Copying blob bd36c7bfe5f4 done  
Copying blob 785ef8b9b236 done  
Copying blob 4d207285f6d2 done  
Copying blob 5a6dad8f55ae done  
Copying blob e1c045e015f5 done  
Copying blob 175333a01129 done  
Copying blob 87f55ccb38ca done  
Copying config 725940b282 done  
Writing manifest to image destination
Storing signatures
STEP 2/7: LABEL description="A demo Dockerfile for build Docsify."
--> ad8bf8b6ae3
STEP 3/7: WORKDIR /docs
--> 0e3715efedb
STEP 4/7: RUN npm install -g docsify-cli@latest

added 204 packages in 16s

16 packages are looking for funding
  run `npm fund` for details
npm notice 
npm notice New patch version of npm available! 9.8.0 -> 9.8.1
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v9.8.1>
npm notice Run `npm install -g npm@9.8.1` to update!
npm notice 
--> 0d0c2d5dda6
STEP 5/7: RUN npm install docsify-print

added 1 package in 3s
--> 9efc089bf94
STEP 6/7: EXPOSE 3000/tcp
--> 26f69d9b153
STEP 7/7: ENTRYPOINT docsify serve . 
COMMIT mydoc
--> 850de2e37b4
Successfully tagged localhost/mydoc:latest
850de2e37b41120b2e042f8002d9ed33757d633882a42b1726f9bf5ac1203411
```

Create github account  
- login 
- genrate Token  
-  create repository   

![Screenshot from 2023-08-10 12-10-29.png](Untitled%207e4046fca6de4d6aa328d013f192bab5/Screenshot_from_2023-08-10_12-10-29.png)

Login in github via command line  

```bash
ravi@client3:~/Mydocsify1/Documentation-work$ git config --global user.name "ravi25shankar"
ravi@client3:~/Mydocsify1/Documentation-work$ git config --global user.email ravishankar.ravi@fosteringlinux.com
ravi@client3:~/Mydocsify1/Documentation-work$ gh auth login

? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate GitHub CLI? Paste an authentication token
Tip: you can generate a Personal Access Token here https://github.com/settings/tokens
The minimum required scopes are 'repo', 'read:org', 'workflow'.
? Paste your authentication token: 
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as ravi25shankar
```

Clone that git repository  in created directory (Mydocsify)

```bash
ravi@client3:~/Mydocsify1$ git clone https://github.com/ravi25shankar/Documentation-work.git
Cloning into 'Documentation-work'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), 621 bytes | 103.00 KiB/s, done.
```

Create container  

```bash
ravi@client3:~/Mydocsify1/Documentation-work$ pwd
/home/ravi/Mydocsify1/Documentation-work

ravi@client3:~/Mydocsify1/Documentation-work$ podman run -d --name docsify1 -p 3000:3000 -v /home/ravi/Mydocsify1/Documentation-work:/docs mydoc:latest 
cf648ec07024b470d69d95247b97ad52e420d6eba818975d64b59e0767202f5d
ravi@client3:~/Mydocsify1/Documentation-work$ podman ps
CONTAINER ID  IMAGE                       COMMAND               CREATED        STATUS            PORTS                                           NAMES
30157a5fa65d  quay.io/minio/minio:latest  server /data --co...  2 days ago     Up 2 days ago     0.0.0.0:9000->9000/tcp, 0.0.0.0:9090->9090/tcp  agitated_elgamal
cf648ec07024  localhost/mydoc:latest                            4 seconds ago  Up 4 seconds ago  0.0.0.0:3000->3000/tcp                          docsify1
```

Create index.html file 

```bash
ravi@client3:~/Mydocsify1$ cd Documentation-work/
ravi@client3:~/Mydocsify1/Documentation-work$ touch index.html
ravi@client3:~/Mydocsify1/Documentation-work$ vim index.html 
```

```html

<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <meta charset="UTF-8" />
    <link
      rel="stylesheet"
      href="//cdn.jsdelivr.net/npm/docsify@4/themes/vue.css"
    />
  </head>
  <body>
    <div id="app"></div>
    <script>
      window.$docsify = {
        //...
      };
    </script>
    <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  </body>
</html>

```

Create new file in md format

```bash
avi@client3:~/Mydocsify1/Documentation-work$ touch docsifydoc.md
ravi@client3:~/Mydocsify1/Documentation-work$ ls
docsifydoc.md  index.html  README.md  testdoc.md
	ravi@client3:~/Mydocsify1/Documentation-work$ vim docsifydoc.md
```

Add that file in git  
commit that file with comment 
push that file .

```bash

ravi@client3:~/Mydocsify1/Documentation-work$ git add docsifydoc.md 
ravi@client3:~/Mydocsify1/Documentation-work$ git commit -m "Added docsfiy md file for testing"
[main a9f99d3] Added docsfiy md file for testing
 1 file changed, 16 insertions(+)
 create mode 100644 docsifydoc.md
ravi@client3:~/Mydocsify1/Documentation-work$ git push origin main
Username for 'https://github.com': ravi25shankar
Password for 'https://ravi25shankar@github.com': 
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 2 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 730 bytes | 365.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/ravi25shankar/Documentation-work.git
   99d9855..a9f99d3  main -> main
```

Check on browser
```bash
  
localhost:3000/#/docsifydoc.md 

```
```bash

ip:3000/#/docsifydoc.md

```
