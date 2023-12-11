Question 1 Answer(s):

First Bash Script (saved as `users.sh` in /root/scripts): 

~~~
#!/bin/bash
awk -F':' '{ print $1 ":" $6 }' /etc/passwd > users.txt
~~~

Script Breakdown: 

- `#!/bin/bash`: This line tells the system to use Bash to execute the script.
- `awk -F':' '{ print $1 ":" $6 }' /etc/passwd`: This line invokes the `awk` programming language, which is used for processing and analyzing text files. It uses the `/etc/passwd` path as this is the system file that contains information about user accounts on the system in Linux based environments. 
	- It uses the `-F` option which sets the input field separator, in this case, the semicolon `:`. 
	- `'{ print $1 ":" $6 }'` is the actual AWK command enclosed in single quotes. 
		- `print`: This is an `awk` command to output text.
		- `$1` and `$6`: These are field variables in `awk`. `$1` refers to the first field in a record (line), and `$6` refers to the sixth field. In the context of the `/etc/passwd` file, `$1` is the username, and `$6` is the user's home directory.
		- `":"`: This is a literal string consisting of a colon. It's used here to separate the username and home directory in the output.
		- The entire `print $1 ":" $6` command instructs `awk` to print the first and sixth fields of each line, separated by a colon.
		- ` > users.txt`: the output of this script will be saved to the current directory as a txt file names users. 

Second Bash Script (saved as `checksum.sh` in /root/scripts): 

~~~
#!/bin/bash

output_file="users.txt"
current_users_file="/var/log/current_users"
user_changes_file="/var/log/user_changes"

md5_now=$(md5sum $output_file | awk '{print $1}')

if [ ! -f "$current_users_file" ]; then
    echo "$md5_now" > "$current_users_file"
    echo "Initial MD5 checksum stored."
