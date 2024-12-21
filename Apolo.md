## User flag
After connecting to the VPN and cURLing the provided address with `-L`, I found out that the redirect leads to `apolo.htb`, which I added to `/etc/known_hosts`. After opening the website, I found out that there was a link to `ai.apolo.htb`, which after adding to known hosts too, led to Flowise AI, which is an open source low-code tool for developers to build customized LLM orchestration flows.  
![](media/apolo/Pastedimage20241221123512.png)   
![](media/apolo/Pastedimage20241221123535.png)   

 We are then blocked by a login page. After looking around for a bit, I searched for Flowise vulnerabilities on the internet and was led to this: https://www.exploit-db.com/exploits/52001  
![](media/apolo/Pastedimage20241221124019.png)
This was an authentication bypass vulnerability affecting Flowise 1.6.5. The code only applied authentication for calls to  `/api/v1`. So this could be bypassed by making calls to `/api/V1`, for example. 

Using BurpSuite, I sent a request to `/api/V1/credentials` and a MongoDB credential came up:  
![](media/apolo/Pastedimage20241221124127.png)   
`mongodb+srv://lewis:C0mpl3xi3Ty!_W1n3@cluster0.mongodb.net/myDatabase?retryWrites=true&w=majority` 

Trying to connect to the target machine via SSH with user `lewis` and password `C0mpl3xi3Ty!_W1n3` gives us user access to it.  
We can find the user flag in the home directory of `lewis`:
```
lewis@apolo:~$ cat user.txt 
HTB{llm_ex9l01t_4_RC3}
```

## Root flag
Once we have the user access, we can upload a Linux enumeration script, such as [LinEnum](https://github.com/rebootuser/LinEnum), to scan for misconfigurations on the target machine that would allow us to access the root user files contained in `/root/`. We can do that by opening up an HTTP server on our machine that would serve `LinEnum.sh`, and then downloading it on the target machine using `wget`.

Our machine:  
![](media/apolo/Pastedimage20241217212219.png)
IP `0.0.0.0` indicates a network-interface-insensitive server. This is important as we're connected to the target machine through a separate interface since we're using a VPN.

Target machine:  
![](media/apolo/Pastedimage20241217213244.png)  
The IP for `wget` is our IP from the `tun0` interface (can be checked using the `ifconfig` command) - that's the interface of HTB's VPN that's used to connect to their target machines.

Now let's give the script executable permissions and run it. 
![](media/apolo/Pastedimage20241217214704.png)
...

This part of the output tells us that the user `lewis` (our current user) can run the `rclone` program as a superuser **without supplying the superuser password**:  
![](media/apolo/Pastedimage20241217214633.png)  
We do need the `root` user permissions in order to look inside the `/root` directory, so this will come in handy if this program allows to do that somehow.

Let's learn more about it.  
![](media/apolo/Pastedimage20241217215422.png)  
![](media/apolo/Pastedimage20241217220835.png)

Assuming the root flag has `.txt` extension too, let's try to use the `cat` command with the `--include` option to read all the text files inside the `/root` directory (don't forget the `sudo`, otherwise `rclone` will run with the permissions of `lewis`):  
![](media/apolo/Pastedimage20241217220546.png)
Success!
