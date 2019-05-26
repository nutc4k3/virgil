# flaws.cloud

A DNS lookup of the domain returns the following records:

```bash
➜  ~ dig flaws.cloud any

; <<>> DiG 9.10.6 <<>> flaws.cloud any
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48150
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;flaws.cloud.			IN	ANY

;; ANSWER SECTION:
flaws.cloud.		5	IN	A	52.218.217.42
flaws.cloud.		21600	IN	NS	ns-966.awsdns-56.net.
flaws.cloud.		21600	IN	NS	ns-1061.awsdns-04.org.
flaws.cloud.		21600	IN	NS	ns-1890.awsdns-44.co.uk.
flaws.cloud.		21600	IN	NS	ns-448.awsdns-56.com.
flaws.cloud.		879	IN	SOA	ns-1890.awsdns-44.co.uk. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400

;; Query time: 44 msec
;; SERVER: 192.168.0.1#53(192.168.0.1)
;; WHEN: Sun May 26 20:50:44 BST 2019
;; MSG SIZE  rcvd: 257
```

The A record returns 52.218.217.42

```bash
➜  ~ nslookup 52.218.217.42
Server:		192.168.0.1
Address:	192.168.0.1#53

Non-authoritative answer:
42.217.218.52.in-addr.arpa	name = s3-website-us-west-2.amazonaws.com.

Authoritative answers can be found from:
```

It's hosted in an S3 bucket. We can attempt to read stored files in the bucket as follows:

```text
➜  ~ aws s3 ls s3://flaws.cloud --no-sign-request
2017-03-14 03:00:38       2575 hint1.html
2017-03-03 04:05:17       1707 hint2.html
2017-03-03 04:05:11       1101 hint3.html
2018-07-10 17:47:16       3082 index.html
2018-07-10 17:47:16      15979 logo.png
2017-02-27 01:59:28         46 robots.txt
2017-02-27 01:59:30       1051 secret-dd02c7c.html
```

This indicates it's permissions are faulty.

Viewing [http://flaws.cloud/secret-dd02c7c.html](http://flaws.cloud/secret-dd02c7c.html) reveals the secret URL for level 2. [http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/](http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/)

