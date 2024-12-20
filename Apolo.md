## Apolo

This was a fullpwn challenge classed as "very easy".

After connecting to the VPN and cURLing the provided address with `-L`, I found out that the redirect leads to `apolo.htb`, which I added to `/etc/known_hosts`. After opening the website, I found out that there was a link to `ai.apolo.htb`, which after adding to known hosts too, led to Flowise AI, which is an open source low-code tool for developers to build customized LLM orchestration flows. 

 We are then blocked by a login page. After looking around for a bit, I searched for Flowise vulnerabilities on the internet and was led to this: https://www.exploit-db.com/exploits/52001

This was an authentication bypass vulnerability affecting Flowise 1.6.5. The code only applied authentication for calls to  `/api/v1`. So this could be bypassed by making calls to `/api/V1`, for example. 

Using BurpSuite, I sent a request to `/api/V1/credentials` and a MongoDB credential came up: 

`mongodb+srv://lewis:C0mpl3xi3Ty!_W1n3@cluster0.mongodb.net/myDatabase?retryWrites=true&w=majority` 
