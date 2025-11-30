# Empire : Lupin one  CTF writeup

## Tools used

- Netdiscover
- Nmap
- Ffuf
- Gobuster
- dcode for cipher identification and decryption
- 


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
    
## Step 3

Now that we know that there exists a web server on the machine we can use the URL `http://10.38.1.117` to go to the web page.

<img width="950" height="885" alt="image" src="https://github.com/user-attachments/assets/fd4332ea-c056-414b-8579-111badeb3e85" />

We find nothing much here.

## Step 4

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

## Step 5

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

## Step 6

We need to know the encoding format that was used and the tools that you will use to decrypt the text.
We will use the Cipher Identifier tool on the dcode website.

Now to access the internet and go to the dcode website you need to save the VM state and change the VM network settings to bridged adapter.

Go to the URL https://www.dcode.fr/cipher-identifier and paste the message.

After the analysis the tools points out that the most probable format is base 58.

<img width="949" height="760" alt="image" src="https://github.com/user-attachments/assets/7eb2220d-e63e-485c-8bfc-de75726f5dce" />

