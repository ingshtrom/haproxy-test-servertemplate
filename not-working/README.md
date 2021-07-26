## Not working with internal infra

1) the exact version output where you had this issue (-vv from binary)
```
root@11e42a406cdb:/tmp# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  3.7 193088 75512 ?        Ss   16:48   0:00 /opt/hapee-2.2/sbin/hapee-lb -W -db -f /etc/hap
root         9  0.2  3.4 352808 70724 ?        Sl   16:48   0:00 /opt/hapee-2.2/sbin/hapee-lb -W -db -f /etc/hap
root        13  0.7  0.1   4248  3500 pts/0    Ss   16:50   0:00 bash
root        25  0.0  0.1   5904  2952 pts/0    R+   16:50   0:00 ps aux
root@11e42a406cdb:/tmp# /opt/hapee-2.2/sbin/hapee-lb -vv
HA-Proxy version 2.2.0-1.0.0-238.455 2021/07/08 - https://haproxy.org/
Status: long-term supported branch - will stop receiving fixes around February 2025.
Known bugs: https://www.haproxy.com/documentation/hapee/2-2r1/onepage/changelog/#2.2.0
Running on: Linux 5.10.25-linuxkit #1 SMP Tue Mar 23 09:27:39 UTC 2021 x86_64
Build options :
  TARGET  = linux-glibc
  CPU     = generic
  CC      = gcc
  CFLAGS  = -O2 -g -Wall -Wextra -Wdeclaration-after-statement -fwrapv -Wno-address-of-packed-member -Wno-unused-label -Wno-sign-compare -Wno-unused-parameter -Wno-clobbered -Wno-missing-field-initializers -Wno-stringop-overflow -Wno-cast-function-type -Wtype-limits -Wshift-negative-value -Wshift-overflow=2 -Wduplicated-cond -Wnull-dereference -Werror -DMAX_SESS_STKCTR=12 -DSTKTABLE_EXTRA_DATA_TYPES=10
  OPTIONS = USE_PCRE_JIT=1 USE_LINUX_TPROXY=1 USE_LINUX_SPLICE=1 USE_OPENSSL=1 USE_LUA=1 USE_SLZ=1 USE_CPU_AFFINITY=1 USE_NS=1 USE_SYSTEMD=1 USE_MODULES=1
  DEBUG   =

Feature list : +EPOLL -KQUEUE +NETFILTER -PCRE +PCRE_JIT -PCRE2 -PCRE2_JIT +POLL -PRIVATE_CACHE +THREAD -PTHREAD_PSHARED +BACKTRACE -STATIC_PCRE -STATIC_PCRE2 +TPROXY +LINUX_TPROXY +LINUX_SPLICE +LIBCRYPT +CRYPT_H +GETADDRINFO +OPENSSL +LUA +FUTEX +ACCEPT4 -CLOSEFROM -ZLIB +SLZ +CPU_AFFINITY +TFO +NS +DL +RT -DEVICEATLAS -51DEGREES -WURFL +SYSTEMD -OBSOLETE_LINKER +PRCTL +THREAD_DUMP -EVPORTS +MODULES -MEMORY_PROFILING

Default settings :
  bufsize = 16384, maxrewrite = 1024, maxpollevents = 200

Built with multi-threading support (MAX_THREADS=64, default=4).
Built with libslz for stateless compression.
Compression algorithms supported : identity("identity"), deflate("deflate"), raw-deflate("deflate"), gzip("gzip")
Built with transparent proxy support using: IP_TRANSPARENT IPV6_TRANSPARENT IP_FREEBIND
Built with PCRE version : 8.39 2016-06-14
Running on PCRE version : 8.39 2016-06-14
PCRE library supports JIT : yes
Encrypted password support via crypt(3): yes
Built with gcc compiler version 9.3.0
Built with the Prometheus exporter as a service
Built with OpenSSL version : OpenSSL 1.1.1f  31 Mar 2020
Running on OpenSSL version : OpenSSL 1.1.1f  31 Mar 2020
OpenSSL library supports TLS extensions : yes
OpenSSL library supports SNI : yes
OpenSSL library supports : TLSv1.0 TLSv1.1 TLSv1.2 TLSv1.3
Built with Lua version : Lua 5.3.3
Built with network namespace support.

Available polling systems :
      epoll : pref=300,  test result OK
       poll : pref=200,  test result OK
     select : pref=150,  test result OK
Total: 3 (3 usable), will use epoll.

Available multiplexer protocols :
(protocols marked as <default> cannot be specified using 'proto' keyword)
            fcgi : mode=HTTP       side=BE        mux=FCGI
       <default> : mode=HTTP       side=FE|BE     mux=H1
              h2 : mode=HTTP       side=FE|BE     mux=H2
       <default> : mode=TCP        side=FE|BE     mux=PASS

Available services : prometheus-exporter
Available filters :
        [SPOE] spoe
        [COMP] compression
        [TRACE] trace
        [CACHE] cache
        [FCGI] fcgi-app
```