else

    md5_prev=$(cat $current_users_file)

    if [ "$md5_now" != "$md5_prev" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') changes occurred" >> $user_changes_file
        echo "$md5_now" > "$current_users_file"
        echo "Changes detected and logged."
    else
        echo "No changes detected."
    fi
fi
~~~

Script Breakdown:  

- `#!/bin/bash`: This line tells the system to use Bash to execute the script.

~~~
output_file="users.txt"
current_users_file="/var/log/current_users"
user_changes_file="/var/log/user_changes"
~~~

- These three lines define the variables for the paths of the files we will be needing to meet the requirements of the second script. 
	- `output_file`: The file with usernames and home directories (`users.txt`).
	- `current_users_file`: A file in `/var/log/` to store the current MD5 checksum.
	- `user_changes_file`: A file in `/var/log/` to log any changes.

- `md5_now=$(md5sum $output_file | awk '{print $1}')` : This command calculates the MD5 checksum of the `users.txt` file. It uses `md5sum` to generate the checksum and `awk` to extract just the checksum part (excluding the filename).

~~~
if [ ! -f "$current_users_file" ]; then echo "$md5_now" > "$current_users_file" echo "Initial MD5 checksum stored." else ... fi
~~~

- This block checks if the `current_users_file` exists. If it doesn't, the script creates it and writes the current MD5 checksum to it. 
- `md5_prev=$(cat $current_users_file)`: Inside the first else block the scripts will check to see if there is already a "`current_users_file`". It will read the previous checksum of this file to determine if true or not. 
- `if [ "$md5_now" != "$md5_prev" ]; then ... else echo "No changes detected." fi`: Here, the script compares the current checksum (`md5_now`) with the previous one (`md5_prev`). If they are different, it implies that the `users.txt` file has changed.

~~~ 
echo "$(date '+%Y-%m-%d %H:%M:%S') changes occurred" >> $user_changes_file 
echo "$md5_now" > "$current_users_file" 
echo "Changes detected and logged." 
~~~

- If a change is detected, the script logs the date and time of the change to `user_changes_file`. Then it updates `current_users_file` with the new checksum.
- While not required, the echo messages have been added to display helpful information on the terminal. Example below when ran on local machine.  
	- ![[Pasted image 20231211124103.png]]

*Note: These scripts will need to be made executable before running it if not using the bash command. If using the bash command, it will not require the scripts to be executable. To make the scripts executable run the following command in the terminal `chmod +x "script.sh"`. The name script.sh will be replaced by the name you provided when creating the file.

Crontab entry: 
- `0 * * * * /root/scripts/users.sh && /root/scripts/checksum.sh`
	- This entry schedules both `users.sh` and `checksum.sh` scripts to run every hour on the hour

Question 2 Answer: 
### Possible Causes of Slowness

1. **Resource Utilization Issues**: The server's hardware resources might be over-utilized. With only 8GB RAM and 2 CPU cores, it's possible that the web application, web server, and database are competing for limited resources, especially during peak usage times.
2. **Database Performance**: Since the data is stored in a relational database, slow queries or an unoptimized database could be a bottleneck. This could be due to factors like large datasets, lack of proper indexing, or inefficient queries.
3. **Application Code Inefficiency**: The application code itself could be inefficient, particularly if it's not well-optimized for performance. This might include heavy computations, large data processing, or inefficient use of the MVC framework.
4. **Network Issues**: Although all components are on a single machine, network issues can still play a role, such as DNS resolution problems or issues with the internal loopback network interface.
5. **Web Server Configuration**: Misconfigurations or suboptimal settings in the web server could lead to performance issues. This includes things like improper handling of static assets or a poorly tuned number of worker processes.
6. **External Dependencies**: If the application relies on external services or APIs, delays in these services can manifest as slowness in the web application.
### Troubleshooting Steps

1. **Monitor Resource Usage**:
    - Use tools like `top`, `htop`, or `vmstat` to monitor CPU and memory usage in real time.
    - Check for any processes that are consuming an unusually high amount of resources.
2. **Analyze Database Performance**:
    - Review the database's performance metrics, such as query execution times.
    - Look for slow queries and examine if they can be optimized
    - Ensure that the database is not running out of memory
3. **Profiling Application Code**:
    - Utilize profiling tools specific to the MVC framework being used to identify slow parts of the application.
    - Check for query problems, inefficient loops, or heavy computations in the application code.
4. **Inspect Web Server Logs and Configuration**:
    - Review web server logs for any errors or warnings.
    - Analyze the configuration for any potential issues like misconfigurations in the services. 
5. **Network Diagnostics**:
    - Use tools like `ping` and `traceroute` to check for network latency, even though the services are on the same machine, there could be an issue between them as well as issues with external connections.
6. **External Service Dependencies**:
    - Verify the performance and availability of any external services, like APIs, the application depends on.
7. **Review Application and System Logs**:
    - Check system logs for any hardware-related errors.
    - Review application logs for any runtime errors or warnings that could indicate issues.
8. **Conduct Load Testing**:
    - Use load testing tools to simulate high traffic and observe how the system behaves.
9. **User-Specific Issues**:
    - Verify if the issue is isolated to specific user or browser, which might indicate client-side issues.
10. **Updating and Patching**: 
	- Ensure all components (web server, database, application framework) are up-to-date with the latest patches.

Question 3 Answer: 

From studying the graph given, the Git commands below could have resulted in this commit graph: 

1. The first commit was made `git commit -m "first commit"`
2. Changes where made to the code and a second commit was made `git commit -m "second commit"`
3. A branch was made after the second commit `git checkout -b "feature-branch"`
4. There was a third commit on this new branch `git commit -m "third commit"`
5. After the third commit, context switch to the main branch `git checkout main`
6. Feature-branch and Main branch merged `git merge feature-branch -m "Merge"`
7. Lastly, there was a fourth commit `git commit -m "fourth commit"`

Question 4 Answer: 

Best Practices for Feature Development in a Live Project

Hello, developers!

Today we will discuss how to implement a feature update or change to your live project while avoiding the risk of something going terribly wrong and not being able to fix it. Remember, the project is already in the hands of users, so maintaining the integrity of the code that is live is of upmost importance. 

For this best practice, we will be taking advantage of Git. and thankfully, you have already been diligent and used Git for version control. Your `master branch`, or "main" branch if you want to refer to it as that, is the current, stable version of your project. This branch must remain protected and we must avoid making any missteps that may not be easily reversible. 

That being said, the first thing we will need to do is to create a `feature branch` from the `master branch`.  This `feature branch` will be your development workspace. Here you will be able to conduct changes to your project without affecting the `master branch`. 

To create this new `feature branch` you will need to execute the following command in the root directory of your project: 

`git checkout -b feature-branch`

This command will create a new branch with the name feature branch. To change the name to something else, simply replace the `feature-branch` with any name of your choice. The `-b` option will automatically check out your new branch, meaning, you will be able to start working on your new branch immediately without it affecting the `master branch`. 

This new branch has a couple of benefits, some of which are that: 
1. It separates any new code changes to your project from the stable version that is live. This allows for a cleaner and more focused development process since you no longer risk breaking your live project. 
2. It offers flexibility in the development process. If you make a change and the changes don't work as you intended them too, you can simply discard the branch without affecting the `master branch`. 
3. It allows for testing to be done easier and while avoiding any possible downtime. You are able to test thoroughly without worrying about messing up all the hard work that has already been done up to this point. 

Once you have created, reviewed, and tested your new feature, its time to merge this new branch 
with your `master branch`. 

The first step in merging your branches will be to switch over to your `master branch`. To do so, you will run the following command: 

`git checkout master`

Once you are in the `master branch`, you can integrate your `feature branch` with the following command: 

`git merge feature-branch -m "notes"` 

*The -m option lets you pass on a message. Feel free to replace `notes` with whatever message you would like to include that will help you keep track of this change.

Congratulations! Your `master branch` is now up to date with the newly added features. However, nothing in life worth doing is easy, so if you encounter any errors after the merge, they will need to be sorted out. 

Thank you for reading this weeks blog post, we hope you join us again next week. 

Happy developing!

Question 5 Answer: 

A recent technical blog that I have been reading is https://learnk8s.io/blog. This blog contains in-depth technical articles with a focus on mastering the fundamentals of Kubernetes. As someone who is currently studying to take the CKA (Certified Kubernetes Administrator) exam, I have been focused on consuming as much as I can about Kubernetes. My experience with Kubernetes is from my most recent role as a DevOps engineer. I was responsible for maintaining live environments running in a Kubernetes cluster(s). It was my first experience working with this tool and I really enjoyed using it and learning about it on a day to day hands-on basis. 

I am no expert in the tool, so I took it upon myself to learn as much as I can from outside resources that are available online. I landed on learnk8s from a post someone made on a social media platform and I am very content with everything they offer. The knowledge in their blogs and handbooks are very useful. 

To add more practice and studying I also signed up to KodeKloud (again, someone mentioned it on a social media post). This platform gives you hands on practice using live environments meant for testing. They offer a space where you can troubleshoot issues with Kubernetes environments and I believe that the best learning experiences come from situations where things break and its your job to fix them. 