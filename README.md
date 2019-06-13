# CI/CD Pipeline

This is meant to be a simple CI/CD pipeline, suited for home usage. These are the steps to make it work on a Unix based system. It is assumed that the same folders naming is maintained.


1) Initialize Gerrit code review. So uncomment the line ```#    entrypoint: /entrypoint.sh init```, save the file and run 

```
docker-compose up gerrit
```

Although Gerrit tutorial [1] does not specify so, at the first attempt you will probably get this a error

```
Attaching to infra-gerrit
gerrit_1      | fatal: /var/gerrit/etc/mail
gerrit_1      | fatal: Cannot make directory /var/gerrit/etc/mail
infra-gerrit exited with code 128
```

This is an issue related to permissions. An easy, but somehow brutal fix, is to run ```chown -R $(whoami) gerrit```. I didn't came with a better solution until now, but it's in my TODO list. Then execute again the ```compose up``` command and after the gerrit initialization phase exits with code 0, comment the *entrypoint.sh* line and lift up all the containers with

```
docker-compose up -d
```

#### Note on Jenkins

If Jenkins container keeps restarting (check logs ```docker logs infra-jenkins```), it might be a permission problem. You can fix it issuing

```
chown -R 200:200 nexus-data/
```

For more info on Docker permission read articles [2] and [3]




## HANDS ON

After login, under Browse menu, move to Repositories section and create a new repository. Name it and leave default values for dropdowns. Now open 
the command-line and move to **gerrit/git** directory. There will be two default repositories created by Gerrit and there should be the repository 
created earlier.

Only the first time run this command

```shell
rm -rf repository_name.git && git clone --mirror url
```

that will replace the empty content created by Gerrit with the files downloaded from the git repository. Basically what will be stored 
inside **repository_name.git** is the copy of the remote **.git** folder.

Now on your local machine move to the proper working directory and run a `git clone [gerrit_repository/project].git`. This will make a full clone of 
the project as it on Gerrit. Check, by mean of `git remote show origin`, that the *origin* is set to be the Gerrit host. 

**Push code to Gerrit**

After working on the fixes/features, commit the code locally and push it to Gerrit with `git push origin [local branch name]`. If new *tags* were crea
ted
during this stage, push them by means of `git push --tags`. 

**Updating Gerrit** 

From the second time on, is not needed a new *clone --mirror* to update Gerrit, but is enough to run a 

```shell
git remote update
```

this command will update all the `/refs` without overriding the local branches/tags create

### Refs

[1] <a href="https://gerrit.googlesource.com/docker-gerrit/+/refs/tags/v3.0.0">Gerrit Tutorial</a>

[2] <a href="https://medium.com/@mccode/understanding-how-uid-and-gid-work-in-docker-containers-c37a01d01cf">Understanding how uid and gid work in docker containers</a>

[3] <a href="https://docs.docker.com/engine/security/userns-remap/">Isolate containers with a user namespace</a>