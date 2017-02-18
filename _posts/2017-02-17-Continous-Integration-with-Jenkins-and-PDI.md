Continous Integration with Jenkins and PDI

# Download and Install Jenkins

Sample installation using Tomcat:

```
cd ~/apps
wget http://mirrors.ukfast.co.uk/sites/ftp.apache.org/tomcat/tomcat-9/v9.0.0.M17/bin/apache-tomcat-9.0.0.M17.zip
unzip apache-tomcat-9.0.0.M17.zip
rm apache-tomcat-9.0.0.M17.zip
chmod 700 apache-tomcat-9.0.0.M17/bin/*.sh
cd apache-tomcat-9.0.0.M17/webapps
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war

cd ../bin
sh ./startup.sh
```

Go to `localhost:8080/jenkins` and enter the password (which can be found under `~/.jenkins/secrets/initialAdminPassword`.

After logging in you will be presented with an option to install **plugins**. At this stage you want to install the [Git plugin](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin).


# Schedule PDI Job


Sources:
 
- [Pentaho Data Integration scheduling with Jenkins](http://opendevelopmentnotes.blogspot.co.uk/2014/09/pentaho-data-integration-scheduling.html)


Once logged on to the Jenkins web interface do the following:

1. Click on **New Item**.
2. Provide a name for the item and choose **Freestyle project**.
3. Under **Build Triggers** tick **Build periodically**. Insert a **Crontab** like schedule into the **Schedule** box (e.g. `* * * * *` to run the job every minute).
4. In the **Build** section click **Add build step** and then choose **Execute shell**. Insert the shell command into the text box. This is the same command as you would normally use to run your jobs or transformations. Example:

  ```
  /home/dsteiner/apps/pentaho-ee/design-tools/data-integration/pan.sh \
  -file=/home/dsteiner/git/diethardsteiner.github.io/sample-files/pdi/test.ktr
  ```

5. Click **Save**.
6. Wait a bit and soon you should see the build running (on the **Build History** section). Click on the build number and then on **Console Output** to see the log.


# On Commit Build

Sources:

- [Automatically triggering a Jenkins build on Git commit](http://www.andyfrench.info/2015/03/automatically-triggering-jenkins-build.html)


In the context of PDI **build** means running a PDI job or transformation.

We will use a [git hook](https://git-scm.com/book/be/v2/Customizing-Git-Git-Hooks) to notify **Jenkins** about a new commit. 

A quick intro to **Git Hooks**:

- Git Hooks let you define action on specific events (e.g. `post-commit`).
- There are client and server-side versions of git hooks. Git hooks are stored in `<git-root>/.git/hooks` and are basically shell scripts (or any other scripting language). Filenames do not have an extension.
- Git hooks do not get version controlled and are not pushed to the server. Keep this in mind when creating client side hooks. They are local to each repo.
- You can use a symlink and store the shell script within the standard repo path. As an alternative, Git also provides a [Template Directory](https://git-scm.com/docs/git-init#_template_directory) mechanism that makes it easier to install hooks automatically. All of the files and directories contained in this template directory are copied into the .git directory every time you use git init or git clone.
- Git hooks have to be **executable**.


Create a file called `post-commit` (no extension) in the `.git/hooks` directory of your git project. Add the following to the file (adjust to your requirements):

```bash
#!/bin/sh
curl http://localhost:8080/git/notifyCommit?url=file:///home/dsteiner/git/diethardsteiner.github.io
```

Finally make the file executable:

```bash
$ chmod 700 .git/hooks/post-commit
```

The webservice endpoint `notifyCommit` is exposted by the **Git plugin** we installed earlier on. For this endpoint to work you have to **enable Poll SCM** in your **Jenkins build configuration**. **Important**: Don't specify a schedule!

