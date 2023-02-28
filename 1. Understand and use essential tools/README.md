# 1. Access a shell prompt and issue commands with correct syntax

`CTRL + ALT + F1..6` - allows us to change terminal session

# 2. User input-ouput redirection (>, >>, |, 2>, etc.)
* `>` - appends to file, overwrites
* `>>` - appends to file, does not overwrite data
* `|` - send data to another command
* `STDIN (0)`
* `STDOUT (1)`
* `STDERR (2)`
```
[root@rhcsa-node-1 ~]# find . type f -name 'ajr*' 2>error.log
./ajr                                                        
[root@rhcsa-node-1 ~]# cat error.log                         
find: ‘type’: No such file or directory                      
find: ‘f’: No such file or directory                         
[root@rhcsa-node-1 ~]# find . type f -name 'ajr*' 2>&1       
./ajr                                                        
find: ‘type’: No such file or directory                      
find: ‘f’: No such file or directory                         
```

# 2. Use grep and regular expressions to analyze text
```
grep 'some_value' <some_file> # take all occurrences of the file and print to screen
grep -v 'some_value <some_file> # excludes lines from a file
grep -E # supports regular expressions
grep -i # case insensitive
```

# 3. Access remote systems using SSH
```
ssh 192.168.122.10 # ssh as current user
ssh user1@192.168.122.10 # ssh as user1
ssh -I ~/.ssh_id_rsa 192.168.122.10 # ssh to a server, specifying the identity file/ssh key
```

# 4. Login and switch users in multiuser targets
```
[root@rhcsa-node-1 ~]# systemctl get-default                  
multi-user.target                                             
[root@rhcsa-node-1 ~]# systemctl set-default multi-user.target
```

# 5. Archive, compress, unpack, and uncompress files using tar, star, gzip, and bzip2
```
[root@rhcsa-node-1 ~]# tar -czvf tmp.tar.gz /tmp
tar: Removing leading `/' from member names     
/tmp/                                           
[root@rhcsa-node-1 ~]# tar -xzvf tmp.tar.gz     
tmp/                                            
[root@rhcsa-node-1 ~]# touch file               
[root@rhcsa-node-1 ~]# gzip file                
[root@rhcsa-node-1 ~]# gunzip file              
[root@rhcsa-node-1 ~]#                          
```

# 6. Create and edit text files

# 7. Create, delete, copy, and move files and directories
```
touch file.txt
echo 'hello'> file.txt
mkdir ~/directory
mkdir -p ~/directory/subdirectory1/subdirectory2
rm -rf ~/directory
cp file1 file
cp -r directory1/ new_directory2/
mv file1 file2
```

# 8. Create hard and soft links
```
[root@rhcsa-node-1 ~]# echo 'hello' > file1  
[root@rhcsa-node-1 ~]# ln file1 file2        
[root@rhcsa-node-1 ~]# cat file2             
hello                                        
[root@rhcsa-node-1 ~]# ls -i file*           
34952705 file  34952706 file1  34952706 file2
[root@rhcsa-node-1 ~]# rm file2              
rm: remove regular file 'file2'? y           
[root@rhcsa-node-1 ~]# ln -sv file1 file2    
'file2' -> 'file1'                           
[root@rhcsa-node-1 ~]# cat file2             
hello                                        
[root@rhcsa-node-1 ~]# ls -i file*           
34952705 file  34952706 file1  34952707 file2
```
# 9. List, set, and change standard ugo/rwx permissions
* `4 - write`
* `2 - read`
* `1 - execute`

We can also use the non-octal variant of `wr, r, x`
This can be used as follows: `chown u+rwx, g+rwx, o+rwx`
```
[root@rhcsa-node-1 ~]# touch file.txt                                    
[root@rhcsa-node-1 ~]# ls -l file.txt                                    
-rw-r--r--. 1 root root 0 Feb 28 19:43 file.txt                          
[root@rhcsa-node-1 ~]# mkdir directory1                                  
[root@rhcsa-node-1 ~]# ls -ld directory1                                 
drwxr-xr-x. 2 root root 6 Feb 28 19:44 directory1                        
[root@rhcsa-node-1 ~]# chmod -R 755 directory1/                          
[root@rhcsa-node-1 ~]# ls -ld directory1/                                
drwxr-xr-x. 2 root root 6 Feb 28 19:44 directory1/                       
[root@rhcsa-node-1 ~]# stat directory1/                                  
  File: directory1/                                                      
  Size: 6               Blocks: 0          IO Block: 4096   directory    
Device: fd00h/64768d    Inode: 1497161     Links: 2                      
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root) 
Context: unconfined_u:object_r:admin_home_t:s0                           
```

# 10. Locate, read, and use system documentation including man, info, and files in /usr/share/doc
```
man -k passwd # show all manpages with the similar string
/usr/local/share/doc # location of docs for various packages installed
info # newer to man, more verbosity on documents
info -k passwd # show all info pages with this similar string
```