2) your configuration (most important part here would be resolvers section).

See the `haproxy-internal.cfg` file.

3) Output of show servers state <backend>  from the Runtime API

```

root@11e42a406cdb:/tmp# echo 'show servers state headless-test' | socat stdio unix-connect:/run/haproxy.sock
1
# be_id be_name srv_id srv_name srv_addr srv_op_state srv_admin_state srv_uweight srv_iweight srv_time_since_last_change srv_check_status srv_check_result srv_check_health srv_check_state srv_agent_state bk_f_forced_id srv_f_forced_id srv_fqdn srv_port srvrecord
5 headless-test 1 srv1 - 0 96 1 1 153 15 3 0 14 0 0 0 - 0 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 2 srv2 - 0 96 1 1 153 15 3 0 14 0 0 0 - 0 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 3 srv3 - 0 96 1 1 153 15 3 0 14 0 0 0 - 0 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 4 srv4 - 0 96 1 1 153 15 3 0 14 0 0 0 - 0 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 5 srv5 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 6 srv6 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 7 srv7 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 8 srv8 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 9 srv9 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 10 srv10 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 11 srv11 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 12 srv12 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 13 srv13 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 14 srv14 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
5 headless-test 15 srv15 - 0 32 1 1 197 1 0 0 14 0 0 0 - 80 _headless._tcp.headless-test.infra-routing.svc.cluster.local
```

4) dig output for corresponding SRV record

From **within** docker container running locally and trying to hit CoreDNS in EKS:
```
root@11e42a406cdb:/tmp# dig _headless._tcp.headless-test.infra-routing.svc.cluster.local SRV @10.128.115.161

; <<>> DiG 9.16.1-Ubuntu <<>> _headless._tcp.headless-test.infra-routing.svc.cluster.local SRV @10.128.115.161
;; global options: +cmd
;; connection timed out; no servers could be reached
```

From **host** which has direct access to CoreDNS in EKS over VPN:
```
> dig _headless._tcp.headless-test.infra-routing.svc.cluster.local SRV @10.128.115.161

; <<>> DiG 9.10.6 <<>> _headless._tcp.headless-test.infra-routing.svc.cluster.local SRV @10.128.115.161
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20211
;; flags: qr aa rd; QUERY: 1, ANSWER: 20, AUTHORITY: 0, ADDITIONAL: 21
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;_headless._tcp.headless-test.infra-routing.svc.cluster.local. IN SRV

;; ANSWER SECTION:
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-104-111.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-104-222.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-109-235.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-117-129.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-119-220.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-125-69.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-130-110.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-135-112.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-146-85.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-148-153.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-153-200.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-66-249.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-69-229.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-72-65.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-73-22.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-74-133.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-76-177.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-85-86.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-89-58.headless-test.infra-routing.svc.cluster.local.
_headless._tcp.headless-test.infra-routing.svc.cluster.local. 2 IN SRV 0 5 80 10-128-98-0.headless-test.infra-routing.svc.cluster.local.

;; ADDITIONAL SECTION:
10-128-98-0.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.98.0
10-128-104-222.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.104.222
10-128-130-110.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.130.110
10-128-148-153.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.148.153
10-128-135-112.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.135.112
10-128-73-22.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.73.22
10-128-74-133.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.74.133
10-128-76-177.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.76.177
10-128-89-58.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.89.58
10-128-104-111.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.104.111
10-128-69-229.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.69.229
10-128-72-65.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.72.65
10-128-119-220.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.119.220
10-128-117-129.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.117.129
10-128-109-235.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.109.235
10-128-66-249.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.66.249
10-128-125-69.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.125.69
10-128-85-86.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.85.86
10-128-153-200.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.153.200
10-128-146-85.headless-test.infra-routing.svc.cluster.local. 2 IN A 10.128.146.85

;; Query time: 41 msec
;; SERVER: 10.128.115.161#53(10.128.115.161)
;; WHEN: Mon Jul 26 12:52:51 EDT 2021
;; MSG SIZE  rcvd: 1992


```

