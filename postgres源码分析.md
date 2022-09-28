src\bin\pg_basebackup\pg_receivewal.c

http://www.postgres.cn/docs/14/app-pgreceivewal.html

https://www.cnblogs.com/skpupil/p/16414475.html

# pg_receivewal
```c
src\bin\pg_basebackup\pg_receivewal.c


connection start 
fe-connect.c line 2517

if (connect(conn->sock, addr_cur->ai_addr,
2721


main 468
    初始化，参数解析，获取连接681，检查wal size 714
    StreamLog(); src\bin\pg_basebackup\pg_receivewal.c 760
        stream分配内存，获得数据库连接，检查版本，检查系统
        FindStreamingStart(&stream.timeline); 406---》200  
        //根据目的文件夹中最大id的文件
        //找最后的xlog segment，决定本次的起点。
        //获取目的目录src\port\dirent.c
        //得到文件
        //看是否是partical，是否压缩
        XLogSegmentOffset(stream.startpos, WalSegSz);    
        //找到在segment开始的地方复制下一个
        //设置stream
        ReceiveXlogStream(conn, &stream);   
        //从特定的position开始复制log stream
        //src\bin\pg_basebackup\receivelog.c 437
            while(1){
                if(!existsTimeLineHistoryFile){     
                    //查看本地是否存在wal，如果是从上次结束的地方在开始
                    res = PQexec(conn, "TIMELINE_HISTORY %u");    //538
                    //res = PQexec(conn, START_REPLICATION %s%X/%X TIMELINE %u");
                    //参考http://www.postgres.cn/docs/14/protocol-replication.html
                }  
                    // 执行语句，指示服务器开始启动流WAL，从 WAL 位置XXX/XXX开始
                    // 如果TIMELINE选项被指定，流传送会在时间线tli上开始，否则会选择服务器的当前时间线
                    // 完成这个操作后，后面只需要从本地buffer中读取和处理

                    PQexecStart(conn);
                    PQsendQuery(conn, query);
                        PQsendQueryInternal(conn, query, true);
                            pqPutMsgStart('Q', conn)
                            pqPuts(query, conn)
                                pqPutMsgBytes(s, strlen(s) + 1, conn)
                                    memcpy(conn->outBuffer + conn->outMsgEnd, buf, len);
                                    //这里放到了socket发送buffer中就可以自动完成发送的任务。
                            pqPutMsgEnd(conn)
                    PQexecFinish(conn)

/*
fe-exec.c
line2199 PQexecStart
line2201 PQsendQuery
    line1326 PQsendQueryInternal
        line1626 PQsendQueryStart
            pqPutMsgStart
            pqPuts
            pqPutMsgEnd

            pqPipelineFlush
                line3788 pqFlush
                    line970 fflush    fe-misc.c line 970 
                    line972 pqSendSome
                        line834 ->fe-secure.c 288 pqsecure_write
                            line307 pqsecure_raw_write 
                                line332 send() 这是socket编程linux底层函数
line2203    PQgetResult
    PQgetResult 获得一个执行query的结果返回值
        pqsecure_read
            pqsecure_raw_read
                recv linux socket api
*/

                HandleCopyStream(conn, stream, &stoppos); //处理query返回的stream包
                    CopyStreamReceive()
                        rawlen = PQgetCopyData //从socket buffer中copy到用户态
                            pqGetCopyData3
                                getCopyDataMessage
                                    pqGetc //获取一个char，表示类型
                                    pqGetInt //int 表示这个包的size length
                            malloc //如果rawlen!=0, 根据获得的长度，分配内存，并从socketbuf中拷贝到用户态
                            memcpy
                        if(rawlen == 0)
                            CopyStreamPoll
                                select //使用linux select等待数据到达，直到超时
                            pqReadData //select拿到通知，数据已经到达
                                pqReadData
                                    pqsecure_read
                                        pqsecure_raw_read
                                            recv //linux socket api 从socket中收数据
                    while(1) { 
                        r = CopyStreamReceive(conn, sleeptime, stream->stop_socket, &copybuf);
                            rawlen = PQgetCopyData(conn, &copybuf, 1);
                                return pqGetCopyData3(conn, buffer, async);
                                    for;;{
                                        msgLength = getCopyDataMessage(conn); 
                                        //获取一行的message数据，message是stream的控制信息，后面的keep live，
                                        //或者是wal data等信息。
                                            pqGetc(&id, conn) //从socket连接中获取
                                            pqGetInt //int 表示这个包的size length
                                        malloc //根据获得的长度，分配内存，并从socketbuf中拷贝到用户态
                                        memcpy
                                    }
                        while (r != 0)//一次query持续在这个循环中接收和处理服务器发送来的package，
                        //除非超时退出此循环
		                {
                              根据获取到的服务器massage，不同操作
                              if (copybuf[0] == 'k')
                              {
                                  ProcessKeepaliveMsg(conn, stream, copybuf, r, blockpos,
                                                          &last_status)
                              }
                              else if (copybuf[0] == 'w')
                              {
                                  ProcessXLogDataMsg(conn, stream, copybuf, r, &blockpos)
                                      write(walfile, copybuf + hdr_len + bytes_written,bytes_to_write) 
                                      //这里完成写本地文件，完成备份。
                              }
                              CopyStreamReceive //同上
                        }
                    }
            }

```

