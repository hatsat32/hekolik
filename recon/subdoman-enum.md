# Subdoman Enum

#### Manuel

* dnsdumpter
* n99l

#### Passive

* `subfinder -d $domain -all -nC -recursive -silent -t 50 -config subfinder.yml -o example.com.subs.subfinder`
* `amass enum -list -config config.ini` api keyleri kontrol et
* `amass enum -passive -config config.ini -d example.com -o example.com.subs.amass.passive`
* `amass enum -active -brute -w subdomain-wordlist.txt -d example.com -o example.com.subs.amass.active`
* `assetfinder -subs-only example.com > example.com.subs.assetfinder`
  * `cat domains | assetfinder -subs-only`
* `findomain -t example.com -u example.com.subs.findomain`
* `sudomy -d example.com` → \~/Sec

#### İnternet Arşivleri

* `echo example.com | gau -subs -t 10 -o out.wayback.gau`
* `gauplus -t 5 -random-agent -subs example.com | unfurl -u domains | anew output.txt`
* `waybackurls example.com | unfurl -u domains | sort -u output.txt`

#### Github Subdomain Enum

* `github-subdomains -d example.com -t tokens.txt -o output.txt`

#### Rapid7

* Rapid7 Project Sonar `crobat`
* `crobat -s example.com > output.txt`

#### Sertifikalar

* `python3 ctfr.py -d target.com -o output.txt`
* `curl "<https://tls.bufferover.run/dns?q=.dell.com>" | jq -r .Results[] | cut -d ',' -f4 | grep -F ".dell.com" | anew -q output.txt`
* `curl "<https://dns.bufferover.run/dns?q=.dell.com>" | jq -r '.FDNS_A'[],'.RDNS'[] | cut -d ',' -f2 | grep -F ".dell.com" | anew -q output.txt`

#### Recursive

```shell
for sub in $( ( cat subdomains.txt | rev | cut -d '.' -f 3,2,1 | rev | sort | uniq -c | sort -nr | grep -v '1 ' | head -n 10 && cat subdomains.txt | rev | cut -d '.' -f 4,3,2,1 | rev | sort | uniq -c | sort -nr | grep -v '1 ' | head -n 10 ) | sed -e 's/^[[:space:]]*//' | cut -d ' ' -f 2);do 
    subfinder -d example.com-all -silent | anew -q passive_recursive.txt
    assetfinder --subs-only example.com | anew -q passive_recursive.txt
    amass enum -passive -d example.com | anew -q passive_recursive.txt
    findomain --quiet -t example | anew -q passive_recursive.txt
done
```

#### DNS BruteForce

* `dnsvalidator -tL <https://public-dns.info/nameservers.txt> -threads 100 -o resolvers.txt`
* `puredns bruteforce best-dns-wordlist.txt $domain -r resolvers.txt -w output.txt`

#### Permutation/Alterations

* `gotator -sub subdomains.txt -perm permutations_list.txt -depth 1 -numbers 10 -mindup -adv -md > gotator1.txt`
* `puredns resolve permutations.txt -r resolvers.txt`

#### JS / Source Code

* `cat subdomains.txt | httpx -random-agent -retries 2 -no-color -o probed_tmp_scrap.txt`
* `gospider -S probed_tmp_scrap.txt --js -t 50 -d 3 --sitemap --robots -w -r > gospider.txt`
* `puredns resolve scrap_subs.txt -w scrap_subs_resolved.txt -r resolvers.txt`

#### Google Analitics / vhost

* `analyticsrelationships --url <https://www.bugcrowd.com`>
* `gobuster vhost -u <https://example.com> -t 50 -w subdomains.txt`

#### HTTP Probe

* `httpx`
* `cat hosts.txt | httpx -follow-redirects -status-code -random-agent -o output.txt`
* `cat subdomains.txt | httpx -random-agent -retries 2 -no-color -o output.txt`
* `cat subdomains.txt | httpx -random-agent -retries 2 -threads 150 -no-color -ports 81,300,591,593,832,981,1010,1311,1099,2082,2095,2096,2480,3000,3128,3333,4243,4567,4711,4712,4993,5000,5104,5108,5280,5281,5601,5800,6543,7000,7001,7396,7474,8000,8001,8008,8014,8042,8060,8069,8080,8081,8083,8088,8090,8091,8095,8118,8123,8172,8181,8222,8243,8280,8281,8333,8337,8443,8500,8834,8880,8888,8983,9000,9001,9043,9060,9080,9090,9091,9200,9443,9502,9800,9981,10000,10250,11371,12443,15672,16080,17778,18091,18092,20720,32000,55440,55672 -o output.txt`
* `COMMON_PORTS_WEB="81,300,591,593,832,981,1010,1311,1099,2082,2095,2096,2480,3000,3128,3333,4243,4567,4711,4712,4993,5000,5104,5108,5280,5281,5601,5800,6543,7000,7001,7396,7474,8000,8001,8008,8014,8042,8060,8069,8080,8081,8083,8088,8090,8091,8095,8118,8123,8172,8181,8222,8243,8280,8281,8333,8337,8443,8500,8834,8880,8888,8983,9000,9001,9043,9060,9080,9090,9091,9200,9443,9502,9800,9981,10000,10250,11371,12443,15672,16080,17778,18091,18092,20720,32000,55440,55672"`
* `sudo unimap --fast-scan -f subdomains.txt --ports $COMMON_PORTS_WEB -q -k --url-output > unimap_commonweb.txt`
* `cat unimap_commonweb.txt | httpx -random-agent -status-code -silent -retries 2 -no-color | cut -d ' ' -f1 | tee probed_common_ports.txt`

***

* wayback machine
* `httpx -no-fallback ports http:80,https:443,https:8443,http:8080,https:8080,http:8000,https:8000`

### Kaynaklar

* [https://wordlists.assetnote.io/](https://wordlists.assetnote.io)
* `unfurl`
