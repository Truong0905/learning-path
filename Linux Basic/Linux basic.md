# I. BASIC ComMandS
## 1.  File System Structure and Description
![[Pasted image 20241222111106.png]]

## 2. Linux File or Directory Properties
![[Pasted image 20241222111416.png]]

![[Pasted image 20241222111440.png]]

## 3. What is root?
![[Pasted image 20241222111603.png]]

## 4. Changing **Password**
![[Pasted image 20241222111718.png]]

## 5. Creating Files and Directories (touch, cp, vi, mkdir)
![[Pasted image 20241222111828.png]]

## 6. Copying directories
![[Pasted image 20241222111930.png]]

## 7. Finding Files and Directories (find, locate)
```
find . -name "name of the file"
```
(starting from this current directory)
![[Pasted image 20241222112049.png]]

![[Pasted image 20241222112100.png]]

## 8. Soft and Hard Links (ln)
![[Pasted image 20241222112211.png]]
![[Pasted image 20241222112217.png]]
![[Pasted image 20241222112224.png]]

## 9. Access Control List (ACL)
**getfacl** : Get information 
**setfacl** : set  permission
![[Pasted image 20241222112739.png]]


## 10. Help Commands
![[Pasted image 20241222112830.png]]

## 11. Input and Output Redirects (>, >>, <, stdin, stdout and stderr)

![[Pasted image 20241222112908.png]]
- [!] Using ">"  for override
   Using ">>" for append

![[Pasted image 20241222113102.png]]
## 12. Standard Output to a File (tee command)
![[Pasted image 20241222113251.png]]
## 13.  Pipes ( | )

![[Pasted image 20241222113353.png]]

## 14. File Maintenance Commands (cp, rm, mv, mkdir, rmdir)

## 15. File Display Commands (cat, less, more, head, tail)

## 16. cut - Text Processors Commands
![[Pasted image 20241222113511.png]]

![[Pasted image 20241222113645.png]]
![[Pasted image 20241222113656.png]]

## 17. awk - Text Processors Commands
![[Pasted image 20241222114148.png]]
- [!] -F: this is delimiter < you can choose something else )
![[Pasted image 20241222114227.png]]
## 18. grep/egrep - Text Processors Commands
![[Pasted image 20241222114440.png]]


![[Pasted image 20241222114507.png]]

![[Pasted image 20241222114524.png]]



![[Pasted image 20241222114555.png]]

![[Pasted image 20241222114603.png]]
![[Pasted image 20241222114625.png]]
![[Pasted image 20241222114630.png]]

![[Pasted image 20241222114753.png]]
![[Pasted image 20241222114743.png]]


![[Pasted image 20241222114812.png]]
![[Pasted image 20241222114816.png]]



![[Pasted image 20241222114832.png]]
![[Pasted image 20241222114853.png]]


![[Pasted image 20241222114914.png]]
![[Pasted image 20241222114922.png]]


![[Pasted image 20241222114944.png]]
![[Pasted image 20241222114951.png]]

## 19.  sort/uniq - Text Processors Commands

![[Pasted image 20241222115107.png]]
![[Pasted image 20241222115113.png]]

![[Pasted image 20241222115127.png]]

![[Pasted image 20241222115140.png]]
![[Pasted image 20241222115147.png]]

![[Pasted image 20241222115217.png]]
![[Pasted image 20241222115222.png]]

![[Pasted image 20241222115247.png]]

![[Pasted image 20241222115306.png]]


![[Pasted image 20241222115341.png]]


## 20.  wc - Text Processors Commands
![[Pasted image 20241222115516.png]]

![[Pasted image 20241222115549.png]]
![[Pasted image 20241222115601.png]]


![[Pasted image 20241222115616.png]]
![[Pasted image 20241222115625.png]]


![[Pasted image 20241222115723.png]]

## 21. Compare Files (diff and cmp)

![[Pasted image 20241222115802.png]]

![[Pasted image 20241222120155.png]]

## 22. Compress and uncompress (tar, gzip, gunzip)
![[Pasted image 20241222120252.png]]

![[Pasted image 20241222120307.png]]
![[Pasted image 20241222120320.png]]


- Untar
![[Pasted image 20241222120340.png]]
![[Pasted image 20241222120353.png]]

## 23. Truncate File Size (truncate)

![[Pasted image 20241222120520.png]]


![[Pasted image 20241222120539.png]]

![[Pasted image 20241222120559.png]]
![[Pasted image 20241222120603.png]]

## 24. Combining and Splitting Files
![[Pasted image 20241222120634.png]]


# II. System administration

## 1. Monitor Users (who, last, w, id)
- **who** : How many users that are logging in this system
![[Pasted image 20241222121252.png]]

- **last** : show a listing of last logged in users
![[Pasted image 20241222121428.png]]

- **w** : similar to **who** but with more information
![[Pasted image 20241222121545.png]]

- **finger** : tracking users ( need to install)
## 2. Talking to Users (users, wall, write)

- **users** :  print the user names of users currently logged in to the current host
![[Pasted image 20241222121826.png]]

- **wall** :  write a message to all users
![[Pasted image 20241222121953.png]]
![[Pasted image 20241222122005.png]]

- **write**  \[user_name]: write — send a message to another user

## 3.  Linux Directory Service - Account Authentication

- We have 2 types of account:
	- Local accounts : that created by command **user add**
	- Domain/ Directory accounts
## 4. Difference between Active Directory, LDAP, IDM, WinBIND, OpenLDAP etc
- [?] What are all these directory services are?
- [?] What are the protocols?

- Active Directory for Microsoft ( Microsoft is the owner and who has built this product.)
- Red Hat build this product called IDM (Identity Manager)
-  OpenLDAP < this is opensource >   < LDAP is a protocol > just like IDM, ... It's a open source and it is specifically used for Linux
- IBM Directory services  => there are directory services
- WinBIND : used in Linux to communicate with Windows (Samba )
	- Samba created this product called WinBIND which allows Windows users or Windows Active Directory users to log into Linux machines using WinBIND.
- LDAP is not a directory service, it's just a protocol. It stands for Lightweight Directory Access Protocol. If you are downloading any of the above directory service or directory structure in your Linux or Windows, you need a protocol to communicate to it, and that protocol is called LDAP.

## 5. System Utility Commands (date, uptime, hostname, uname, which, cal, bc)

- **date** : print or set the system date and time
![[Pasted image 20241222122934.png]]

- **uptime**: how long been the system up for and how many users have been logged into the system,
![[Pasted image 20241222123044.png]]

- **uname** : what system is here ?
![[Pasted image 20241222123131.png]]

- **which** :  tells you the location of your command that you run.
![[Pasted image 20241222123235.png]]

- **cal** :  displays a calendar and the date 
 ![[Pasted image 20241222123354.png]]
![[Pasted image 20241222123425.png]]

## 6. Processes, Jobs and Scheduling

- **Applications** which are also referred to **as services**. An application or a service is like a program that **runs in your computer**.
- A **script** is something that **is written** **in a** **file** and then **packaged it** to in a way that **it will execute**
- [i]  All applications you would have to run that as a script and that will run in the background. (And also all those different commands that we run all of them are also referred to as scripts.)

- [?] What is a process?
- When you run an application or when you start up an application, it actually generates process with it's process ID. Now processes could be one associated to that application.
































































































































































































































