# GSoC 2022: pgmoneta, Write-Ahead Log (WAL) infrastructure

This project aims to integrate the PostgreSQL Write-Ahead Log (WAL) data structures into pgmoneta.

I will work on this project by:
1. Study the PostgreSQL Write-Ahead Log (WAL) format
2. Replace usage of pg_receivewal inside pgmoneta with a pgmoneta native solution
3. Create a foundation for a pgmoneta library that can be used for WAL interaction

For more information, please look at my [proposal](https://docs.google.com/document/d/1KKKDU6iP0GOkAMSdGRyxFJRgW964JFVROnpKkbzWyNw/edit#).

[working doc](https://docs.qq.com/doc/DTXl3Smp5ekhlT0pQ?u=287e9a2da7b4489ba77400085897ec11)
  
# GSoC TimeLine

**Jun 13 - Jun 24**
- [x] Setup development environment
- [x] Compile pgmoneta
- [x] Explore pgmoneta code base 

**Jun 20 - Jul 1**
- [ ] Study the PostgreSQL Write-Ahead Log (WAL) format
  
* Pgmoneta is a backup/restore solution for PostgreSQL. 
Currently, it uses pg_receivewal to stream the write-ahead log from a running PostgreSQL cluster. Pg_receivewal is called a binary from within pgmoneta. 

* I will replace that call with a native solution that uses the streaming protocol to get the WAL from the server. 

* When implementing the code, I will look into pg_receivewal and come up with my own native solution. 

* Also, pg_receivewal does not acknowledge reply, so I will try to fix this with the native solution.

* You have to make a clean-room implementation so you will depend on code only in our repository.

* After the call to pgmoneta_server_authenticate you will have an authenticated socket connection to PostgreSQL. Using that socket you can then create messages (message.h|c) and communicate with the server. So, you need to figure out which messages that you need and how that translate into a similar functionality that we have using pg_receiverwal now.


**Jul 4 - Jul 8**
Hi jesper,


- [ ] Come up with a feasible design of replace usage of pg_receivewal inside pgmoneta with a pgmoneta native solution
- [ ] Discuss with mentor

**Jul 11 - Aug 13**
- [ ] Code implementation

**Aug 15 - Aug 26**
- [ ] Testing

**Aug 29 - Sep 2**
- [ ] Documentation

**Sep 5 - Sep 9**
- [ ] Write a summary for the project

**Sep 12 - Sep 16**
- [ ] Figure out a way to create a foundation for a pgmoneta library that can be used for WAL interaction
- [ ] Discuss with mentor

**Sep 19 - Sep 23**
- [ ] Design data structures to understand the PostgreSQL WAL format
- [ ] Provide a mechanism to load WAL records into memory and have a representation of the record

**Sep 26 - Oct 21**
- [ ] Code implementation

**Oct 24 - Oct 28**
- [ ] Testing

**Oct 31 - Nov 4**
- [ ] Documentation

**Nov 7 - Nov 11**
- [ ] Submit final project and write summary




# make

git clone https://github.com/pgmoneta/pgmoneta.git
cd pgmoneta
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Debug ..
make

# get started


/home/pgmoneta/pgmoneta/build/src/pgmoneta-admin master-key

/home/pgmoneta/pgmoneta/build/src/pgmoneta-admin -f /home/pgmoneta/config/pgmoneta_users.conf -U repl -P secretpassword add-user


/home/pgmoneta/pgmoneta/build/src/pgmoneta -c /home/pgmoneta/config/pgmoneta.conf -u /home/pgmoneta/config/pgmoneta_users.conf


// /home/pgmoneta/pgmoneta/build/src/pgmoneta-cli -c /home/pgmoneta/config/pgmoneta.conf backup primary

# View backup
/home/pgmoneta/pgmoneta/build/src/pgmoneta-cli -c /home/pgmoneta/config/pgmoneta.conf details

lsof -i:5432
kill -9 96572


/usr/pgsql-14/bin/pg_ctl -D /tmp/pgsql/ start

psql -h localhost -p 5432 -U myuser mydb

DROP TABLE t_test;



CREATE TABLE t_test(
    ID INT PRIMARY KEY NOT NULL,
    NAME TEXT NOT NULL,
    AGE INT NOT NULL,
    ADDRESS CHAR(59),
    SALARY REAL
);


insert into t_test SELECT generate_series(1,500000)
as key,repeat( chr(int4(random()*26)+65),4), (random()*(6^2))::integer,null,(random()*(10^4))::integer;

\q

cd /tmp/

pgmoneta -c /home/pgmoneta/config/pgmoneta.conf -u /home/pgmoneta/config/pgmoneta_users.conf


ls /home/pgmoneta/backup/primary/wal/

CMD: PGPASSWORD="secretpassword" /usr/pgsql-14/bin//pg_receivewal -h localhost -p 5432 -U repl --no-loop --no-password -D /home/kk/GSoC/upload/pgmoneta-plus/backup/primary/wal/
