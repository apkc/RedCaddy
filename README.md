# RedCaddy
C2 redirector base on caddy

## Table of content
* [Overview](#Overview)
* [Feature](#Feature)
* [Note](#Note)
* [Quick start](#Quick-start)
* [Step by step](#Step-by-step)
* [References](#References)

### Overview
Generate caddyfile with c2 malleable profiles

### Feature
- Block IP by GEOIP country
- Allow requests by header matcher
- User-agent & IP blacklist
- Support multiple redirection
- TeamServer port warden

### Note
- **The "redwarden_parser.py" under modules is from [RedWarden](https://github.com/mgeeky/RedWarden) by [mgeeky](https://github.com/mgeeky)**  
- Plenty of inspiration from this article: [🇬🇧 Carrying the Tortellini's golf sticks](https://aptw.tf/2021/11/25/c2-redirectors-using-caddy.html)  
- IP Blacklists from [RedGuard's IP Blacklists](https://github.com/wikiZ/RedGuard/blob/main/data/banned_ips.go)
- User-Agent Blacklists from [mitchellkrogza's UA blacklists](https://github.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/blob/master/_generator_lists/bad-user-agents.list)  
- self-signed-cert.py modified from [CarbonCopy](https://github.com/paranoidninja/CarbonCopy) 

### Quick start
- Generate self-signed certificate
- Build the custom caddy with specific modules (optional)
- **Make sure `set trust_x_forwarded_for "true";` already enabled in c2 profile**
- Copy your C2 profile into RedCaddy
- Add your redirect rules into files (E.g [chains.list](https://github.com/XiaoliChan/RedCaddy/blob/main/chains.list))
- Finally, generate Caddyfile with the ugly python script.

### Step by step
- Generate self-signed certificates with self-signed-cert.py :  
`python3 cert-test.py -t [Https Server] -l 192.168.85.133`
![image](https://user-images.githubusercontent.com/30458572/196379200-a2e080d4-86b9-4755-b560-38d2887204ff.png)
As you can see, `*.crt`, `*.key`, `*.p12`, `*.store` are generated  
![image](https://user-images.githubusercontent.com/30458572/196379755-ec5bf87f-fca9-4395-8e64-f568a73c5d18.png)

- Build the custom caddy with specific modules (optional)  
```
wget https://github.com/caddyserver/xcaddy/releases/download/v0.3.1/xcaddy_0.3.1_linux_amd64.tar.gz
tar -zxvf xcaddy_0.3.1_linux_amd64.tar.gz
./xcaddy build \
    --with github.com/aksdb/caddy-cgi/v2 \
    --with github.com/porech/caddy-maxmind-geolocation
```

- Enable `set trust_x_forwarded_for "true";` in c2 profile  
![image](https://user-images.githubusercontent.com/30458572/196095882-c60f306c-b11d-4642-af0c-86779200b3d3.png)

- Copy the C2 profile into RedCaddy, in this case, I use [threatexpress‘s jquery-c2.4.3.profile](https://github.com/threatexpress/malleable-c2/blob/master/jquery-c2.4.3.profile) as demo  
![image](https://user-images.githubusercontent.com/30458572/195805856-bb7e5352-6227-42df-92da-7682511cc7c1.png)

- Edit redirection rules, here is the format:
```
443:https:192.168.85.133:10001:warden:50050
1443:https:192.168.85.133:10002
2443:https:192.168.85.133:10003
3433:https:192.168.85.133:10004
```
- **Q: What is "warden"?**  
A: Warden is a whitelist function feature to protect your teamserver port, this will generate a random link with random secure strings. The user without the ability to connect to teamserver before trigged it ("warden" behind 443 means handling the link on port 443).

- Pass arguments the generator.py needed, then hit enter.  
`python generator.py -f jquery-c2.4.3.profile -l [Ethernet Interface IP Address] -r chains.list -c CN -o Caddyfile`
![image](https://user-images.githubusercontent.com/30458572/195813570-bb067849-e606-4a8f-b2e6-595ff0321aa0.png)

- Run caddy with caddyfile which is generated :)  
`sudo ./caddy run --config Caddyfile --adapter caddyfile`
![image](https://user-images.githubusercontent.com/30458572/195814646-fb301054-877c-4e72-b5c2-97bfa2d5f818.png)

- **Q: Why not use json or yaml format?**  
A: Sorry, I don't know how to write caddyfile in json/yaml format.

- **Q: Can response 404 with unmatch routes?**  
A: Well, caddy can't do this ¯\\_(ツ)_/¯.

### Reference
- [Malleable C2 Profile parser](https://github.com/mgeeky/RedWarden/blob/master/plugins/malleable_redirector.py)  
- [🇬🇧 Carrying the Tortellini's golf sticks](https://aptw.tf/2021/11/25/c2-redirectors-using-caddy.html)  
- [Cobalt stagger](https://improsec.com/tech-blog/staging-cobalt-strike-with-mtls-using-caddy)  
- [How to create self-signed certificates](https://gist.github.com/cecilemuller/9492b848eb8fe46d462abeb26656c4f8)  
- [Caddy access control](https://blog.xm.mk/posts/da50/)  
