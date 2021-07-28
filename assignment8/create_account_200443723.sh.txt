#!/bin/bash
# Please fill out the following information when submitting this assignment.
# Firstname: Yu-Hsuan
# Last Name: Huang
# Date: 2021/07/27
# Student ID: 200443723
echo "This script will create a new user account on this system. The script will also attempt to generate the relevant RSA encryption files and configure the account for SSH login. "
# Step 1 : Get Information ################################################################################
echo "Please provide the following information for the new user. "
echo "First Name: "
read FirstName
echo "Lastname: "
read LastName
echo "Email address: "
read email
echo "username: "
read username



# Step 2 : adduser #######################################################################################
#     < insert adduse command here > <20 points>
# Did adduser work? what to do if there is an error?
# Gives exit status 1 if adduser failed or generated error
#	< insert If statement here to handle adduser errors >

adduser --disabled-password --gecos "$FirstName $LastName, $email" $username
if [ $? -eq 0 ]
	then
		echo "Linux User account for ${FirstName} ${LastName} was created successfully."
		# echo "Account created"
else
	echo '!!!!!! User account was not added. !!!!!!'
	# echo "Accound was not added"
	exit 1
fi



# Step 3 : Generate Random Password ######################################################################
# Generate a strong, 16 characters, random password
#	< Insert random number and password generation here >
Password=`< /dev/urandom tr -dc _A-Z-a-z-0-9\%\&\*\$\#\@\! | head -c${1:-16};echo;`



# Step 4 : Generate a message for user ###################################################################
# Create a directory to store the information we will need to provide to the user later
mkdir $username
# Generate a message for the user and store in account_info.txt
#	< Insert commands to generate message to user here >
echo "Hello $FirstName" > ./$username/account_info.txt
echo "You have been granted a user account on our system. " >> ./$username/account_info.txt
echo "Your User Name is: " $username  >> ./$username/account_info.txt
echo "Your randomly generated Passphrase is: " $Password >> ./$username/account_info.txt
echo "You are being provided with your own RSA private key. The server will be configured with your public key in order to allow you access to server via SSH. " >> ./$username/account_info.txt
echo "Make sure to protect and guard your private key and passphrase at all costs! " >> ./$username/account_info.txt



# Step 5 : Generate RSA keys, and check for errors #######################################################
#Generate the RSA key, store RSA key pairs in the directory
#    < insert your ssh-keygen command here > <30 points>
chmod 655 ./$username
ssh-keygen -q -N "MyPassphrase" -f ./$username/id_rsa

# Did ssh-keygen work? if failed, delete the user account and home directory, also give exit Status 2
#	<insert IF statement to handle ssh-keygen errors >
if [ $? -eq 0 ]
   then
        echo "RSA key was generated successfully"
else
    echo '!!!!!! RSA Key was not created, ssh‐keygen exit status '$?' !!!!!!'
    echo "Undoing the changes made to system so far"
    deluser $username && rm -r /home/$username
    exit 2
fi



# Step 6 : Move the public key to appropriate location ###################################################
# If the .ssh directory does not exists in user's home directory, then create it and make sure it is owned by the user
#	< insert IF statement here >  <30 points>
#    < don't forget to set permissions >
if [ ! -d "/home/$username/.ssh" ]
	then
		mkdir /home/$username/.ssh
		chmod 700 /home/$username/.ssh
		chown $username:$username /home/$username/.ssh
fi

# move the public key to user's .ssh/authorized_keys file, and change ownership to the user
#	< insert your MV command here > <20 points>
#	< Are you done? did you set permissions? >
mv ./$username/id_rsa.pub /home/$username/.ssh/authorized_keys
chown $username:$username /home/$username/.ssh/authorized_keys



echo "The authorized_keys file was placed in user's folder. "
# end... does script work as-it without errors (was testing done?) <50 points>
