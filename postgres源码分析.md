src\bin\pg_basebackup\pg_receivewal.c

http://www.postgres.cn/docs/14/app-pgreceivewal.html

```C
//main 468
//    初始化，参数解析，获取连接681，检查wal size 714
    StreamLog(); //760
        stream分配内存，获得数据库连接，检查版本，检查系统
        FindStreamingStart(&stream.timeline); //406---》200  //根据目的文件夹中最大id的文件
                                              //   找最后的xlog segment，决定本次的起点。
                                              // 获取目的目录src\port\dirent.c
                                              // 得到文件
                                              // 看是否是partical，是否压缩
        XLogSegmentOffset(stream.startpos, WalSegSz);    // 找到在segment开始的地方复制下一个
                                                         // 设置stream
        ReceiveXlogStream(conn, &stream);   //从特定的position开始复制log stream
                                            // src\bin\pg_basebackup\receivelog.c
            while(1){
                if(!existsTimeLineHistoryFile){
                    res = PQexec(conn, "TIMELINE_HISTORY %u");
                    //参考http://www.postgres.cn/docs/14/protocol-replication.html
                }  
                //执行语句，指示服务器开始启动流WAL，从 WAL 位置XXX/XXX开始。
                //如果TIMELINE选项被指定，流传送会在时间线tli上开始，
                //否则会选择服务器的当前时间线。完成这个操作后，
                //后面只需要从本地buffer中读取和处理。
                res = PQexec(conn, START_REPLICATION %s%X/%X TIMELINE %u); 
                    PQexecStart(conn)
                    PQsendQuery(conn, query)
                        PQsendQueryInternal(conn, query, true);
                            pqPutMsgStart('Q', conn)
                            pqPuts(query, conn)
                                pqPutMsgBytes(s, strlen(s) + 1, conn)
                                    //这里放到了socket发送buffer中就可以自动完成发送的任务。
                                    memcpy(conn->outBuffer + conn->outMsgEnd, buf, len);
                            pqPutMsgEnd(conn)
                    PQexecFinish(conn)

                res = HandleCopyStream(conn, stream, &stoppos);
                    while(1) {
                        r = CopyStreamReceive(conn, sleeptime, stream->stop_socket, &copybuf);
                            rawlen = PQgetCopyData(conn, &copybuf, 1);
                                return pqGetCopyData3(conn, buffer, async);
                                    for;;{
                                        //获取一行的message数据，message是stream的控制信息，
                                        //后面的keep live，或者是wal data等信息。
                                        msgLength = getCopyDataMessage(conn); 
                                            pqGetc(&id, conn) //从socket连接中获取
                                    }
                        //根据获取到的服务器massage，不同操作
                        if (copybuf[0] == 'k')
                        {
                            ProcessKeepaliveMsg(conn, stream, copybuf, r, blockpos,
                                                    &last_status)
                        }
                        else if (copybuf[0] == 'w')
                        {
                            ProcessXLogDataMsg(conn, stream, copybuf, r, &blockpos)
                                write(walfile, copybuf + hdr_len + bytes_written,bytes_to_write) //这里完成写本地文件，完成备份。
                        }
                    }
            }
         

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