# PQexec
```
fe-exec.c
line2199 PQexecStart
line2201 PQsendQuery
    line1326 PQsendQueryInternal
        line1626 PQsendQueryStart
            pqPutMsgStart
            pqPuts
            pqPutMsgEnd

            pqPipelineFlush
                line3788 pqFlush
                    line970 fflush    fe-misc.c line 970 
                    line972 pqSendSome
                        line834 ->fe-secure.c 288 pqsecure_write
                            line307 
                                line332 send() 这是socket编程linux底层函数
line2203    PQgetResult
    PQgetResult 获得一个执行query的结果返回值
        pqsecure_read
            pqsecure_raw_read
                recv linux socket api


fe-misc.c line 970
结合手册 196页：都可以使用 fflush()库函数强制将 stdio 输出
流中的数据（即通过 write()）刷新到内核缓冲区中

```


```
把dtm 以 格式 输入到 后面的变量
sscanf( dtm, "%s %s %d  %d", weekday, month, &day, &year );

```



log
------   ------   ----|--- 



CMD: PGPASSWORD="secretpassword" /usr/pgsql-14/bin//pg_receivewal -h localhost -p 5432 -U repl --no-loop --no-password -D /home/kk/GSoC/upload/pgmoneta-plus/backup/primary/wal/



为什么需要这个项目，
原来：本地数据库pg_receivewal，首先安装本地数据库，再发起一个socket连接。
要做的工作：pgmoneta 已经建立一个socket连接，重用这个连接，完成wal备份。把远程目录，使用socket，复制到本地。



# RowDescription (B)
Byte1('T')
Identifies the message as a row description.

Int32
Length of message contents in bytes, including self.

Int16
Specifies the number of fields in a row (can be zero).

Then, for each field, there is the following:

String
The field name.

Int32
If the field can be identified as a column of a specific table, the object ID of the table; otherwise zero.

Int16
If the field can be identified as a column of a specific table, the attribute number of the column; otherwise zero.

Int32
The object ID of the field's data type.

Int16
The data type size (see pg_type.typlen). Note that negative values denote variable-width types.

Int32
The type modifier (see pg_attribute.atttypmod). The meaning of the modifier is type-specific.

Int16
The format code being used for the field. Currently will be zero (text) or one (binary). In a RowDescription returned from the statement variant of Describe, the format code is not yet known and will always be zero.


# Query (F)
Byte1('Q')
Identifies the message as a simple query.

Int32
Length of message contents in bytes, including self.

String
The query string itself.