Were you able to replicate this consistently or sporadically? 

Yup, I can reproduce every time I have tried.


**Logs from haproxy:**
```
haproxy  | [NOTICE] 206/164845 (1) : New worker #1 (9) forked
haproxy  | [WARNING] 206/164845 (9) : headless-test/srv1 changed its IP from (none) to 10.128.89.58 by DNS additional record.
haproxy  | [WARNING] 206/164845 (9) : Server headless-test/srv1 ('10-128-89-58.headless-test.infra-routing.svc.cluster.local') is UP/READY (resolves again).
haproxy  | [WARNING] 206/164916 (9) : headless-test/srv2 changed its IP from (none) to 10.128.117.129 by DNS additional record.
haproxy  | [WARNING] 206/164916 (9) : Server headless-test/srv2 ('10-128-117-129.headless-test.infra-routing.svc.cluster.local') is UP/READY (resolves again).
haproxy  | [WARNING] 206/164916 (9) : headless-test/srv3 changed its IP from (none) to 10.128.119.220 by DNS additional record.
haproxy  | [WARNING] 206/164916 (9) : Server headless-test/srv3 ('10-128-119-220.headless-test.infra-routing.svc.cluster.local') is UP/READY (resolves again).
haproxy  | [WARNING] 206/164916 (9) : headless-test/srv4 changed its IP from (none) to 10.128.125.69 by DNS additional record.
haproxy  | [WARNING] 206/164916 (9) : Server headless-test/srv4 ('10-128-125-69.headless-test.infra-routing.svc.cluster.local') is UP/READY (resolves again).
haproxy  | [WARNING] 206/164929 (9) : Server headless-test/srv1 is going DOWN for maintenance (entry removed from SRV record). 3 active and 0 backup servers left. 0 sessions active, 0 requeued, 0 remaining in queue.
haproxy  | [WARNING] 206/164929 (9) : Server headless-test/srv2 is going DOWN for maintenance (entry removed from SRV record). 2 active and 0 backup servers left. 0 sessions active, 0 requeued, 0 remaining in queue.
haproxy  | [WARNING] 206/164929 (9) : Server headless-test/srv3 is going DOWN for maintenance (entry removed from SRV record). 1 active and 0 backup servers left. 0 sessions active, 0 requeued, 0 remaining in queue.
haproxy  | [WARNING] 206/164929 (9) : Server headless-test/srv4 is going DOWN for maintenance (entry removed from SRV record). 0 active and 0 backup servers left. 0 sessions active, 0 requeued, 0 remaining in queue.
haproxy  | [NOTICE] 206/164929 (9) : haproxy version is 2.2.0-1.0.0-238.455
haproxy  | [NOTICE] 206/164929 (9) : path to executable is /opt/hapee-2.2/sbin/hapee-lb
haproxy  | [ALERT] 206/164929 (9) : backend 'headless-test' has no server available!
```

Furthermore, I found that when running `docker run -ti ubuntu` and then `dig_headless._tcp.headless-test.infra-routing.svc.cluster.local SRV @10.128.115.161`
resulted in the same client timeout, even when running the same commands locally (Mac OSX) still
worked fine. I'm wondering if maybe there is some restriction somewhere on the payload size for the
UDP packet, or maybe they just weren't able to be reconstructed since UDP DNS packets have the 8192
byte size limit.

If I run `dig _headless._tcp.headless-test.infra-routing.svc.cluster.local SRV @10.128.115.161 +tcp`
from within the same environment (the ubuntu container), then everything resolves as expected.

# Conclusion

So it looks like HAPEE crashes when it times out attempting to make the DNS query.
