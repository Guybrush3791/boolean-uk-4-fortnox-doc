# Troubleshoot VSC problems on Windows WSL

## Prologue - When is this guide useful?

This guide assumes that you are using WSL on your Windows 11 machine and have successfully installed the Ubuntu Distro as a subsystem for your Windows 11 machine. If you have not done that, you can follow [this](https://www.youtube.com/watch?v=zZf4YH4WiZo) easy YouTube video.

**NOTE, VERY IMPORTANT:** From now on, you have to get used to handling two separate file systems. One belonging to WSL and one belonging to Windows 11. By default, when you open an ubuntu terminal or open a normal windows terminal window and write `wsl` you will most likely be presented with this:

![alt text](image-11.png)

Similarly when we open a "real" WSL terminal:

![alt text](image-12.png)

Notice how the file path starts with `/mnt/c/`. This is the root of a lot of confusion on how WSL behaves. This is actually not the "real" `C:` drive on your windows machine. It is only a reference or shortcut that WSL uses to refer to your actual `C:` drive. Building real java projects here will cause problems down the line, especially when using build tools like Gradle, SpringBoot or using virtual environments like devbox or docker to build applications and services because the file path is not the "real" file path. please refer to the bottom of this document for a more in depth explanation. What you have to do is go completely out of that directory and try to find `/home`. This is how you can do that simply following these commands:

![alt text](image-13.png)

**THIS IS WHERE YOU SHOULD ALWAYS WORK:** all of your applications, projects, exercises, challenges and so on should be built from the `/home/(username)` directory. You can nest in to `/home/(username)` however long you want but the "starting directory" has to be `/home/(username)`. this is the "real" Linux filesystem. 

## Continuation - How do i start building applications

This guide is useful to all issues pertaining to when your VSCode editor does not recognize errors or problems in your code due to you running a virtual environment via WSL and devbox and spinning up services on said environments.

For example, when opening VSCode "normally", a person might do something like this:


1. Spin up services `cd Path/to/project && Devbox services up`
2. enter the Devbox virtual environment ``Devbox shell`
3. Opening up VSCode `code .`

![alt text](image1.png)

## Symptoms

When opening up VSCode using this "normal" method and you start coding. you will notice quite a lot of problems. These are the most common ones that i have found among my colleagues:

1. Java Projects extension and Gradle not building.

This is probably the most common symptom. You have opened a java project using `code .` in VSC and the Java Projects will not load properly and Gradle will not start. you can see this at the bottom-left of your VSC window.

![alt text](__assets/images/wsl-pre-require/image.png)

Keep in mind that Gradle is broken _but_ the service is still running and working just fine. This of course assumes that the code you have in your project _actually_ works and is error-free.

![alt text](image-1.png)

In this example, you see that the DB is running, the service is up and DBeaver showing that the DB is properly connected even though Gradle is showing a `Gradle: Build Error` inside VSC.

2. The other very common symptom is that your LSP (the thing making red squiggly lines under your code lines) will be broken. This shows by either not generating any red lines at all or generating red lines everywhere. This will result in your coding experience being very annoying because auto-suggestions for importing packages, annotations, types will not show up as you code. This means that coding in VSC basically is like coding on a NotePad application. Not so enjoyable. 

![alt text](image-3.png)

In this example, we should have a `import org.springframework.stereotype.Service` statement in order to make this code compile properly. However, in normal circumstances, we should just be able to write the `@Service` annotation and this should appear as a suggestion for the given imported library. But now, the LSP has no context of the java project so it can not give good suggestions either.

Another example of this is if we remove the `@Service` annotation, we should get a yellow squiggly line saying something like `The import org.springframework.stereotype.Service is never used`. This is the LSP not properly recognizing falacies/problems/errors in the code _before_ we try compiling.

## Solution

1. Press on the `><` button on the bottom-left corner of your VSC window, if you hover your mouse over it, it should say `Open a remote window`. This feature in VSC allows you to pretty much open a new instance of VSC in a completely custom environment outside of VSC vanilla environment which is somewhere on Windows 11 standard `C:` drive. In our case, the custom environment is WSL, a virtual subsystem which sits outside of windows own filesystem.

2. VSC will ask you, which environment you want the new VSC instance to open in. Since we are using WSL you want to pick the `Connect to WSL` option.

![alt text](image-5.png)

3. If its the first time doing this on your computer, it might take a long time (several minutes) for the VSC window to properly load in. This is normal, we are pretty much installing VSC from scratch again but from within our WSL environment. When it is all done, look at the `><` button on the bottom-left again. it should now say `>< WSL: Ubuntu`. And if you open up the Explorer side menu it should now say `Connected to remote`. and if you press `Open Folder` it should now show you the WSL filesystem instead of the vanilla Windows 11 file explorer.

![alt text](image-6.png)

4. If you use that same side menu and go to the Extensions menu. you will now notice that all of your extensions are missing. This is a good sign as this means that you have properly created a fresh installation of VSC. For this guide you only need to install `Project Manager for Java` and `Spring Boot Extension Pack`. Those extension have sub-extensions that will be installed automatically so it might take some time.

![alt text](image-7.png)

5. Open up the folder you want to work in. You can use the VSC prompt for this. in my case it was `Day13/jpa-api` but it can be any folder that is a Java project. The VSC window will now probably re-load once again but it should be much faster than the previous step as we are now just opening a folder.

6. When the Java project is open, you should now finally see the `Java projects` tab appear under the explorer. Notice that now when you open the same file you will now see it behaving more normally. You should see `Java: Ready` on the bottom-left and if you open a valid Java file you will start seeing the correct LSP (red and yellow squiggly lines)

![alt text](image-8.png)

**NOTE: If step 6 does not work and you are stuck in an infinite loading loop and getting a lot of error messages and popups from VSCode. try restarting your machine. this procedure involves saving new environment variables on the windows machine and can cause it to get confused about having two root paths pointing to similar VSC installations. When the computer is restarted, open a new ubuntu terminal and cd in to that directory and once again write `code .` it will either ask you to connect to WSL (yes, do that) or it will just work automatically**

7. Now, everytime you open a new ubuntu terminal on your windows machine and write `code .` you should always see `>< WSL: Ubuntu` and the `Java: Ready` / `Java Projects` tab should work automatically.


## Why does this solution work?

This problem happens because, as standard, VSC is installed on windows' own file system somewhere on the `C:` drive. While WSL, as standard, actually decouples from `C:` and creates it own file system. see proof of this in the next image.

![alt text](image-9.png)

This means that everything we do on WSLs file system does not have the correct context of what is done in `C:`. every time we install a new program like firefox, VSC, IntelliJ, Docker, Teams etc. WSL does not know. If you have ever had a machine that Dual Boots Windows and Linux with GRUB, this is probably easier to understand. WSL is not quite like DualBooting as that would be like having two separate OS partitions on your harddrive that has to be loaded in at boot. WSL instead operates on the Windows layer as a subsystem (hence the name, Windows Subsystems for Linux). This is also why our settings and extensions will be gone on VSC, because we have pretty much loaded windows at a location on our computer that it has not loaded in before.

# Author
Thanks to *Kemal Cikota* for the help, github account:
https://github.com/Kemalcikota113/java-tdd-oop-bank-challenge