# in main.c, calls pgmoneta_wal:



1685 static void init_receivewals(void)
 char follow[MISC_LENGTH];           /**< Follow a server */

1486 static void wal_streaming_cb(struct ev_loop* loop, ev_periodic* w, int revents)
   bool wal_streaming;                 /**< Is WAL streaming active */