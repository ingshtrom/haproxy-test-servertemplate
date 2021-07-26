# Reproduce environment
```
k3d cluster create
export KUBECONFIG="$HOME/.k3d/kubeconfig-k3s-default.yaml"
kubectl cluster-info
docker build . -t test-haproxy
k3d images import test-haproxy
kubectl apply -f headless.yaml
kubectl apply -f haproxy.yaml
pod=$(kubectl get po -lapp=haproxy --no-headers -o jsonpath='{.items[*].metadata.name}')
kubectl scale deploy/haproxy --replicas=10
```


## Working Example with this repository

1) the exact version output where you had this issue (-vv from binary)
```
> kubectl exec -it $pod -- /opt/hapee-2.2/sbin/hapee-lb -vv
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

See the `haproxy.yaml` ConfigMap portion.

3) Output of show servers state <backend>  from the Runtime API

```
> kubectl exec -it $pod -- bash -c 'echo "show servers state headless-test" | socat stdio unix-connect:/run/haproxy.sock'
1
# be_id be_name srv_id srv_name srv_addr srv_op_state srv_admin_state srv_uweight srv_iweight srv_time_since_last_change srv_check_status srv_check_result srv_check_health srv_check_state srv_agent_state bk_f_forced_id srv_f_forced_id srv_fqdn srv_port srvrecord
5 headless-test 1 srv1 10.42.0.73 2 64 1 1 51 15 3 4 6 0 0 0 10-42-0-73.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 2 srv2 10.42.0.44 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-44.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 3 srv3 10.42.0.61 2 64 1 1 51 15 3 4 6 0 0 0 10-42-0-61.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 4 srv4 10.42.0.69 2 64 1 1 51 15 3 4 6 0 0 0 10-42-0-69.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 5 srv5 10.42.0.60 2 64 1 1 51 15 3 4 6 0 0 0 10-42-0-60.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 6 srv6 10.42.0.68 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-68.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 7 srv7 10.42.0.65 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-65.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 8 srv8 10.42.0.63 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-63.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 9 srv9 10.42.0.58 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-58.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 10 srv10 10.42.0.70 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-70.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 11 srv11 10.42.0.71 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-71.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 12 srv12 10.42.0.72 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-72.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 13 srv13 10.42.0.59 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-59.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 14 srv14 10.42.0.67 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-67.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
5 headless-test 15 srv15 10.42.0.64 2 64 1 1 350 15 3 4 6 0 0 0 10-42-0-64.headless-test.default.svc.cluster.local 80 _headless._tcp.headless-test.default.svc.cluster.local
```

4) dig output for corresponding SRV record

```
> k exec -it $pod -- dig _headless._tcp.headless-test.default.svc.cluster.local SRV

; <<>> DiG 9.16.1-Ubuntu <<>> _headless._tcp.headless-test.default.svc.cluster.local SRV
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9231
;; flags: qr aa rd; QUERY: 1, ANSWER: 20, AUTHORITY: 0, ADDITIONAL: 21
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: d322fd56a2ec8990 (echoed)
;; QUESTION SECTION:
;_headless._tcp.headless-test.default.svc.cluster.local.        IN SRV

;; ANSWER SECTION:
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-44.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-68.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-65.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-63.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-58.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-70.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-71.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-72.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-59.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-67.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-64.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-73.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-61.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-69.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-60.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-62.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-66.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-74.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-75.headless-test.default.svc.cluster.local.
_headless._tcp.headless-test.default.svc.cluster.local. 2 IN SRV 0 5 80 10-42-0-76.headless-test.default.svc.cluster.local.

;; ADDITIONAL SECTION:
10-42-0-62.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.62
10-42-0-64.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.64
10-42-0-71.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.71
10-42-0-74.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.74
10-42-0-75.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.75
10-42-0-44.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.44
10-42-0-65.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.65
10-42-0-73.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.73
10-42-0-61.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.61
10-42-0-66.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.66
10-42-0-59.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.59
10-42-0-58.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.58
10-42-0-60.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.60
10-42-0-69.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.69
10-42-0-63.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.63
10-42-0-68.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.68
10-42-0-70.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.70
10-42-0-76.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.76
10-42-0-72.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.72
10-42-0-67.headless-test.default.svc.cluster.local. 2 IN A 10.42.0.67

;; Query time: 1 msec
;; SERVER: 10.43.0.10#53(10.43.0.10)
;; WHEN: Mon Jul 26 16:29:59 UTC 2021
;; MSG SIZE  rcvd: 1815
```

Were you able to replicate this consistently or sporadically? 

Nope, never. I'm not sure what is different...
