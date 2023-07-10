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
touch authorized_keys\
imagehere\
\
With that taken care of, back on your attacker box, cat you id_rsa.pub (the public key and copy the entire output.\
image\
\
All that is left to do is to echo the output of your id_rsa.pub into the authorized_keys file on the target box. I always like to double-check key was formatted properly by taking a look at the authorized_keys file. Alternatively to echoing the contents of your public key to the authorized_keys file, if you have ssh access to the target box you can utilize the text editor of your choice and copy + paste the contents.\
imagehere\
\
Now to utilize the backdoor all you have to do is ssh to the target box and no matter what they change their password to you will always have key access to the host.\
\
Note: make sure to use -i [private key file] in your ssh command to specify you want to make use of key authentication.\
imagehere

# //------------------------------------
