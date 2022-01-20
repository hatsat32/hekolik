# Subdoman Enum

### DNS Resolvers

* [https://public-dns.info/nameservers.txt](https://public-dns.info/nameservers.txt)

```
# https://github.com/vortexau/dnsvalidator.git
dnsvalidator -tL https://public-dns.info/nameservers.txt -threads 100 -o resolvers.txt
```

### Yatay Subdomain Bulma

Bazı büyük şirketler birden çok alt şirkete ve domaine sahiptirler. Mesela google şirketi youtube, blogger gibi birçok domaine şirkete sahiptir. İlgili domainlerin tamamını bulmak için çeşitli yöntemlerimiz var.

* [https://www.whoisxmlapi.com/](https://www.whoisxmlapi.com)
* [https://www.whoxy.com/](https://www.whoxy.com)
* [https://www.crunchbase.com/](https://www.crunchbase.com)
* [https://bgp.he.net/](https://bgp.he.net) ASN numarasını bul ve bu numara ile arama yap.

```
whois -h whois.radb.net  -- '-i origin $ASN$' | grep -Eo "([0-9.]+){4}/[0-9]+" | uniq
```

## Pasif Teknikler

### Manuel

Bazı servisler ücretli api sağlarken web siteleri aracılığı ile ücretsiz servis sağlayabilirler.

* dnsdumpter.com ([https://dnsdumpster.com/](https://dnsdumpster.com))
* subdomainfinder.c99.nl ([https://subdomainfinder.c99.nl/index.php](https://subdomainfinder.c99.nl/index.php))

### Araçlar

Passive araçlar ile subdomainleri bulmaya çalış

```
# subfinder config ile
subfinder -d $domain -all -t 50 -config subfinder.yml

# amass
amass enum -list -config config.ini # api keyleri kontrol et
amass enum -passive -config config.ini -d example.com -o out
amass enum -active -brute -w subdomain-wordlist.txt -d example.com -o out

# assetfinder
assetfinder -subs-only example.com > out
cat domains | assetfinder -subs-only > out

# findomain
findomain -t example.com -u out

# sudomy
sudomy -d example.com
```

### İnternet Arşivleri

İnternet arşivleri bize subdomain bulmada yardımcı olabilir.

```
# github.com/bp0lr/gauplus
gauplus -t 5 -random-agent -subs example.com |  unfurl -u domains | anew out.txt

# github.com/tomnomnom/waybackurls
waybackurls example.com |  unfurl -u domains | sort -u output.txt

# gau
echo example.com | gau -subs -t 10 -o out.wayback.gau
```

### Github Subdomain Enum

Github token gereklidir.

```
# github.com/gwen001/github-subdomains
github-subdomains -d example.com -t tokens.txt -o output.txt
```

### Rapid7 Dataset

Rapid7 oldukça geniş bir datasetine sahiptir. Subdomain kullanmak için kullanabiliriz.  `crobat` aracı ile subdomain bulabiliriz.

```
# github.com/cgboal/sonarsearch/cmd/crobat
crobat -s example.com > output.txt
```

### Sertifika Logları

```
# https://github.com/UnaPibaGeek/ctfr.git
python3 ctfr.py -d target.com -o output.txt
```

Tek Satırlık komutlar

```
curl "https://tls.bufferover.run/dns?q=.dell.com" | jq -r .Results[] | cut -d ',' -f4 | grep -F ".dell.com" | anew -q output.txt
```

```
curl "https://dns.bufferover.run/dns?q=.dell.com" | jq -r '.FDNS_A'[],'.RDNS'[]  | cut -d ',' -f2 | grep -F ".dell.com" | anew -q output.txt
```

### Recursive

Bulunan subdomainler üzerinden tekrar arama yaparak daha fazla subdomain bulabiliriz. Bunun için kısa bir bash script.

```shell
for sub in $( ( cat subdomains.txt | rev | cut -d '.' -f 3,2,1 | rev | sort | uniq -c | sort -nr | grep -v '1 ' | head -n 10 && cat subdomains.txt | rev | cut -d '.' -f 4,3,2,1 | rev | sort | uniq -c | sort -nr | grep -v '1 ' | head -n 10 ) | sed -e 's/^[[:space:]]*//' | cut -d ' ' -f 2);do 
    subfinder -d example.com-all -silent | anew -q passive_recursive.txt
    assetfinder --subs-only example.com | anew -q passive_recursive.txt
    amass enum -passive -d example.com | anew -q passive_recursive.txt
    findomain --quiet -t example | anew -q passive_recursive.txt
done
```

## Aktif Teknikler

### DNS BruteForce

```
# github.com/d3mondev/puredns/v2
puredns bruteforce wordlist.txt example.com -r resolvers.txt -w output.txt
```

Kullanılabilecek wordlistler:

* best-dns-wordlist.txt ([https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt))
* 2M ([https://gist.github.com/jhaddix/f64c97d0863a78454e44c2f7119c2a6a](https://gist.github.com/jhaddix/f64c97d0863a78454e44c2f7119c2a6a))
* 2m-subdomains.txt ([https://wordlists-cdn.assetnote.io/data/manual/2m-subdomains.txt](https://wordlists-cdn.assetnote.io/data/manual/2m-subdomains.txt))
* [https://wordlists.assetnote.io/](https://wordlists.assetnote.io)

### Permutation & Alterations

```
# github.com/Josue87/gotator
gotator -sub subdomains.txt -perm permutations_list.txt -depth 1 -numbers 10 -mindup -adv -md > gotator1.txt

puredns resolve permutations.txt -r resolvers.txt
```

* wordlist: [https://gist.githubusercontent.com/six2dez/ffc2b14d283e8f8eff6ac83e20a3c4b4/raw](https://gist.githubusercontent.com/six2dez/ffc2b14d283e8f8eff6ac83e20a3c4b4/raw)

### JS / Source Code

```
cat subdomains.txt | httpx -random-agent -retries 2 -no-color -o probed_tmp_scrap.txt

# github.com/jaeles-project/gospider
gospider -S probed_tmp_scrap.txt --js -t 50 -d 3 --sitemap --robots -w -r > gospider.txt

puredns resolve scrap_subs.txt -w scrap_subs_resolved.txt -r resolvers.txt
```

### Google Analitics / vhost

```
# github.com/Josue87/AnalyticsRelationships
analyticsrelationships --url https://www.bugcrowd.com
```

### TLS & CSP & Cname

```
# github.com/glebarez/cero
cero in.search.yahoo.com | sed 's/^*.//' | grep -e "\." | anew
```

```
cat subdomains.txt | httpx -csp-probe -status-code -retries 2 -no-color | anew csp_probed.txt | cut -d ' ' -f1 | unfurl -u domains | anew -q csp_subdomains.txt
```

### VHOST probing

```
gobuster vhost -u https://example.com> -t 50 -w subdomains.txt
```

### HTTP Probe

Web Porsts wordlist: ([https://gist.github.com/sidxparab/459fa5e733b5fd3dd6c3aac05008c21c](https://gist.github.com/sidxparab/459fa5e733b5fd3dd6c3aac05008c21c))



```
COMMON_PORTS_WEB="81,300,591,593,832,981,1010,1311,1099,2082,2095,2096,2480,3000,3128,3333,4243,4567,4711,4712,4993,5000,5104,5108,5280,5281,5601,5800,6543,7000,7001,7396,7474,8000,8001,8008,8014,8042,8060,8069,8080,8081,8083,8088,8090,8091,8095,8118,8123,8172,8181,8222,8243,8280,8281,8333,8337,8443,8500,8834,8880,8888,8983,9000,9001,9043,9060,9080,9090,9091,9200,9443,9502,9800,9981,10000,10250,11371,12443,15672,16080,17778,18091,18092,20720,32000,55440,55672"
```

```
sudo unimap --fast-scan -f subdomains.txt --ports $COMMON_PORTS_WEB -q -k --url-output > unimap_commonweb.txt
```

```
cat unimap_commonweb.txt | httpx -random-agent -status-code -silent -retries 2 -no-color | cut -d ' ' -f1 | tee probed_common_ports.txt
```

### Kaynaklar & Araçlar

* [https://sidxparab.gitbook.io/subdomain-enumeration-guide/](https://sidxparab.gitbook.io/subdomain-enumeration-guide/) :star:
* [https://wordlists.assetnote.io/](https://wordlists.assetnote.io) :star:
* `unfurl`
* `dnsvalidator` ([https://github.com/vortexau/dnsvalidator](https://github.com/vortexau/dnsvalidator))
